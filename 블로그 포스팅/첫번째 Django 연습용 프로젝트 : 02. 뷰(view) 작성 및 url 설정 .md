## 소개



이번 포스팅에서는, django의 뷰를 작성하고, URL 라우팅을 설정하는 것에 대해서 다루도록 하겠다.





## MVT 모델



장고(Django)의 모델은 MVT(Model-View-Template) 모델을 따른다.





대략적인 MVT의 동작구조는 아래 그림과 같다.



<<사진>>





대략적으로 아래와 같은 과정을 따른다.



1. 사용자가 django로 만든 웹 서비스에 서비스를 요청
2. 요청한 URL에 따라서 해당 URL에 매칭되는 view에 매핑된 함수를 호출
3. 호출된 View에 있는 함수가 Web Request를 받아서, 적절하게 비즈니스 로직을 실행
   1. 이때, 모델에 있는 데이터를 가져오거나, 모델에 있는 데이터를 수정/삭제/생성 수행
4. 비즈니스 로직이 다 실행되고, 이에 따른 적절한 web response를 반환
   1. 반환된 web response는 template에 맞게 적절하게 가공되서 반환 됨 



이때, web request/response 는 보통 HTTP request를 의미한다.



이제, 저 사진에 있는 MVT가 각각 무엇을 의미하는지 알아보도록 하자.



각 요소에 대한 설명은 아래 표와 같다.

| 요소     | 설명                                                      |
| -------- | --------------------------------------------------------- |
| Model    | 데이터를 어떠한 방식으로 정의하고 저장할건지를 정의       |
| View     | Web Request를 받아서 Web Response를 반환해주는 함수(요소) |
| Template | 사용자에게 UI(즉 View)를 어떻게 보여 줄지를 담당          |





이제 개략적으로 django의 MVT 구조에 대해 살펴 보았으니, 실제로 코드를 작성해보도록 하자.





## 1. Django App 뷰(View) 작성



먼저, 지난시간에 생성한 sudoku app에서 views.py 를 열어서 아래와 같이 작성을 해보자.



```python
from django.http import HttpResponse


def index(request):
  return HttpResponse("index page")


def ranking(request):
  return HttpResponse("ranking view page")


def get_ranking_list(request):
  return HttpResponse("get ranking list page")


def register_ranking(request):
  return HttpResponse("register ranking page")


def check_sudoku(request):
  return HttpResponse("check sudoku page")


def make_sudoku(request):
  return HttpResponse("sudoku make page")
  
```



이제 저 view에 선언한 함수를 URL과 매칭을 해주면,

사용자가 매칭된 URL에 브라우저로 요청을 보내는 경우, 

URL에 매칭된, 함수가 실행이 되어 그에 맞는 결과를 반환하게 된다.



이제 저 view와 url을 연결해보도록 하자.





## 2. Django App URL 설정(sudoku/urls.py)



다음으로, sudoku 폴더에서, urls.py 를 생성하고 아래와 같이 작성을 해주자.



```python
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('ranking', views.ranking, name='ranking'),
    path('sudoku/ranking/register', views.register_ranking, name='register_ranking'),
    path('sudoku/ranking', views.get_ranking_list, name='get_ranking_list'),
    path('sudoku/check', views.check_sudoku, name='check_sudoku'),
    path('sudoku/make', views.make_sudoku, name='make_sudoku'),
]
```



django에서는 path 함수를 이용해서 url을 매칭해 줄 수 있다.

또한, django.urls 에 URL이나 라우팅과 관련된 함수들이 모두 모여있다는 것도 유추 할 수 있다.



마지막으로, 저 URL에 정규표현식을 사용하여 복수개의 URL을 지정할 수 도 있고, URL로 파라미터를 받을 수 도 있다.



path함수의 인자는 아래와 같이 주어진다.



| 인자(첫번째 부터) | 설명               |
| ----------------- | ------------------ |
| URL               | 요청할 URL         |
| Method            | view method        |
| Name              | view method의 이름 |



이제 app에 대한 url 매칭은 끝났으니, 전체 프로젝트에서 url을 매칭하도록 하자.



## 3. URL 설정(urls.py)



다음으로, django_sudoku 에 있는 urls.py 를 아래와 같이 수정해보도록 하자.



```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('', include('sudoku.urls')),
    path('admin/', admin.site.urls)
]
```



위에서 만든 urls.py 를 include함수를 이용해서 포함 시킬 수 있다.



또한, admin/ 이라는 url이 기본적으로 지정되어 있는데, 

Django에서는 admin페이지에 대한 모듈들을 기본으로 지원하는 아주 편한 특징이 있다.

admin에 관련된 부분은 추후 포스팅에서 다루도록 하겠다.





다음으로는, django 앱과 관련된 설정을 해보도록 하겠다.





## 4. Django 앱 설정



URL만 설정하고 끝났으면 좋겠지만, 한 가지 더 해야 할 일이 남아있다.

지난시간에도 설명한 부분이지만, Django는 플러그인 기반의 아키텍처로 되어 있어서 자유롭게

다른 사람의 앱을 등록할수 있다고 설명한 바 있다.



이를 다시 생각해보면, 우리가 만든 django 앱이라도 Django프로젝트에 등록해주어야 한다는 사실을 유추할 수 있다.



이제 settings.py 를 열어서 아래와 같이 우리가 만든 앱을 등록하도록 하자.



```python
# Application definition

INSTALLED_APPS = [
    'sudoku.apps.SudokuConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```



음..? 그런데 뭔가 이상하다. 우리는 SudokuConfig라는 파일을 만들어 준 기억이 없다.



그렇다, 이제 저 SudokuConfig를 만들어야 된다.



이제 저 SudokuConfig 를 만들어 보도록 하자.





## 5. AppConfig 생성



sudoku폴더의 apps.py 를 열어서 아래와 같이 SudokuConfig 클래스를 만들자.



```python
from django.apps import AppConfig


class SudokuConfig(AppConfig):
    name = 'sudoku'
```



위의 코드와 같이 작성 함으로써, Django 프로젝트에 등록할 Config 클래스를 만들 수 있다.

여담으로, "name" 이름과 같은 파이썬 모듈이 같은 앱 내에 존재하는 경우 에러를 발생시키게 되므로 주의해야 한다.





## 6. 실행



이제 작성이 완료되면 실행을 해보도록 하자.



```shell
$ python3 manage.py runserver 0.0.0.0:8000
```



실행화면은 아래와 같다.







다음 포스팅에서는, sudoku api를 만들어보도록 하겠다.