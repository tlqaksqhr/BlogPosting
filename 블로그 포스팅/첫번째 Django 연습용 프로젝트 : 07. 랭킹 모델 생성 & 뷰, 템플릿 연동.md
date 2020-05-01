## 소개



이번 포스팅에서는, 지금까지 만든 스도쿠 게임에 랭킹 기능을 추가 해보도록 하겠다.



또한, 프로그램이 재시작 되더라도, 랭킹 기록이 계속 남게 하기 위해

데이터 베이스를 이용하여 랭킹기록을 저장하는 것도 다루도록 하겠다.





## 데이터 베이스?



데이터 베이스는 쉽게 말하면 데이터를 저장하는 공간이다.



데이터를 어디다 저장하는데? 라고 물어 보면, 아래와 같은 대답을 해줄 수 있다.

1. 메모리
2. 파일
   1. 로컬파일
   2. 네트워크하고 자기 PC에 분산처리
3. JSON
4. 표 형식으로
5. XML
6. 그래프(?!)



저 위에서 언급한 종류 모두 실제로 회사에서 사용하는 데이터베이스 종류이다.



게다가 특히, 가장 많이 쓰이는 데이터 베이스는 관계형 데이터베이스(RDBMS) 라는 종류의 데이터 베이스 이다.



관계형 데이터베이스 라고 하면 어려워 보이지만, 사실 관계형 데이터베이스와 유사한 구조의 문서를 액셀이라는 이름으로 다루고있다. 



(물론, 액셀과는 다르게 여러 사용자가 접근했을때의 경우에 대한 처리나, 데이터 중복 이상, 데이터 삽입/갱신 이상, 샤딩, 액셀보다 훨씬 다양한 Aggregation 연산 등과 같은 훨씬 다양한 기능을 제공한다.)



또한, 데이터베이스에 값을 넣고, 삭제하고, 조회하고 수정하는 경우, 

그냥 동네가게에서 콩나물 사듯이 "이거 주세요" 라고는 할 수 없으니, 데이터를 조작/조회 하는 일종의 규칙이 있어야 한다.



이러한 규칙을 RDBMS에서는 보통 SQL이라고 한다.



하지만, 이미 웹 개발을 위해서 프로그래밍 언어를 배우는데도 바쁜 상황인데, SQL문을 또 배워야 한다.



또한, SQL문을 이미 알고있다고 치더라도, 한 프로젝트에 SQL문 + 기존 언어 와 같이 2가지 이상의 언어가 혼재되어, 유지보수도 힘들다.



마지막으로, Django 모델은 객체(Object) 기반인데, RDBMS는 테이블(Relation) 기반인 문제가 있다.



즉, 비즈니스 로직이 RDBMS에 종속되어, 나중에 데이터베이스를 교체해야한다거나, 

개발 중인 프레임워크를 바꿔야 되는 경우 둘 다 뜯어고쳐야 되는 최악의 상황이 발생하게 된다.



Django에서는 이러한 문제를 해결할 방법이 없는것인가?





##Django ORM



다행히, 장고(Django)에서는, 이러한 데이터베이스와 객체간의 불일치를 해결 해주고, 

따로 쿼리문을 쓰지않고 데이터베이스를 좀 더 편하게 다룰 수 있게 해주기 위한, 

ORM(Object Relation Mapping) 라이브러리를 제공한다.



ORM 에 대한 자세한 내용은 아래의 주소를 참조하기를 바란다. (JPA 설명과 ORM에 대한 정의를 간략하게 정리하였다.)

https://semtax.tistory.com/29?category=804335



먼저 테이블 정의를 아래와 같이 간단하게 정의 할 수 있다.



```python
from django.db import models

class Post(models.Model):
    author = models.CharField(max_length=255)
    title = models.CharField(max_length=255)
    text = models.TextField()  
```





예를 들어, 모든 게시물을 가지고 오는 것을 아래와 같이 간단하게 할 수 있다.



```python
from blog.models import Post

Post.objects.all()
```



또한, 데이터 베이스에 게시물을 저장하거나 수정/삭제 하는 것도 아래와 같이 편하게 할 수 있다.



```python
>>> Post.objects.create(author=me, title='Sample title', text='Test')
```





## 구현



먼저 Django에서 ORM을 사용하기 위해서는 모델을 등록 해주어야 한다.



sudoku/models.py 파일을 열어서 아래와 같이 작성을 해주도록 하자.



```python
from django.db import models

class Ranking(models.Model):
    name = models.CharField(max_length=255)
    elapsed_time = models.TimeField()
```



코드를 설명하기 전에 이러한 스도쿠 게임에서 랭킹을 남기기위해 필요한 정보를 생각해보면 아래와 같다.

1. 이름
2. 걸린시간



위의 코드는 위에서 언급한 2가지 정보를 Django ORM 형태로 작성을 한 것이다.



먼저 이름은 CharField 라는 자료구조를 이용해서 작성을 하였다.

해당 자료구조의 특징은, 길이를 지정할 수 있다는 것이다. 

그리고 데이터베이스로 변환할때는 VARCHAR(MAX_LENGTH) 자료로 변환이 되게 된다.



다음으로, 걸린 시간은 TimeField를 이용해서 정의 할 수 있다.

Django에서는 TimeField를 이용해서 HH:MM:SS 형태의 시간을 저장할 수 있다.





다음으로 views.py 를 열어서 아래와 같이 코드를 작성해준다.



```python
from django.shortcuts import render
from django.http import HttpResponse, JsonResponse
from django.views import templates
from sudoku.models import Ranking
from .api.sudokus import Sudoku

from .models import Ranking

import json
import datetime


....


def ranking(request):
    context = {}
    return render(request, 'sudoku/ranking.html',context)
  
  

def get_ranking_list(request):
    ranking_list = Ranking.objects.order_by('elapsed_time')[:100]
    ranking_data = serializers.serialize("json",ranking_list, fields=('name','elapsed_time'))
    ranking_data = json.loads(ranking_data)
    ranking_data = [{**item['fields'],**{"pk" : item['pk']}} for item in ranking_data]
    ranking_data = {
        "data" : ranking_data
    }
    return JsonResponse(ranking_data)

  

def register_ranking(request):
    if 'elapsed_time' not in request.session:
        return JsonResponse({'status' : 'failed'})

    data = json.loads(request.body)
    name = data['name']
    elapsed_time = request.session['elapsed_time']

    datetime_args = elapsed_time//3600,(elapsed_time%3600)//60,elapsed_time%60
    d = datetime.time(datetime_args[0], datetime_args[1], datetime_args[2]) 

    ranking = Ranking(name = name, elapsed_time = d)
    ranking.save()

    return JsonResponse({'status' : 'success'})
  
...
```



views.py 에서 구현한 view는 아래와 같다.



1. 랭킹 등록
2. 랭킹 조회(상위 100명)
3. 랭킹 페이지 





그런 뒤, 랭킹 템플릿 페이지인 ranking.html를 아래와 같이 작성 해주자.



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
    </style>
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

    <script type="text/javascript">
        axios.get('/sudoku/ranking')
        .then(get_sudoku_ranking);

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
    </script>
</body>
```





마지막으로, index.html을 열어서 아래와 같이 수정 해주도록 하자.



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

    <script type="text/javascript">
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
    </script>
</body>
</html>
```



이제 코드 작성이 완료되었으면, 실제 데이터 베이스에 우리가 작성한 모델을 반영시켜주기만 하면 된다.





먼저, manage.py 가 있는 django_sudoku 폴더에 들어가서 

아래 명령어를 이용해서 데이터베이스 마이그레이션을 작성해준다.



```shell
$ python3 manage.py makemigrations
```





다음으로, 작성한 마이그레이션을 실제 데이터베이스에 전송할 SQL문으로 바꾸 줄 아래 명령어를 작성 해주자.

```shell
$ python3 manage.py sqlmigrate 0001
```



위 명령어를 이용해서, 실제 데이터베이스에 테이블을 생성하겠다는 SQL문을 생성시킬 수 있다. 

(참고로 위의 0001은 마이그레이션 번호 혹은 마이그레이션 레이블 이라고 불리는 것이다)



데이터 베이스 마이그레이션은, 나중에 데이터베이스의 스키마(즉 데이터베이스 테이블의 구조)가 바뀌었을때, 

이렇게 수정하면 된다고 알려주는 일종의 가이드 혹은 단서라고 알고있으면 된다.



마지막으로, 아래 명령어를 이용해서 실제 데이터베이스에 우리가 작성한 모델을 반영 시켜주자.

```shell
$ python3 manage.py migrate
```





만약, 생성한 데이터베이스를 테스트 해보고 싶으면 아래명령어를 이용해서 Django Shell에서 테스트가 가능하다.



```shell
$ python3 manage.py shell
```





## 실행 결과





이제, 스도쿠를 열심히 플레이하고 제출 버튼을 눌러서, 랭킹페이지에 들어가보면 아래와 같은 결과를 확인 할 수 있다.









이제 랭킹기능도 대략적으로 구현이 완료되었다.



다음 포스팅에는 CSRF와 Django Admin에 대해서 다루어 보도록 하겠다.