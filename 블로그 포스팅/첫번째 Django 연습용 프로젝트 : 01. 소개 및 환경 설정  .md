## 소개



이번 포스팅에서는, 파이썬과 장고(Django) 프레임워크를 이용해서 웹 기반의 스도쿠 게임을 만들기 위해서 Django 설치 및 세팅을 하는법에 대해서 다루도록 하겠습니다.





## 장고(Django)?



장고(django)는, 파이썬을 기반으로 한 오픈소스 웹 프레임워크 이다.



일반적인 웹 프레임워크와 다르다고 할 수 있는 점은, 

기존의 다른 웹 프레임워크들(Spring MVC, code ignitor, laravel 등)과는 다르게, 

MTV 라는 구조를 가진다는 것이 특징이다.



또한, 웹 프로젝트를 작성 할때, 각 기능들을 app이라는 단위로 각각 분리해서 개발 할 수 있다는 특징이 있다.



마지막으로, django는 플러그인 기반의 아키텍처를 지원 하기 때문에, 

비록 다른 django 프로젝트에서 만든 앱이라고 하더라도,

해당 앱 부분만 떼와서 우리의 프로젝트에 자유롭게 가져다 붙일수 있다.

이에 대한 자세한 내용은 다음 포스팅에서 다루도록 하겠다.



이제, django 프로젝트를 직접 만들어보도록 하자.





## 환경 설정



### 1. Pyenv 설정



(먼저, pyenv 설치가 안되신 분들은 아래의 링크를 참조해 주시기를 바랍니다.

pyenv 설치하는 법 : https://semtax.tistory.com/28



먼저 아래와 같이, pyenv로 파이썬을 다운로드 받아 준다.



```shell
$ pyenv install 3.6.8
```



그리고 나서, 아래와 같이 virtualenv 환경을 만들어 준다.



```shell
$ pyenv virtualenv 3.6.8 django_sudoku
```



마지막으로, 아래와 같이 가상환경을 실행 시켜준다.



```shell
$ pyenv activate django_sudoku
```





### 2. Django 설치



pyenv를 이용해서 가상 환경을 실행시켜준 상태에서, 아래와 같이 django를 설치 해준다.



```shell
$ pip3 install django
```



이렇게 하면, Django 설치가 완료 된다.





### 3. Django 프로젝트 생성



설치하고 나서, 아래와 같이 django_sudoku 라는 이름으로, 프로젝트를 생성시켜준다.



```shell
$ django-admin startproject django_sudoku
```



그리고 django_sudoku 폴더에 들어가면 아래와 같은 파일들을 확인 할 수 있다.



```shell
drwxr-xr-x   7 semtax  staff     224  3  1 15:53 .
drwxr-xr-x   8 semtax  staff     256  2 29 00:04 ..
drwxr-xr-x   8 semtax  staff     256  2 29 23:36 django_sudoku
-rwxr-xr-x   1 semtax  staff     633  2 28 19:39 manage.py
```



해당 manage.py 파일을 통해 앱 생성, 웹 서버 실행, 데이터베이스 설정과 같은, django와 관련된 다양한 설정들을 수행 할 수 있다. 





다시, django_sudoku 폴더에 들어가 보면 아래의 파일들을 확인 할 수 있다.



```shell
drwxr-xr-x  8 semtax  staff   256  2 29 23:36 .
drwxr-xr-x  7 semtax  staff   224  3  1 15:53 ..
-rw-r--r--  1 semtax  staff     0  2 28 19:39 __init__.py
-rw-r--r--  1 semtax  staff   403  2 28 19:39 asgi.py
-rw-r--r--  1 semtax  staff  3965  3  1 16:12 settings.py
-rw-r--r--  1 semtax  staff   801  3  1 00:05 urls.py
-rw-r--r--  1 semtax  staff   403  2 28 19:39 wsgi.py
```



각 파일들의 역할은 아래와 같다.



| 파일이름    | 역할                                                         |
| ----------- | ------------------------------------------------------------ |
| asgi.py     | 파이썬 ASGI 파일 관련 설정(WSGI와 유사한 것)                 |
| settings.py | django 설정값들이 들어있는 파일. 해당 파일을 수정해서 django의 여러 설정들을 수행 할 수 있다. |
| urls.py     | django에서의 URL 라우팅 경로 설정을 할 수 있는 파일.         |
| wsgi.py     | 파이썬 WSGI 파일 관련 설정(파이썬에서 HTTP프로토콜을 어떤식으로 처리할지 정해놓은것) |



### 4. Django App 생성



프로젝트를 생성하고 나서, 생성한 프로젝트 폴더(django_sudoku 폴더)에 들어 간 뒤에,

아래 명령어를 이용해서 django 앱을 생성 가능하다.



```shell
$ python3 manage.py startapp sudoku
```

 

생성한 sudoku 앱에 있는 파일들은 아래와 같다.



```shell
drwxr-xr-x  13 semtax  staff   416  3  1 00:23 .
drwxr-xr-x   7 semtax  staff   224  3  1 15:53 ..
-rw-r--r--   1 semtax  staff    90  2 29 23:30 admin.py
-rw-r--r--   1 semtax  staff    86  2 29 23:42 apps.py
-rw-r--r--   1 semtax  staff   140  2 29 21:16 models.py
-rw-r--r--   1 semtax  staff    60  2 28 21:54 tests.py
-rw-r--r--   1 semtax  staff  2313  3  1 16:15 views.py
```





각 파일들의 역할은 아래와 같다.



| 파일이름  | 역할                                                         |
| --------- | ------------------------------------------------------------ |
| admin.py  | django app의 관리자와 관련된 설정들을 해주는 파일이다, 여기다 모델을 등록해서 admin에서 관리가 가능하다. |
| apps.py   | django app의 설정값들이 들어있는 파일. 이 파일을 수정해서 django app의 설정들을 수행 할 수 있다. |
| models.py | django의 모델들을 작성하는 곳이다. 모델은 일종의 데이터를 정의해놓은곳이라고 보면 된다. |
| tests.py  | django app의 각 요소(뷰, 모델) 들을 테스트 하는 코드를 작성하는 곳이다. |
| views.py  | django 앱의 view가 정의되어 있는 파일이다, 여기에 뷰를 작성 해주면 된다. |





## 5. 실행



생성한 프로젝트를 아래의 명령어로 실행 해보자.





```shell
$ python3 manage.py runserver 0.0.0.0:8000
```





실행화면은 아래와 같다.



![Django welcome page](https://learndjango.com/static/images/django30_welcome.png)





다음 포스팅에서는, django 뷰를 생성하는 것에 대해서 다루도록 하겠다. 

