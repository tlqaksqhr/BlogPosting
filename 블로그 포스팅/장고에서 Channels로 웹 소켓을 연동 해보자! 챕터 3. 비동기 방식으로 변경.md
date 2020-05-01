## 개요 



이번 포스팅에서는 지난 포스팅에서 작성한, 웹소켓 Consumer를 비동기방식으로 바꾸어 보도록 하겠다.





## 왜 비동기로 바꾸는가?





사실, 기존의 동기방식은 비동기 방식에 비해, I/O가 끝날때 까지 계속 기다려야 하므로 비효율 적이라는 단점이 존재한다.



하지만, 비동기로 작성하는 경우 I/O가 끝나지 않아도 즉시 결과값이 리턴된다. 



따라서, I/O과 완료되면 완료된거에 따른 콜백함수가 호출되는 방식으로 진행된다. 

때문에, 계속 다른일을 할 수 있어서 더 효율적이게 된다.



여담으로, Django ORM은 동기방식으로 작동한다.



이제 동기 방식으로된 코드를 비동기 방식으로 바꾸어 보자



## 예제



chat/consumer.py 를 아래 처럼 수정해준다.



```python
from channels.generic.websocket import AsyncWebsocketConsumer
import json

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = 'chat_%s' % self.room_name

        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

    async def disconnect(self, close_code):
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )

    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )

    async def chat_message(self, event):
        message = event['message']

        await self.send(text_data=json.dumps({
            'message': message
        }))
```



비동기 방식으로 Consumer를 사용하는 경우 위의 코드처럼 AsyncWebsocketConsumer와 같이,

앞에 Async가 붙은 컨슈머(Consumer) 클래스를 사용해주면 된다. 



그리고 파이썬에서는 async 키워드를 이용해서 async로 함수가 동작한다는 사실을 알려준다.

또한, await를 이용해서 async 함수를 호출하고 실제 결과값을 받아 올 수 있다.

(I/O가 완료된 경우에만 await에 있는 함수부분이 실행 됨)



## 실행



실행 결과는 지난 포스팅과 똑같다.