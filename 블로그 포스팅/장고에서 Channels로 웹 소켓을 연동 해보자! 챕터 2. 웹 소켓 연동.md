## 개요



이번 포스팅에서는 실제로 Channel 라이브러리를 이용해서 웹 소켓 요청을 처리하는 부분을 작성하도록 하겠습니다.





## 웹 소켓?



웹 소켓은 말 그대로, 웹 브라우저에서 서버와 계속 연결을 유지하면서 소켓 통신을 할 수 있게 해주는 프로토콜을 의미한다.

주로 채팅 서버나, 연결을 계속 유지하면서 메시지를 받거나 전송해야 하는 곳(Ex : 푸시 알림, 게임)에 많이 쓰인다.



보통, HTTP/S 등으로, 웹소켓을 연결한다는 메시지와 기본적인 세션정보들을 전송하고, 

그 이후에 실제 웹소켓으로 통신하는 방식으로 이루어져 있다.



사실, 기존의 http나 ajax 같은것들이 있는데 굳이 웹소켓 같은 것을 써야하나? 라고 생각하는 사람들이 있을 수 도 있다.



하지만, 푸시 알림이나 서버와 계속 연결을 유지하면서 데이터를 주고받아야 하는 경우에 HTTP를 이용하는 경우는 매우 비효율적이고(데이터 오버헤드 문제가 발생한다), 연결 상태를 계속 유지하는것도 어렵기 때문에(keep-alive 속성이 있기는 하지만, 권장되지는 않는다.) 웹 소켓(websocket)과 같은 기술이 나오게 되었다.



node.js와 같은경우 socket.io 라는 라이브러리를 주로 사용하게 된다.

(사실 각 언어별로 socket.io 구현체가 있기는 하지만, django와 연동되는 라이브러리들은 따로 존재하지 않았다.)



이제 개략적으로 설명을 했으니, 예제를 작성해보도록 하자.



## HTML과 Django View 작성



먼저, 아래와 같이 templates/chat/room.html을 추가 해준다.



```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Chat Room</title>
</head>
<body>
    <textarea id="chat-log" cols="100" rows="20"></textarea><br />
    <input id="chat-message-input" type="text" size="100"><br />
    <input id="chat-message-submit" type="text" size="100"><br />
    
    <script type="text/javascript">
        let roomName = "{{ room_name | escapejs }}";

        let chatSocket = new WebSocket(
            `ws://${window.location.host}/ws/chat/${roomName}/`
        );

        chatSocket.onmessage = (e) => {
            let data = JSON.parse(e.data);
            let message = data['message'];
            document.querySelector("#chat-log").value += (message + '\n');
        };

        chatSocket.onclose = (e) => {
            console.error('Chat socket closed unexpectedly');
        };

        document.querySelector("#chat-message-input").focus();
        document.querySelector("#chat-message-input").addEventListener("keyup",(e) => {
            if (e.keyCode === 13) { 
                document.querySelector("#chat-message-submit").click();
            }
        });

        document.querySelector("#chat-message-submit").addEventListener("click", (e) => {
            let messageInputDom = document.querySelector("#chat-message-input");
            let message = messageInputDom.value;
            chatSocket.send(JSON.stringify({
                'message' : message
            }));

            messageInputDom.value = '';
        });
    </script>
</body>
</html>
```



다음으로, chat/views.py 에 아래와 같이 코드를 추가 해준다.



```python
from django.shortcuts import render

def index(request):
    return render(request, 'chat/index.html', {})

def room(request, room_name):
    return render(request, 'chat/room.html', {
        'room_name': room_name
    })

```



위 코드를 이용해서 현재 채팅방의 이름을 HTTP로 받아올 수 있다.



위의 코드를 작성 완료하고 나서, chat/urls.py를 아래와 같이 수정해준다.



```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('<str:room_name>/', views.room, name='room'),
]
```





일단 위의 과정까지 마치면 웹소켓으로 통신하는 부분을 제외하고는 완료하였다. 







## 웹소켓 Consumer 작성



이제 실제 웹소켓과 통신하는 부분을 작성 해보도록 하자.



먼저, chat 폴더 안에 consumer.py 를 생성하고 아래와 같이 작성해보도록 하자.



```python
from channels.generic.websocket import WebsocketConsumer
import json

class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.accept()

    def disconnect(self, close_code):
        pass

    def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        self.send(text_data=json.dumps({
            'message': message
        }))
```



Channel에서는 WebsocketConsumer 클래스를 상속받아서, 웹 소켓을 처리할 수 있는 핸들러 클래스를 만들 수 있는데, 

해당 방식으로 생성한 핸들러 클래스는 동기방식으로 작동을 한다.



또한, 커넥션 1개당 Consumer 1개가 생성되며, 기본적으로 다른 Consumer와는 데이터를 공유하지 않는다.

만약, 데이터를 공유해야 하는 경우 특별한 방법이 필요하다. 그 방법에 대해서는 밑에서 설명하도록 하겠다.



각 함수별 역할은 아래와 같다.



1. connect : 사용자와 websocket 연결이 맺어졌을때 호출
2. receive : 사용자가 메시지를 보내면 호출 됨
3. disconnect : 사용자와 websocket 연결이 끊겼을때 호출



해당 코드에서는 일단 메시지를 받으면 메시지를 보낸 사람한테만 보낸 메시지 그대로 전송을 해준다.

다시말하면 그냥 echo서버와 비슷한 역할을 수행한다.



작성이 완료되었으면, chat/routing.py 를 열어서 아래와 같이 수정을 해준다.



```python
from django.urls import re_path

from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer),
]
```



마지막으로, 우리가 작성한 URL을 Channel router에 아래와 같이 등록을 해주도록 합시다.



```python
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import chat.routing

application = ProtocolTypeRouter({
    'websocket': AuthMiddlewareStack(
        URLRouter(
            chat.routing.websocket_urlpatterns
        )
    ),
})
```



먼저, 코드를 설명하면, 사용자로부터 연결이 생성되면 위의 ProtocolTypeRouter가 어떤 프로토콜로 

연결이 됬는지를 파악하고, 프로토콜에 해당하는 Middleware로 연결해주게 된다.



예제에서는, AuthMiddlewareStack으로 연결 되는데, 해당 미들웨어는 장고의 **AuthenticationMiddleware**와 비슷하게, 인증된 유저여부에 따라서 연결의 **scope** 를 제한하게 된다. **scope** 는 일종의 Django의 세션과 비슷한거라고 보면 된다.



마지막으로 실제 URLRouter에서 넘겨받은 URL을 기반으로 그에 맞는 핸들러를 실행해주게 된다.



이제 여기까지 작성을 완료하면 일단 웹소켓과 통신을 하는 부분은 작성을 하였다. 



하지만, 단순히 echo서버의 역할만 할 뿐 내가 작성한 채팅내용이 채팅방에 접속한 다른 유저들에게는 보이지 않는다. 

이제 다른 유저들에게도 내 채팅내용이 보이도록 작성해보도록 하겠다. 





## CHANNEL LAYER 작성



먼저, 코드를 작성하기에 앞서서, Channel 라이브러리의 Channel Layer에 대해 알아보도록 하자.



위에서 언급했듯이 Channel 레이어는 각 Consumer끼리 데이터의 공유를 하기 위해서는 특별한 설정을 해야한다고 언급하였다.



Channel에서는 이런 Consumer끼리의 데이터 공유를 위해 Channel Layer라는 것을 도입하였다.



채널 레이어(Channel Layer)는 아래 2가지의 구성요소로 이루어져 있다.



1. 채널 : 일종의 메시지를 보낼 통로이자, 메시지를 공유할 저장소
   1. rabbitMQ의 메시지 큐와 비슷함.
2. 그룹 : 채널들을 여러개 묶어서 관리하는 단위
   1. rabbitMQ의 토픽과 비슷함.
3. 백엔드 : 실제 내용을 공유할 저장소의 종류 선택. 종류는 아래와 같다(일종의 rabbitMQ의 메시지 브로커랑 비슷함).
   1. 메모리
   2. redis
   3. AMQP



이번 예제에서는 redis를 백엔드로 사용할것이다.



먼저 레디스를 설치해주자.



가장 편한 방법은 PC나 서버에 도커를 설치하고 아래 명령어를 이용해서 도커를 띄워주는 것이다.



```shell
$ docker run -p 6379:6379 -d redis:2.8
```



다음으로 channels_redis를 설치 해주자. 

아래 명령어로 설치가 가능하다.



```shell
$ pip3 install channels_redis

또는

$ python3 -m pip install channels_redis
```



설치가 완료되면, 아래와 같이 mysite/settings.py 에 아래 내용을 추가해주자.



```python
ASGI_APPLICATION = 'mysite.routing.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('127.0.0.1', 6379)],
        },
    },
}
```



먼저 아래 명령어를 이용해서 django 서버를 띄운다.



```shell
$ python3 manage.py runserver
```



테스트는 아래와 같이 가능하다.



```python
$ python3 manage.py shell

>>> import channels.layers
>>> channel_layer = channels.layers.get_channel_layer()
>>> from asgiref.sync import async_to_sync
>>> async_to_sync(channel_layer.send)('test_channel', {'type': 'hello'})
>>> async_to_sync(channel_layer.receive)('test_channel')
{'type': 'hello'}
```

 

마지막으로 실제 websocket 연결을 처리할 chat/consumer.py 를 아래와 같이 수정해주자.



```python
from asgiref.sync import async_to_sync
from channels.generic.websocket import WebsocketConsumer
import json

class ChatConsumer(WebsocketConsumer):
    def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = 'chat_%s' % self.room_name

        # "room" 그룹에 가입
        async_to_sync(self.channel_layer.group_add)(
            self.room_group_name,
            self.channel_name
        )

        self.accept()

    def disconnect(self, close_code):
        # "room" 그룹에서 탈퇴
        async_to_sync(self.channel_layer.group_discard)(
            self.room_group_name,
            self.channel_name
        )

    # 웹소켓 으로 부터 메시지 수신
    def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']

        # "room" 그룹에 메시지 전송
        async_to_sync(self.channel_layer.group_send)(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message
            }
        )

    # "room" 그룹에서 메시지 전송
    def chat_message(self, event):
        message = event['message']

        # 웹 소켓으로 메시지 전송
        self.send(text_data=json.dumps({
            'message': message
        }))
```



먼저, async_to_sync 함수를 통해 비동기 방식으로 동작하는 함수들을 전부 동기 방식으로 변경하였다.

그리고 채팅방을 위에서 언급한 그룹 단위로 관리하는데, 이를 위에서 잠깐 언급한 scope를 통해서 관리하고 있다.



일단 코드에서 사용된 주요 변수 및 함수 들은 아래와 같다.



1. **self.scope\['url_route\']\['kwargs'\]\['room_name'\]**
   1. self.scope : 각 Consumer 에서 연결정보를 가지고 있는 변수
   2. 위 코드 처럼 작성하는 경우, room_name(group name)을 얻어올 수 있다.
   3. 또한, 소켓 통신을 연결한 사용자가 인증된 사용자인지도 검증 가능하다.
2. **self.room_group_name** **=** **'chat_%s'** **%** **self.room_name**
   1. 해당 채널이 속하는 그룹의 이름을 정해주는 부분.
   2. 위와 같이 작성하여 해당 채널이 속하는 그룹의 이름을 정할 수 있다.
   3. 그룹이름은 숫자, 알파벳, "_", 마침표 만 들어갈 수 있다.
3. **async_to_sync(self.channel_layer.group_add)(...)**
   1. **self.channel_layer.group_add** = 실제로 채널을 그룹에 등록하는 함수
   2. 해당 함수는 기본적으로 비동기로 작동하기 때문에 async_to_sync 함수를 통해 동기모드로 실행을 하였다.
   3. **self.room_group_name** 으로 이름을 정했다고 하더라도, 이미 있는 이름인 경우 실패할 수 있다.
4. **self.accept()**
   1. 연결을 요청한 사용자와 연결을 허용해 주는 함수
   2. 소켓의 accept 함수와 같은 역할을 수행함
5. **async_to_sync(self.channel_layer.group_discard)(...)**
   1. 그룹에서 등록을 해제하는 함수
6. **async_to_sync(self.channel_layer.group_send)**
   1. 실제로 그룹에 속한 모든 채널에 메시지를 보내는 함수
   2. 'type'을 이용해서 실제 메시지를 수신했을때의 이벤트 핸들러를 지정해 줄 수 있다.
      1. 이번 예제에서는 chat_message 함수를 핸들러 함수로 지정하였다.



이제 작성을 완료했으니, 실제로 채팅서버를 실행시켜보자.	





## 실행



먼저 아래 명령어를 실행시켜서 서버를 띄워보자



```shell
$ python3 manage.py runserver
```



그리고, 브라우저를 켜서 탭 2개를 띄운 뒤, 2개의 탭에서 모두 http://127.0.0.1:8000/chat/lobby/ 에 접속한다.



그런 뒤, 첫번째 탭에서 메시지를 치면 2번째 탭에서도 메시지가 보이는 것을 확인 할 수 있다.



