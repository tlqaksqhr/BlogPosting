## 소개



이번 포스팅에서는, 지금까지 작성한 프로젝트에, django template를 생성하고 연동하겠습니다.



또한, javascript로 AJAX를 연동하는 시간도 역시 갖도록 하겠습니다.







## Django 템플릿?



Django에서 템플릿은, HTTP Response를 받아서, 사용자가 보는 화면을 그려주는 역할을 담당한다.

다시 말하면, 템플릿이라는 영어단어 의미 그대로, 출력되는 틀을 정해주는것을 말한다.



쉽게 생각해보면 붕어빵 틀 같은것을 생각해보면 된다.

대신 Django는 웹 프레임워크이기 때문에, 붕어빵 대신에, HTML이나, JSON을 만들어준다고 생각하면 된다.



또한, django에서는 이러한 view를 그려주는 템플릿 언어라는것을 지원한다.

그 중에서 특히 가장 많이 쓰는것이, [Jinja2](https://jinja.palletsprojects.com/en/2.11.x/) 엔진을 가장 많이 쓴다.



우리가 진행할 예제에서는, html 형식의 템플릿과, JSON 형식의 템플릿을 사용할 예정이다. 



다음으로는, Ajax에 대해서 알아보도록 하겠다.







## AJAX?





Ajax를 위키피디아에서 찾아보면, 아래와 같은 정의가 나온다.



>  **Ajax** (short for "Asynchronous [JavaScript](https://en.wikipedia.org/wiki/JavaScript) and [XML](https://en.wikipedia.org/wiki/XML)") is a set of [web development](https://en.wikipedia.org/wiki/Web_development) techniques using many web technologies on the [client side](https://en.wikipedia.org/wiki/Client_side) to create [asynchronous](https://en.wikipedia.org/wiki/Asynchronous_I/O) [web applications](https://en.wikipedia.org/wiki/Web_application). With Ajax, web applications can send and retrieve data from a [server](https://en.wikipedia.org/wiki/Web_server) asynchronously (in the background) without interfering with the display and behavior of the existing page



즉, 비동기 적으로 서버에 데이터를 요청해서 페이지를 따로 갱신하지 않아도, 부분적으로 페이지를 갱신할 수 있게 해주는 기술들을 의미한다.



그리고 비동기적으로 작동을 하므로, 보통 응답이 왔을때나, 에러가 발생했을때에 대한 콜백 함수(callback function)을 지정하고 해당 콜백함수에서 작업을 처리하는 방식으로 진행된다.



또한, AJAX로 데이터를 교환할때는 HTML, JSON, XML, 심지어 일반 텍스트도 주고 받을 수 있지만, 요즘에는 실질적으로 JSON 형식을 가장 많이 사용한다.





마지막으로, AJAX 같은 경우에는, 브라우저에 내장된 함수인 [XMLHttpRequest](https://en.wikipedia.org/wiki/XMLHttpRequest) 이 있기는 하지만, 실제로는 axios나 다른 웹 프론트엔드 프레임워크/라이브러리 에 있는 AJAX 라이브러리 들을 많이 사용한다.

(여담으로, 최근에는 fetch() 라는 내장 함수가 새로 생겨서 해당 함수를 쓰는 경우도 많이 있기는 하다.)





이제 설명이 어느정도 끝났으므로, 실제로 코드를 작성해보도록 하겠다. 





## 코드 작성



먼저, index페이지를 그리기 위해 views.py 에 코드를 추가 해보도록 하겠다.



아래와 같이 추가해주면 된다.



```python
from django.shortcuts import render
from django.http import HttpResponse, JsonResponse
from django.views import templates
from .api.sudokus import Sudoku

from django.core import serializers

import json
import datetime

from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def index(request):
    context = {}
    return render(request, 'sudoku/index.html',context)
```



django에서는 render 라는 함수를 이용해서, HTML 페이지를 그릴 수 있다.



첫 번째, 인자로는 HTTP 리퀘스트를 넘겨주고, 두번째 인자로는 html 파일의 경로, 마지막으로 context라는 인자를 넘겨준다.



저 context와 같은 경우, 위에서 언급한 템플릿 엔진에 넘겨 줄 변수들을 모아놓은 공간이라고 생각 하면 된다.



그리고, 저 html 파일의 경로와 같은 경우에는 "<앱 이름>/templates/" 경로에 있는 html파일을 자동으로 찾게 된다.

즉, 우리 프로젝트 같은 경우에는, "sudoku/templates/sudoku/index.html" 이 되는 것이다.





이제 해당 경로에 맞춰서 폴더를 만들어 주고 아래와 같이 html을 추가 해주도록 하자.



```html
<!DOCTYPE html>
<html>
<head>
    
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>
    
    <style>
      	// css file 작성하는 곳
    </style>
</head>
<body>

    <div id="main-flexbox">
        <div id="nav-bar-flexbox">
            <h3>WEB SUDOKU!</h3>
        </div>
        <div id="content-flexbox">
            <div id="dummy-flexbox"></div>
            <div id="sudoku-flexbox">
                <div class="gridbox-container">
                </div>
            </div>
            <div id="button-flexbox">
                <div>
                    <a id="submit-btn" class="btn btn-primary" role="button">SUBMIT</a>
                </div>
                <div id="timer-box">
                    <h3>00:00:00</h3>
                </div>
            </div>
        </div>
        <div id="fail-toast-container" style="position: absolute; top: 40%;">
            <div id="fail-toast" class="toast">
                <div class="toast-header">
                    <strong class="mr-auto text-danger">제출한 답안이 틀렸습니다!</strong>
                </div>
                <div class="toast-body text-danger">
                    제출한 답안이 틀렸습니다!
                </div>
            </div>
        </div>

        <div id="success-toast-container" style="position: absolute; top: 40%;">
            <div id="success-toast" class="toast" data-autohide="false">
                <div class="toast-header">
                    <strong class="mr-auto text-primary">축하합니다!</strong>
                    <button type="button" class="ml-2 mb-1 close" data-dismiss="toast">&times;</button>
                </div>
                <div class="toast-body text-danger">
                    <div>
                    	<strong class="mr-auto text-primary">축하합니다!</strong>
                    </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script type="text/javascript">
			// 여기에 자바스크립트 코드를 작성하시기 바랍니다.
    </script>
</body>
</html>
```





다음으로 주석에 적혀 있는대로, style태그 안에 아래와 같이 css를 작성 해보도록 하자.



```css
.gridbox-container {
  display: grid;
  grid-template-columns: repeat(9, 3em);
}

.gridbox-item {
  background-color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 1);
  padding: 1em;
  font-size: 1em;
  text-align: center;
  font-weight: bold;
}

.input-item-decorator {
  color: rgba(136,137,220,1);
}

.fixed-item-decorator {
  color: rgba(0,0,0,1);
}

.cell-padding-left {
  margin-left: 0.2em;
}

.cell-padding-bottom {
  margin-bottom: 0.2em;
}

#main-flexbox{
  display: flex;
  flex-flow: column;
  align-items: center;
  height: 100%;
}

#content-flexbox{
  display: flex;
  flex: 1;
  width: 100%;
  flex-flow: row;
}

#dummy-flexbox {
  display: flex;
  flex: 2;
  flex-flow: column;
  justify-content: center;
  align-items: center;
}

#sudoku-flexbox {
  flex: 4;
  flex-flow: row;
  height: 100%;
}

#sudoku-flexbox > .gridbox-container{
  justify-content: center;
}

#button-flexbox {
  display: flex;
  flex: 2;
  flex-flow: column;
  align-items: center;
  justify-content: space-between;
  height: 100%;
}

.btn-primary, .btn-primary:hover, .btn-primary:active, .btn-primary:visited, .btn-primary:focus {
  width: 8em;
  background-color: rgba(200, 177, 232, 1);
  border-color: rgba(200, 177, 232, 1);
}

#submit-btn {
  margin-bottom: 2em;
}

#main-flexbox {
  margin-top: 1em;
}

#nav-bar-flexbox {
  margin-bottom: 1em;
}

#fail-toast {
  position: relative;
}
```







마지막으로, 주석에 적혀있는대로, 맨 아래 있는 스크립트 태그 안에, 자바스크립트 코드를 작성 해보도록 하자.



```javascript
 axios.get('/sudoku/make')
  .then(make_sudoku);

let timer_textbox = document.querySelector("#timer-box > h3");
var elapsed_time = 0;

$('#success-toast-container').hide();
$('#fail-toast-container').hide();

let timer = setInterval(function(){
  elapsed_time+=1;

  let seconds = parseInt(elapsed_time%60);
  let minutes = parseInt((elapsed_time%3600)/60);
  let hours = parseInt(elapsed_time/3600);

  seconds = seconds.toString().padStart(2, '0');
  minutes = minutes.toString().padStart(2, '0');
  hours = hours.toString().padStart(2, '0');

  timer_textbox.innerText = `${hours}:${minutes}:${seconds}`;

},1000);

let button_flexbox = document.querySelector("#submit-btn");

button_flexbox.addEventListener("click",function(){
  axios.post('/sudoku/check',
             get_sudoku_data()
            ).then(process_result);
});

function get_sudoku_data() {
  let data = {
    'elapsed_time':elapsed_time,
  }

  let grid_container = document.querySelector(".gridbox-container");
  let board = [];

  for(let i=0;i<9;i++){
    for(let j=0;j<9;j++){
      let k = i*9 + j;
      let cell = grid_container.children[k];
      if(k%9==0){ 
        board.push([]);
      }
      board[i].push(Number(cell.innerText));
    }
  }

  data['puzzle'] = board;

  return data;
}

function process_result(response) {
  let status = response.data.status;
  clearInterval(timer);

  if(status == 'clear'){
    $('#success-toast-container').show();
    $('#success-toast').toast('show');

  }else{
    $('#fail-toast-container').show();
    $('#fail-toast').toast('show');
  }
}

function make_sudoku(response){
  let board = response.data.board;
  let grid_container = document.querySelector(".gridbox-container");
  for(let i=0;i<9;i++){
    for(let j=0;j<9;j++){
      let node = document.createElement("div");
      node.classList.add("gridbox-item")

      if(i!=0 && i%3==2)
        node.classList.add("cell-padding-bottom");

      if(j!=0 && j%3==0)
        node.classList.add("cell-padding-left");

      if(board[i][j]==0){      
        node.contentEditable = true;
        node.innerText = "";
        node.classList.add("input-item-decorator");
      }else{
        node.innerText = board[i][j].toString();
        node.classList.add("fixed-item-decorator");
      }
      grid_container.appendChild(node);
    }
  }
}
```



이번 프로젝트에서는 [axios](https://github.com/axios/axios) 라는 라이브러리를 이용해서 ajax 통신을 수행하였다.



위의 자바스크립트에서 하는일은 대략적으로 아래와 같다.



1. Sudoku 퍼즐을 가져와서 9*9 사각형에 배열
2. 제출 버튼을 눌렀을때, sudoku 퍼즐과, 푸는데 걸리는 시간을 /sudoku/check에 제출
3. 정답인 경우, 축하메시지 창이, 오답인 경우 다시풀라는 창을 띄우는 기능
4. 시간 카운팅(초 단위로 세어 줌)





## 실행 결과



실행 결과는 아래 사진과 같다.









이번 시간까지 한 내용만 있으면, 사실 기본적인 목적(스도쿠 게임을 제공)은 달성하였다. 

하지만, 스도쿠 게임 "만" 플레이 하게 하면 약간 재미가 없다. 



따라서, 다음 포스팅에서는 랭킹 기능을 추가해보는 시간을 가지도록 하겠다.