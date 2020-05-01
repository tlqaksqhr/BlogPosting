## 개요



이번 시간에는, 메가박스 알리미 서비스 제작을 위한, django 설치 및 기본 설정을 다루려고 합니다.



## 설치



python pip 를 이용해서 아래와 같이 설치를 해줍니다.



```shell
$ pip3 install django
```



아래와 같이 명령어를 실행했을시, 버전이 출력되면 정상적으로 설치된 것입니다.



```shell
$ python -m django --version
```



## 프로젝트 생성



아래 명령어를 이용해서 django 프로젝트를 생성 해줍니다.



```shell
$ django-admin startproject megabox_alarm
```

 

위 명령어를 실행하면 megabox_alarm 이라는 폴더가 생성되고, 해당 폴더에 들어가면 아래와 같은 파일과 폴더가 생성되어있습니다.



```shell
manage.py     megabox_alarm
```



각 파일 및 폴더 설명은 아래와 같습니다.



| 파일명        | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| manage.py     | 프로젝트 내에 있는 django 앱 생성, ORM 설정등을 해주는 커맨드라인 유틸리티 프로그램 |
| megabox_alarm | url 경로 설정이나 데이터베이스 설정 같은 프로젝트와 관련된 설정들이 들어있음 |



그런 뒤, 아래와 같이 manage.py 를 이용해서 django앱을 생성해줍시다.



```shell
$ python3 manage.py startapp megabox_notifier
```



megabox_notifier 앱 생성이 완료 되면, megabox_notifier/views.py 를 열어서 아래와 같이 작성해줍시다.



```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello Megabox notifier service!")
```



그런 뒤, megabox_notifier/urls.py 를 열어서 아래와 같이 작성 해줍시다.



```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```



마지막으로, megabox_alarm/urls.py 를 열어서 아래와 같이 작성을 해줍시다.



```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('megabox_notifier/', include('megabox_notifier.urls')),
    path('admin/', admin.site.urls),
]
```



작성이 완료되면 아래와 같이 명령어를 쳐서 웹 서비스를 실행해 줍시다.



```shell
$ python3 manage.py runserver
```



이제  http://localhost:8000/megabox_notifier/ 로 들어가면, 브라우저에서 "Hello Megabox notifier service!"라는 메시지를 볼 수 있습니다.