## 개요



이번 시리즈 에서는, Django에서 웹 소켓을 쓸 수 있게 해주는 라이브러리인 Channels을 이용해서 채팅서버를 만들어보는 예제를 만들어보도록 하겠습니다.  





## Channels?  

  

Channels 공식 홈페이지에 들어가면 아래와 같은 설명이 나와있습니다.  

  

> Channels is a project that takes Django and extends its abilities beyond HTTP - to handle WebSockets, chat protocols, IoT protocols, and more. It’s built on a Python specification called [ASGI](http://asgi.readthedocs.io/).
> 
>
> It does this by taking the core of Django and layering a fully asynchronous layer underneath, running Django itself in a synchronous mode but handling connections and sockets asynchronously, and giving you the choice to write in either style.
> 
>
> Channel은, Django에서 HTTP이외의 웹소켓이나, IoT 프로토콜과 같은 다양한 프로토콜을 지원하기 위해서 만들어진 프로젝트입니다. ASGI 기반으로 개발 되었습니다.
>
>
> 또한, Django와 형태가 유사하긴 하지만, 전부 비동기 기반으로 개발되었고, 장고 자체는 동기적으로 실행되지만, 실제 Channel에서 커넥션을 처리할때는 비동기로 작동합니다. 그리고 동기모드도 지원을 하기때문에 적절히 골라서 쓰시면 됩니다



즉, Channel은 위의 설명처럼 장고에서, HTTP이외의 프로토콜을 사용할 수 있게 해주고,

비동기로 작동하는 라이브러리 입니다.



따라서, 웹소켓도 지원하고, 웹소켓 이외의 다른 프로토콜도 지원하고 있습니다.



이제 실제 설치 방법에 대해서 다뤄보도록 하겠습니다.





## 설치



먼저, 아래 명령어를 이용해서 Channel 라이브러리 를 설치 해줍니다.



```shell
$ python3 -m pip install -U channels
```



설치 확인은 아래와 같이 할 수 있습니다.



```shell
$ python3 -c 'import channels; print(channels.__version__)'
```





다음으로는, 실제로 채팅서버를 작성하기 위한 기본 뼈대 코드 작성 및 Django에 Channel 적용을 하는 방법에 대해서 알아보도록 하겠습니다.





## 기본 뼈대 코드 작성 및 Django에 Channel 적용하기





먼저 아래 명령어를 이용해서 Django 프로젝트를 생성 해줍시다.



```shell
$ django-admin startproject mysite
```





다음으로, 아래 명령어를 이용해서 Django 앱을 만들어 줍시다.



```shell
$ python3 manage.py startapp chat
```



그런 뒤, 생성한 chat 앱. (mysite/chat 폴더)에서, 아래의 2개 파일을 제외한 파일들을 모두 삭제 해줍시다.



```shell
chat/
    __init__.py
    views.py
```

 

그리고, mysite/settings.py에서 아래와 같이 생성했던 장고 APP을 추가를 해줍시다.



```python
# mysite/settings.py
INSTALLED_APPS = [
    'chat',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```



다음으로는, 실제 사용자의 화면에 보여지는 페이지를 띄워보도록 하겠습니다.



먼저, chat폴더에 templates 폴더를 생성해주고, 그 안에 chat폴더를 생성한 후, index.html 파일을 생성해주시기 바랍니다.



대략 아래와 같은 폴더구조로 생성을 해주시면 됩니다.



```shell
chat/
    __init__.py
    templates/
        chat/
            index.html
    views.py
```



그리고, index.html에 아래 내용을 추가 해주시기를 바랍니다.



```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8"/>
    <title>Chat Rooms</title>
</head>
<body>
    What chat room would you like to enter?<br/>
    <input id="room-name-input" type="text" size="100"/><br/>
    <input id="room-name-submit" type="button" value="Enter"/>

    <script>
        document.querySelector("#room-name-input").focus();
        document.querySelector("#room-name-input").addEventListener("keyup",(e) => {
            if(e.keyCode === 13){
                document.querySelector("#room-name-submit").click();
            }
        });

        document.querySelector("#room-name-submit").addEventListener("click",(e) => {
            let roomName = document.querySelector("#room-name-input").value;
            window.location.pathname = `/chat/${roomName}/`;
        });
    </script>
</body>
</html>
```



이제, chat/views.py 를 아래와 같이 작성 해줍시다.



```python
from django.shortcuts import render

def index(request):
    return render(request, 'chat/index.html', {})
```



다음으로, chat/urls.py 파일을 만들어주고 아래와 같이 작성 해 줍시다.



```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```



그리고 mysite/urls.py 를 아래와 같이 작성 해줍시다.



```python
from django.conf.urls import include
from django.urls import path
from django.contrib import admin

urlpatterns = [
    path('chat/', include('chat.urls')),
    path('admin/', admin.site.urls),
]
```





다음으로, Channel 라이브러리를 Django에 적용해보도록 합시다.



아래 처럼, mystic 폴더 안에 routing.py 파일을 작성해줍시다.



```python
from channels.routing import ProtocolTypeRouter

application = ProtocolTypeRouter({
    # (http->django views is added by default)
})
```



위 코드는, 추후 포스팅에서 설명할 장고의 View에 해당하는 Consumer로 연결될 라우팅(django의 url에 해당함)을 해주는 부분입니다.



다시, mysite/setting.py 에 가서 작성한 라우팅 파일을 아래와 같이 등록해줍시다



```python
INSTALLED_APPS = [
    'channels',
    'chat',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]


ASGI_APPLICATION = 'mysite.routing.application'
```





## 실행



아래와 같이 실행 할 수 있습니다.



```shell
$ python3 manage.py runserver
```



아직 뼈대코드만 작성한거여서, 실제 채팅기능은 동작하지 않습니다.



다음 포스팅에서는 실제로 Channel과 웹소켓을 연동하는 법에 대해서 다뤄보도록 하겠습니다.

 



## 출처



1. https://channels.readthedocs.io/en/latest/tutorial