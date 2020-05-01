## 소개



이번 포스팅에서는, 우리가 작성한 sudoku api를 pytest를 이용해서 테스트를 해보도록 하겠다.



아래글을 먼저 읽기 전에, pytest에 대한 기본 지식이 필요하므로 아래의 포스팅을 읽고 오도록 하자.

https://semtax.tistory.com/37





## 왜 테스트 코드를 작성하는가?



사실, 테스트 코드를 작성하지 않아도 프로그램을 개발 할 수 있다.

오히려 규모가 엄청 작은 프로젝트이거나, 한번 만들고 계속 고도화 할 프로젝트가 아닌 경우 유닛테스트가 불필요 하다.



하지만, 계속 프로젝트를 고도화 해야하는 서비스를 개발을 해야한다고 가정해보자.



이러한 테스트 코드가 없을 경우, 나중에 요구사항을 바꾼다던가, 비즈니스 로직을 바꿔야하는 경우가 생기게 된다.



이때, 위와 같이 기존 코드를 수정해야되는 일이 생기면, 코드 수정으로 인해 버그가 발생하고 이게 왜 버그가 나는지, 

어떤부분이 잘못되었는지를 파악을 못해서 결국 수정이 불가능한 폭탄이 되어 돌아오는 경우가 생기게 된다.



이러한 상황을 방지하기 위해 테스트 코드를 작성을 해두고, 이 테스트 코드를 안전망 삼아서 프로그램을 마음껏 수정 할 수 있다.



따라서, 프로그램이 커지고 나서 수정도 가능해지고, 다른 모듈간의 연계로 인한 버그를 잡는데에도 시간을 절약 할 수 있다.





##테스트 코드 작성



먼저 아래와 같이 test_sudokus.py 파일을 생성하자.



먼저, 정상적인 스도쿠 퍼즐을 넣었을때, 제대로 체크를 하는지를 확인하는 테스트코드를 작성해보자.



```python
from .sudokus import Sudoku

def test_check_method_if_true():
    sudokus = Sudoku()
    true_board = [
        [4,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2],
        [8,2,6,1,9,5,3,4,7],
        [3,7,4,6,8,2,9,1,5],
        [9,5,1,7,4,3,6,2,8],
        [5,1,9,3,2,6,8,7,4],
        [2,4,8,9,5,7,1,3,6],
        [7,6,3,4,1,8,2,5,9]
    ]
    assert sudokus.sudoku_check(true_board) == True
```



테스트를 돌려보면 아래와 같이 테스트를 통과했다고 나온다.



```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 1 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 1 passed in 0.03s ===================================================================
```







다음으로, 정답이 아닌 스도쿠 퍼즐을 넣었을때, 정상적으로 체크하는지에 대한 테스트코드를 작성해보자.



```python
from .sudokus import Sudoku

def test_check_method_if_false():
    sudokus = Sudoku()
    false_board = [
        [7,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2],
        [8,2,6,1,9,5,3,4,7],
        [3,7,4,6,8,2,9,1,5],
        [9,5,1,8,4,3,6,2,8],
        [5,1,9,3,2,6,8,7,4],
        [2,4,8,9,5,7,1,3,6],
        [7,6,3,4,1,8,2,5,9]
    ]
    assert sudokus.sudoku_check(false_board) == False
```





역시나, 테스트를 돌려보면 아래와 같이 테스트를 통과했다고 나온다.



```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 2 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 2 passed in 0.03s ===================================================================
```









다음으로, 정상적인 사이즈가 아닌 스도쿠 퍼즐을 주었을때에 대한 케이스도 테스트 해보자.



```python
from .sudokus import Sudoku

def test_invalid_size_check():
    sudokus = Sudoku()
    invalid_board = [
        [7,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2]
    ]
    assert sudokus.sudoku_check(invalid_board) == False
```





이번에도, 테스트를 돌려보면 아래와 같이 테스트를 통과했다고 나온다.



```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 3 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 3 passed in 0.03s ===================================================================
```





다행히, 3가지 경우 모두 테스트를 통과 하였다.



마지막으로, 혹시 모르니, 잘못된 케이스(리스트가 아닌경우)에 대해서도 처리를 하는지 확인을 해보자.



```python
from .sudokus import Sudoku

def test_invalid_board_int():
    sudokus = Sudoku()
    invalid_board = 1
    assert sudokus.sudoku_check(invalid_board) == False
  
def test_invalid_board_empty_list():
    sudokus = Sudoku()
    invalid_board = []
    assert sudokus.sudoku_check(invalid_board) == False
    
def test_invalid_board_float():
    sudokus = Sudoku()
    invalid_board = 0.1
    assert sudokus.sudoku_check(invalid_board) == False
    
def test_invalid_board_element_not_int():
    sudokus = Sudoku()
    invalid_board = [
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0]
    ]
    assert sudokus.sudoku_check(invalid_board) == False 
```





테스트 해보면 아래와 같은 결과가 나온다.





```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 7 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 7 passed in 0.03s ===================================================================
```







돌려보면 아래와 같이 테스트를 모두 통과했다는 결과가 나온다.



```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 7 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 7 passed in 0.03s ===================================================================
```







위의 모든 코드를 종합해보면 아래와 같다.



```python
from .sudokus import Sudoku

def test_check_method_if_true():
    sudokus = Sudoku()
    true_board = [
        [4,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2],
        [8,2,6,1,9,5,3,4,7],
        [3,7,4,6,8,2,9,1,5],
        [9,5,1,7,4,3,6,2,8],
        [5,1,9,3,2,6,8,7,4],
        [2,4,8,9,5,7,1,3,6],
        [7,6,3,4,1,8,2,5,9]
    ]
    assert sudokus.sudoku_check(true_board) == True

def test_check_method_if_false():
    sudokus = Sudoku()
    false_board = [
        [7,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2],
        [8,2,6,1,9,5,3,4,7],
        [3,7,4,6,8,2,9,1,5],
        [9,5,1,8,4,3,6,2,8],
        [5,1,9,3,2,6,8,7,4],
        [2,4,8,9,5,7,1,3,6],
        [7,6,3,4,1,8,2,5,9]
    ]
    assert sudokus.sudoku_check(false_board) == False

def test_invalid_size_check():
    sudokus = Sudoku()
    invalid_board = [
        [7,3,5,2,6,9,7,8,1],
        [6,8,2,5,7,1,4,9,3],
        [1,9,7,8,3,4,5,6,2]
    ]
    assert sudokus.sudoku_check(invalid_board) == False

def test_invalid_board_int():
    sudokus = Sudoku()
    invalid_board = 1
    assert sudokus.sudoku_check(invalid_board) == False
  
def test_invalid_board_empty_list():
    sudokus = Sudoku()
    invalid_board = []
    assert sudokus.sudoku_check(invalid_board) == False
    
def test_invalid_board_float():
    sudokus = Sudoku()
    invalid_board = 0.1
    assert sudokus.sudoku_check(invalid_board) == False
    
def test_invalid_board_element_not_int():
    sudokus = Sudoku()
    invalid_board = [
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0],
      [1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0]
    ]
    assert sudokus.sudoku_check(invalid_board) == False 
```



최종 테스트를 다시한번 돌려보자.



돌려보면 아래와 같이 테스트를 모두 통과했다는 결과가 나온다.



```shell
================================================================= test session starts ==================================================================
platform darwin -- Python 3.7.6, pytest-5.3.4, py-1.8.1, pluggy-0.13.1
rootdir: /Users/semtax/Desktop/django_study/DjangoSudoku/django_sudoku
plugins: celery-4.4.0
collected 7 items                                                                                                                                      

sudoku/api/test_sudoku.py .......                                                                                                                [100%]

================================================================== 7 passed in 0.03s ===================================================================
```

