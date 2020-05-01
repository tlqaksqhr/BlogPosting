## 개요



이번 포스팅에서는 python의 메시지 큐 라이브러리인 celery에 대해서 소개하고 간단한 예제를 돌려보도록 하겠습니다.



## Celery?



Celery는, 분산 메시지 패싱을 이용해서 비동기적으로 작동하는 작업 큐 입니다.



Celery를 통해서 동기 방식(Synchronous)의 작업을 비동기 방식(Asynchronus)의 코드로 바꿔 줄 수 있습니다.



Celery의 장점 중 하나는, python에서 가장 많이 쓰이는 웹 프레임워크인 Django과 연동하는 기능을 공식적으로 지원한다는것입니다.



Celery 공식 홈페이지에서 Django 연동 가이드도 소개해주고 있어서 상대적으로 다른 프레임워크에 비해서 연동이 쉽습니다.



Celery는 기본적으로 rabbitMQ를 메시지 브로커로 사용하는것을 권장합니다.

그 외에도 redis, mongodb, 심지어 RDBMS(django ORM 포함)도 메시지 브로커로 사용을 할 수 있습니다.



마지막으로, Celery를 작업 스케쥴러로도 사용할 수 있습니다. 특히, 아래와 같은 작업을 수행할 때 유용하게 활용 할 수 있습니다.



- 매일 xx시 yy분에 특정 사이트에서 데이터를 긁어와라
- zz분 마다 특정 사용자들에게 이메일을 발송해라



## 설치



먼저 아래와 같은 명령어를 통해서 rabbitMQ를 설치해줍니다.



```shell
brew install rabbitmq
```



설치가 끝나고 나면, pip에서 아래 명령어를 이용해서 celery 를 설치해주면 됩니다.



```shell
$ pip install celery
```





## 예제



일단 이번 예제에서는, celery가 간략하게 어떻게 돌아가는지 보기 위해 분산처리로 덧셈을 하는(의미는 없지만) 간단한 예제를 진행하도록 하겠습니다.



먼저 tasks.py 를 아래와 같이 작성 해 줍시다.

```python
from celery import Celery

app = Celery('tasks', broker='amqp://guest@localhost//')

@app.task
def add(x, y):
    return x + y
```



위 코드는, rabbitMQ를 메시지 브로커로 설정하여 rabbitMQ를 통해서 작업을 요청하겠다는 의미입니다.

또한, @app.task 데코레이터(decorator)를 이용해서 요청할 작업을 정의 해줍니다.



그런 뒤 celery 워커(worker) 서버를 아래와 같이 실행 해 줍시다.



```shell
$ python3 -m celery -A tasks worker --loglevel=info
```



이제 celery를 이용해서 위에서 선언한 함수를 아래 코드와 같이 호출 할 수 있습니다.



```python
from tasks import add
result = add.delay(4, 4)
```





만약 결과를 돌려받고 싶다면, 아래 코드와 같이 작성하면 됩니다.



```python
app = Celery('tasks', backend='rpc://', broker='amqp://')
```



위의 예제는, celery를 이용해서 처리한 결과를 redis를 이용해서 임시로 저장하고, 그 저장한 값을 돌려주도록 설정해주는 코드입니다. 



그런 뒤, 아래와 같이 결과를 반환 받을 수 있습니다.



```python
result.get()
```





또한, timeout내에 결과를 반환하도록 아래와 같이 코드를 작성할 수 도 있습니다.



```python
result.get(timeout=1)
```

(만약 timeout이내에 결과를 반환하지 못하면, 예외가 발생하게 됩니다.)

