## 소개



이번 포스팅에서는, CSRF가 무엇인지에 대해 다루고 



부가적으로 관리자 기능을 활성화 시키는 법에 대해서도 다루도록 하겠다.







## 문제점



사실, 지난 포스팅 까지 진행 했던 내용을 그대로 실행시키면 에러가 발생 할 것이다.



에러 메시지를 자세히 보면 CSRF 라는 단어와 403 에러가 뜨면서 뭔가 인증에 실패했다라는 말이 보일 것이다.



이제 CSRF가 무엇인지에 대해 알아보자.







## CSRF



CSRF 공격은 Cross-site request forgery 공격의 약자로, 사용자가 자기 의지와는 상관없이 공격자가 의도한 행위를 특정 웹사이트에 요청하게 하는 방식으로 수행하는 공격이다.



예를 들어 우리가 제작한 스도쿠 사이트를 예를 들면, 공격자가 아래와 같은 스크립트를 작성하여 공격을 할 수 있는것이다.



```shell
<a href="http://127.0.0.1:8000/sudoku/make">클릭하세요!</a>
```



이런식으로, 링크를 클릭하면 사용자가 의도하지도 않았는데 스도쿠 퍼즐을 막 제멋대로 요청하게 되는것이다.



사실, 우리 사이트 같은 경우 딱히 CSRF 공격으로 얻을 이득이 별로 없기는 하다.



하지만 결제기능이 들어간 사이트나 개인정보를 수정하는 페이지를 상대로,

CSRF공격을 수행한다면 많은 금전적 피해를 입히게 된다.



실제로, 옥션에서 예전에 CSRF 취약점으로 공격을 당해서 많은 피해을 입은 사례도 존재한다.



이러한, 공격을 막기 위해, Django 에서는 CSRF 토큰 이라는 일종의 난수 비슷한것을 도입하였다.



이 토큰을 통해서, 우리 사이트 이외의 다른 사이트에서 HTTP 요청을 보내는 경우, 

유효하지 않은 요청이라 판단해서 403에러를 내면서 요청을 거부하게 되는 것이다. 



보통 템플릿을 사용하면 아래와 같이 사용을 한다.



```html
<form>
  {% csrf_token %}
  <input type="text" name="name"/>
  ....
</form>
```



하지만, AJAX 통신을 통해 서버와 통신하는 경우에는, 저런 csrf_token 키워드를 사용 할 수가 없다.



정말로 방법이 없는 것일까?  



다행히도, Django에서는, 이러한 ajax 통신을 할때에도 CSRF 토큰을 보낼 수 있는 방법을 제공한다.



코드를 작성하면서 알아보도록 하자.





## 코드 작성



Django 공식 홈페이지에 들어가면 우리와 같은 상황을 마주친 사람들을 위해 가이드를 제공하였다.



https://docs.djangoproject.com/en/3.0/ref/csrf/



대략 내용을 요약하면, 뷰에서 템플릿을 만들어 넘길때, CSRF 토큰값을 쿠키에 담아서 보내준다는 내용이다.

그리고, django 프레임워크에서 설정한 헤더와 쿠키로 CSRF 토큰을 넘기면 된다는 내용이다.



따라서, django에서 하라는 대로 하면 된다.





먼저, settings.py에 아래와 같이 코드를 추가해주자.



```python
.....

CORS_ORIGIN_ALLOW_ALL = True

CORS_ALLOW_CREDENTIALS = True

CSRF_TRUSTED_ORIGINS = (
    'localhost:8000',
    '127.0.0.1:8000',
)

CORS_ORIGIN_WHITELIST = (
    'localhost:8000',
  	'127.0.0.1:8000',
)

CORS_ALLOW_HEADERS = (
    'access-control-allow-credentials',
    'access-control-allow-origin',
    'access-control-request-method',
    'access-control-request-headers',
    'accept',
    'accept-encoding',
    'accept-language',
    'authorization',
    'connection',
    'content-type',
    'dnt',
    'credentials',
    'host',
    'origin',
    'user-agent',
    'X-CSRFToken',
    'csrftoken',
    'x-requested-with',
)

....
```



이렇게 함으로써, ajax 통신을 허용할 호스트와 credential 등을 설정 할 수 있다.







다음으로, index.html 에 있는 스크립트 태그에 가장 최상단에, 아래 코드를 추가 해주자.

```javascript
axios.defaults.xsrfCookieName = 'csrftoken'
axios.defaults.xsrfHeaderName = "X-CSRFToken"
axios.defaults.headers.common['X-CSRFToken'] = getCookie("csrftoken");
```



axios에서는, 위와 같은 방법으로 편리하게 ajax 통신시에 헤더값을 global하게 지정 해서 줄 수 있다.



참고로, 위와 같이 헤더이름을 설정 한 이유는, 장고 프레임워크에서 CSRF 헤더이름과 CSRF 쿠키의 이름이 위 처럼 지정되어 있기 때문이다.



이렇게 함으로써, axios를 이용해서 ajax 통신을 수행할때, CSRF TOKEN 을 헤더에 추가해서 ajax 통신을 수행 할 수 있다.









## 보너스(?) : 관리자 페이지 활성화



이번에는 보너스로, Django에서 기본적으로 관리하는 관리자 페이지 기능을 활성화 시켜보도록 하겠다.



먼저, 아래와 같은 명령어를 이용해서 관리자 페이지를 생성해준다.



```shell
python3 manage.py createsuperuser
```



그리고 나서, http://localhost:8000/admin 으로 접속을 해보자.



그러면 아래와 같이 관리자 화면이 나올것이다.







이렇게 해서, 랭킹 기능과 관리자 페이지 까지 모두 작성을 하였다.

다음 시간에는 마지막으로, 장고에서 static 파일을 작성하는 법에 대해서 다루어 보도록 하겠다.







## 출처



1. https://docs.djangoproject.com/en/3.0/ref/csrf/
2. https://vsupalov.com/avoid-csrf-errors-axios-django/
3. https://jinmay.github.io/2019/04/09/django/django-ajax-csrf/