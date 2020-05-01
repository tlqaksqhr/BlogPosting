## 소개



이번 포스팅에서는, 장고 프레임워크에서 static 파일들(css, js)의 경로를 지정하는 법에 대해서 알아보도록 하겠다.



## Static File?



스태틱 파일(Static File)은, 말 그대로 변하지 않는 데이터들을 의미한다.



대표적인 스태틱 파일들로는, 사진, 동영상, css, js 등과 같은 자료들이 있다.



사실 사진이나 동영상 같은 경우에는 static 파일이기는 하지만 크기나 너무 크므로, 

이러한 파일들을 저장하는데 특화된 CDN(Content Delivery Network) 서비스들을 이용한다.



물론, 요즘에는 css나 js들도 CDN에 보관하는 경우가 매우 많다.

실제로, 우리가 작성한 예제에서 임포트한 부트스트랩관련 라이브러리, 파일들은 모두 CDN을 통해서 가져왔다.





## 장고(Django) 에서의 Static 파일 처리





장고에서는 아래와 같이 간단하게 static 파일을 import 해올 수 있다.



```html
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'sudoku/index.css' %}" />
```



이제 그럼 예제를 작성해보도록 하겠다.





## 예제



sudoku 폴더 내에, static 폴더를 생성하고, static 폴더안에 다시 sudoku 폴더를 생성해주자.



그 다음으로, 생성한 sudoku 폴더안에, 각각 index.js, index.css, ranking.js, ranking.css를 생성 해 주자.



먼저, index.html을 아래와 같이 작성해주자.



```html
<!DOCTYPE html>
<html>
<head>

    {% load static %}
    
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

    <link rel="stylesheet" type="text/css" href="{% static 'sudoku/index.css' %}" />
    
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
                <div>
                    <a id="ranking-btn" class="btn btn-primary" role="button" href="ranking">랭킹 페이지</a>
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
                    <strong class="mr-auto text-primary">축하합니다 랭킹 등록 하시죠!</strong>
                    <button type="button" class="ml-2 mb-1 close" data-dismiss="toast">&times;</button>
                </div>
                <div class="toast-body text-danger">
                    <div>
                        <strong class="mr-auto text-primary">아이디 : </strong>
                        <input id="ranking_username_textbox" type="text" />
                    </div>
                    <div style="display: flex; height: 4em; flex-flow: row; justify-content: center; align-items: center; flex-wrap: nowrap;">
                        <a id="ranking_submit" type="button" class="btn text-primary btn-primary">랭킹 등록하기!</a>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script type="text/javascript" src="{% static 'sudoku/index.js' %}"></script>
    <script type="text/javascript">
        init();
    </script>
</body>
</html>
```



그런 다음, index.css를 아래와 같이 작성해주자.



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

#ranking-btn {
    margin-top: 1em;
    margin-bottom: 1em;
}

#ranking-submit {
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







다음으로 index.js를 아래와 같이 작성 해주자.



```javascript
elapsed_time = 0;

timer = setInterval(function(){
    elapsed_time+=1;

    let timer_textbox = document.querySelector("#timer-box > h3");

    let seconds = parseInt(elapsed_time%60);
    let minutes = parseInt((elapsed_time%3600)/60);
    let hours = parseInt(elapsed_time/3600);

    seconds = seconds.toString().padStart(2, '0');
    minutes = minutes.toString().padStart(2, '0');
    hours = hours.toString().padStart(2, '0');

    timer_textbox.innerText = `${hours}:${minutes}:${seconds}`;

},1000);

function init(){

        axios.defaults.xsrfCookieName = 'csrftoken'
        axios.defaults.xsrfHeaderName = "X-CSRFToken"
        axios.defaults.headers.common['X-CSRFToken'] = getCookie("csrftoken");

        axios.get('/sudoku/make')
            .then(make_sudoku);

        $('#success-toast-container').hide();
        $('#fail-toast-container').hide();

        
        let button_flexbox = document.querySelector("#submit-btn");
        let ranking_button = document.querySelector("#ranking_submit");

        button_flexbox.addEventListener("click",function(){
            axios.post('/sudoku/check',
                get_sudoku_data()
            ).then(process_result);
        });

        ranking_button.addEventListener("click", function(){
            let name = document.querySelector("#ranking_username_textbox").value;
            submit_ranking(name);
        });
}



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

function submit_ranking(name){
    axios.post("/sudoku/ranking/register",{
        'name' : name
    }).then(function(response){
        let status = response.data;
        if(status == 'success'){
            location.href = "/ranking";
        }
        $('#success-toast-container').hide();
    });
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

function getCookie(c_name)
{
    if (document.cookie.length > 0)
    {
        c_start = document.cookie.indexOf(c_name + "=");
        if (c_start != -1)
        {
            c_start = c_start + c_name.length + 1;
            c_end = document.cookie.indexOf(";", c_start);
            if (c_end == -1) c_end = document.cookie.length;
            return unescape(document.cookie.substring(c_start,c_end));
        }
    }
    return "";
}
```



다음으로 ranking.html을 아래와 같이 작성해주자.



```html
<!DOCTYPE html>
<html>
<head>

    {% load static %}
    
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">

    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

    <link rel="stylesheet" type="text/css" href="{% static 'sudoku/ranking.css' %}" />
</head>
<body>
    <div id="main-flexbox">
        <div id="nav-bar-flexbox">
            <h3>WEB SUDOKU RANKING!</h3>
        </div>
        <div id="content-flexbox">
            <div id="dummy-flexbox">
            </div>
            <table id="ranking-table" class="table">
                <thead>
                    <tr>
                        <th>등수</th>
                        <th>아이디</th>
                        <th>경과 시간</th>
                    </tr>
                </thead>
                <tbody id="ranking-list">
                </tbody>
            </table>
            <div id="button-flexbox">
                <a id="ranking-btn" class="btn btn-primary" role="button" href="/">메인 페이지</a>
            </div>
        </div>
    </div>

    <script type="text/javascript" src="{% static 'sudoku/ranking.js' %}"> </script>
    <script type="text/javascript">
        init();
    </script>
</body>
```





다음으로 ranking.js를 아래와 같이 작성 해 주자.



```javascript
function init(){
    axios.get('/sudoku/ranking')
    .then(get_sudoku_ranking);
}

function get_sudoku_ranking(response){
    let ranking_data = response.data.data;
    let ranking_list = document.querySelector("#ranking-list")
    let count = 1;
    ranking_data.forEach(element => {
        let tr = document.createElement("tr");
        let rankElement = document.createElement("td");
        let userElement = document.createElement("td");
        let timeElement = document.createElement("td");

        rankElement.innerText = count.toString();
        userElement.innerText = element.name;
        timeElement.innerText = element.elapsed_time;
        tr.appendChild(rankElement);
        tr.appendChild(userElement);
        tr.appendChild(timeElement);
        ranking_list.appendChild(tr);
        count += 1;
    });
}
```



마지막으로, ranking.css를 아래와 같이 작성해주자.



```css
#main-flexbox{
  display: flex;
  flex-flow: column;
  align-items: center;
  height: 100%;
}

#content-flexbox{
  display: flex;
  width: 70%;
  flex-flow: row;
}


#dummy-flexbox {
  display: flex;
  width: 15%;
  flex-flow: column;
}

#button-flexbox{
  display: flex;
  width: 20%;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  height: 100%;
}

#main-flexbox {
  margin-top: 1em;
}
```





## 실행



실행을 해보면 이전 포스팅과 같이 정상적으로 작동되는것을 확인 할 수 있다.





이로써, 그럭저럭(?) 돌아가는 웹 기반의 스도쿠 게임을 제작 해보았다.



사실, 여기서 다룬 django내용이 전부는 아니다. Django 프레임워크에는 여기서 다룬것보다 훨씬 더 많은 것들이 들어있다.



더 추가 해볼만한 기능들을 숙제느낌으로 아래에다 적어놓도록 하겠다. 



Django를 공부하면서 아래 요구사항들을 추가해보기 바란다.



1. 스테이지 기능 추가
2. 회원가입/로그인 기능
3. 사용자 별 레벨 기능
4. 아이템 판매 & 구입
5. 경험치 / 돈 개념 추가



특히 3~5 번 기능들을 추가로 구현하는 경우, 여러 테이블에 값을 조회하고 생성/수정 하는걸 

중간에 다른 연산이 끼어들지 않고, 한번에 처리가 되야하므로 DB 트랜잭션을 써야 하므로

Django를 이용해서 데이터베이스 트랜젝션을 연습하는데 좋은 밑거름이 된다고 생각을 한다.





이로서 해당 시리즈를 마치도록 하겠다.



지금 까지 봐주신분들 모두 감사합니다..