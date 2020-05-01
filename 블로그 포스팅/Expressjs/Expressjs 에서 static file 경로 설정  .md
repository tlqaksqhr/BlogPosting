# Expressjs 에서 static file 경로 설정



## 개요



이번 문서에서는, expressjs에서 static file 경로를 설정해주는 방법을 알아보도록 하자.

Static 파일들은 클라이언트에서 동작하는 자바스크립트나 css, html과 같은 리소스 파일들을 지칭한다.

이러한 파일들은 단어뜻 그대로 변하지 않는 데이터이므로 따로 관리를 해주는 것이다.



## Static 파일 설정하기



express.js 에서는 아래와 같이 static 파일 경로를 설정해 줄 수 있다.



```javascript
app.use(express.static('public'));
```



위와 같이 설정한 경우, 아래 URL로 접근하면 static파일에 접근이 가능하다.

```javascript
http://localhost:3000/images/kitten.jpg
http://localhost:3000/css/style.css
http://localhost:3000/js/app.js
http://localhost:3000/images/bg.png
http://localhost:3000/hello.html
```





아래와 같이 static 파일 경로를 여러개 설정할수도 있다.



```javascript
app.use(express.static('public'))
app.use(express.static('files'))
```



expressjs 라우팅 기능과 함께 사용도 가능하다

```javascript
app.use('/static', express.static('public'));
```



지금까지 설정한 static 파일경로는,  소스파일이 위치한 경로를 기준으로 하기 때문에 사용자가 원하는 경로에 static파일을 지정해주고 싶을 수도 있다. 



다음과 같은 방식으로 경로 설정이 가능하다.

```javascript
app.use('/static', express.static(path.join(__dirname, 'public')));
```

