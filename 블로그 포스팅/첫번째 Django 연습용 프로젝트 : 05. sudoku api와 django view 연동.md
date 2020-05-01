## 소개





이번 포스팅에서는, 지난 포스팅에서 작성한 sudoku api와 django view를 연동해보는 시간을 갖도록 하겠습니다.







## 코드 작성



먼저 sudoku 폴더의 views.py를 열어서, 아래와 같이 코드를 작성 하도록 하겠습니다.



먼저 views.py 의 make_sudoku 함수 입니다.



```python
from django.http import HttpResponse, JsonResponse
from .models import Ranking

import json

def index(request):
  return HttpResponse("index page")

def ranking(request):
  return HttpResponse("ranking view page")


def get_ranking_list(request):
  return HttpResponse("get ranking list page")


def register_ranking(request):
  return HttpResponse("register ranking page")

def make_sudoku(request):
    sudoku_api = Sudoku()
    board = sudoku_api.generate_sudoku()
    result = {
        'board' : board
    }

    return JsonResponse(result)
```



일단 make_sudoku 함수에서, 이전 포스팅에서 만든 generate_sudoku 함수를 이용해서 스도쿠 퍼즐을 생성 합니다.

그리고, result 변수를 선언해서, 반환할 스도쿠 퍼즐 데이터를 생성해줍니다.



마지막으로, JsonResponse 함수를 이용해서 json 형식으로 변환해서 JSON 데이터가 담긴 HTTP Response 을 리턴합니다.







다음으로는, views.py의 check_sudoku 함수 입니다.



```python
.....

def check_sudoku(request):
    sudoku_api = Sudoku()

    req_data = json.loads(request.body)

    puzzle = req_data['puzzle']
    elapsed_time = req_data['elapsed_time']

    result = sudoku_api.sudoku_check(puzzle)
    data = {}
    if result == True:
        data['status'] = "clear"
        request.session['status'] = "clear"
        request.session['elapsed_time'] = elapsed_time
    else:
        data['status'] = "fail"

    return JsonResponse(data)
```



check_sudoku 함수는, 사용자가 json형식 으로 데이터를 HTTP 프로토콜을 이용해서 보냅니다.

그리고 나서, 받은 데이터를 json.loads 함수를 이용해서 json -> 파이썬 dict 로 변환을 수행합니다.



그런 뒤, 변환한 dict타입의 req_data 변수에서 sudoku 퍼즐(puzzle)과, 푸는데 걸린 시간(elapsed_time)을 받습니다.



그 다음으로, 이전 포스팅에서 만든 sudoku.sudoku_check 함수에 puzzle 변수를 넘겨서, 

스도쿠 퍼즐이 정답인지 아닌지 체크합니다. 



그리고 나서, 정답인 경우, 반환 값인 data['status']에 클리어 했다고 저장을 한 뒤,

Django에서 지원하는 session에 경과 시간(elapsed_time)과 클리어 여부(status)를 저장합니다.





> 참고로, 장고(Django)에서는 request.session 변수를 이용해서, 세션(session) 값을 저장 합니다.
>
> 세션은, 일종의 서버에서 관리하는 임시 저장소이자, 출입표라고 생각하시면 됩니다.
>
> 일상 생활의 예시로는, 뷔페에서 붙여주는 출입증을 생각하면 편합니다 ㅎㅎ..



마지막으로, JsonResponse 함수를 이용해서 json 형식으로 변환해서 JSON 데이터가 담긴 HTTP Response 을 리턴합니다.





지금까지 설명한 코드를 모두 합친결과는 아래와 같습니다.



```python
from django.http import HttpResponse, JsonResponse
from .models import Ranking

import json

def index(request):
  return HttpResponse("index page")


def ranking(request):
  return HttpResponse("ranking view page")


def get_ranking_list(request):
  return HttpResponse("get ranking list page")


def register_ranking(request):
  return HttpResponse("register ranking page")


def check_sudoku(request):
    sudoku_api = Sudoku()

    req_data = json.loads(request.body)
    
    if 'puzzle' not in req_data or 'elapsed_time' not in req_data:
      JsonResponse({'status' : "fail"})

    puzzle = req_data['puzzle']
    elapsed_time = req_data['elapsed_time']

    result = sudoku_api.sudoku_check(puzzle)
    data = {}
    if result == True:
        data['status'] = "clear"
        request.session['status'] = "clear"
        request.session['elapsed_time'] = elapsed_time
    else:
        data['status'] = "fail"

    return JsonResponse(data)


def make_sudoku(request):
    sudoku_api = Sudoku()
    board = sudoku_api.generate_sudoku()
    result = {
        'board' : board
    }

    return JsonResponse(result)
```







## 실행





실행결과는 아래 그림과 같습니다.





<< 실행 결과 그림 post man 결과 보여주기..>>







이제 sudoku api 와 view까지 연동하였습니다. 하지만, 실제로 브라우저 화면에 보여지는 것은 너무 초라합니다.







다음 포스팅에서는, django의 템플릿과 html, css, javascript를 이용해서, 



실제로 브라우저에서 조금 더 보여지기 좋은 화면을 만드는 법에 대해서 다루도록 하겠습니다.

