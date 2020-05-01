

### Expressjs에서 JSON Request Body 파싱하기



expressjs에서 웹 서비스를 제작 했을때, json으로 이루어진 Request Body를 받았을 경우, 요청값을 제대로 받아오지 못하는 문제가 발생한다.

expressjs에서는 이러한 문제를 해결하는 방법으로 크게 2가지 방법을 사용할 수 있다.

1. body-parser 모듈 사용(4.16 이전 버전).
2. express.json() 사용



#### 1. Express 4.x ~ 4.16 이전 버전인 경우(body-parser 사용)

expressjs 4.16 이전 버전에서는 위와 같은 문제를 해결하기 위해 body-parser라는 외부 모듈을 사용해야 한다.



설치방법은 아래와 같다

```shell
npm install body-parser
```



설치한 모듈은 아래와 같이 사용하면 된다.

```javascript
const express = require('express')
const bodyParser = require('body-parser');

const app = express();

app.use(bodyParser.json());

app.post('/', function(req, res){
  console.log(req.body);      // 사용자의 JSON 요청
	res.send(req.body);    // JSON 응답
});

app.listen(3000);
```

실행을 해보면 결과를 제대로 받아오는것을 확인 할 수 있다.



#### 2. Express 4.16 이후 버전인 경우

```javascript
const express = require("express");
const app = express();

app.use(express.json());

app.post('/', function(req, res){
  console.log(req.body);      // 사용자의 JSON 요청
	res.send(req.body);    // JSON 응답
});

app.listen(3000);
```

4.16 이후 부터는, expresses.json() 모듈을 사용하면 json요청을 제대로 받을 수 있다.