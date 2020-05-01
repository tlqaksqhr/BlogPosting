## 소개



이번 포스팅에서는, 실제로 sudoku 퍼즐을 생성하고 검증하는 api를 작성 해보도록 하겠다.





## Sudoku & 필요한 기능 명세



스도쿠는, 9*9 격자에 숫자를 채워넣는 게임이다.



단 조건이 있는데, 각 칸에 해당하는 행/열/3*3 사각형에 1~9 까지의 숫자가 단 1개씩만 들어가 있어야 한다는 조건이다.



보통 이러한 스도쿠는 백트래킹 알고리즘을 이용해서 풀 수 있다.



백트래킹 알고리즘에 대한 자세한 내용은 아래 글을 참조해주기를 바란다.

https://semtax.tistory.com/50



일단 우리가 필요한 기능은 아래 표와 같다.



| 기능                  | 설명                                     |
| --------------------- | ---------------------------------------- |
| 스도쿠 퍼즐 생성      | 스도쿠 퍼즐을 생성해주는 기능            |
| 스도쿠 퍼즐 정답 검증 | 스도쿠 퍼즐을 정확하게 풀었는지 검증하기 |



이제 기능명세가 대충 끝났으니 코드를 작성 해보도록 하자.



## 코드 작성



먼저 sudoku 폴더에 api폴더를 만들고 api폴더 안에, sudokus.py 파일을 작성을 해보도록 하겠다.



먼저 생성자 함수를 아래와 같이 작성 해주자.



```python
import random

class Sudoku():    
  def __init__(self):
        self.SIZE = 9
        self.origin_board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.row = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.col = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.diag = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.terminate_flag = False
        self.__board_init()
        ....
```



위에서 언급한 sudoku 포스팅을 보면 알겠지만, 



스도쿠 생성 및 정답 체크를 하기 위해 i행 j열에 어떤 숫자들이 채워졌는지를 체크하는 row, col 2차원 리스트와, 



몇번째 3*3 사각형에 어떤 숫자들이 채워졌는지를 체크하는 2차원 리스트 변수 diag를 선언한 것을 알 수 있다.





또한, 스도쿠 보드를 만드는것이므로, board라는 2차원 리스트를 선언해주었고, 복사를 위한 임시 변수인 origin_board, 



그리고  보드 크기를 저장하는 변수인 SIZE 변수, 마지막으로 DFS 기반의 백트래킹을 이용해서 스도쿠 생성을 하므로, 



단 1개의 스도쿠 퍼즐만 생성하고 종료하기 위한 종료플래그 변수, terminate_flag를 선언하였다.





다음으로, __board_init() 함수를 작성해보도록 하겠다.



```python
import random

class Sudoku():  
    ....
  	def __board_init(self):
        seq_diag = [0,4,8]
        for offset in range(0,9,3):
            seq = [i for i in range(1,self.SIZE+1)]
            random.shuffle(seq)
            for idx in range(0,9):
                i,j = idx//3,idx%3
                self.row[offset+i][seq[idx]] = 1
                self.col[offset+j][seq[idx]] = 1
                k = seq_diag[offset//3]
                self.diag[k][seq[idx]] = 1
                self.origin_board[offset+i][offset+j] = seq[idx]
```



해당 함수는, 실제로 init함수에서 선언한 클래스 멤버들을 초기화 해주는 함수이다.



__board_init 함수에서는, 보드 생성시 처음부터 모든 보드를 다 만드는것 보다는, 일부분을 채워넣고 나머지를 백트래킹으로 생성하는 것이 효율적이기 때문에, 해당부분도 추가적으로 처리를 해주게 된다.



해당 코드에서는, 1번째, 4번째, 9번째 3*3 사각형을 미리 채워주는 역할을 수행한다.



다음으로는, 스도쿠 생성이 끝나고 나서, 인스턴스 변수들을 초기화 해주는 __clean() 함수이다.



```python
import random

class Sudoku():
   ....    
   def __clean(self):
        self.origin_board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.row = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.col = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.diag = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.terminate_flag = False
```



다음으로는, 실제로 스도쿠 퍼즐을 생성해주는 __make_sudoku() 함수이다.



```python
import random

class Sudoku():
    ....   
    def __make_sudoku(self, k):

        if self.terminate_flag == True:
            return True

        board_size = self.SIZE*self.SIZE
        if k >= board_size :
            for i in range(0,self.SIZE):
                for j in range(0,self.SIZE):
                    self.board[i][j] = self.origin_board[i][j]
            self.terminate_flag = True
            return True

        i,j = k//self.SIZE,k%self.SIZE
        start_num = random.randint(1,self.SIZE)

        if self.origin_board[i][j] != 0:
            self.__make_sudoku(k+1)

        for num in range(1,self.SIZE+1):
            num = 1 + (num + start_num)%9
            d = (i//3)*3 + (j//3)
            if self.row[i][num] == 0 and self.col[j][num] == 0 and self.diag[d][num] == 0:
                self.row[i][num],self.col[j][num],self.diag[d][num] = 1,1,1
                self.origin_board[i][j] = num
                self.__make_sudoku(k+1)
                self.row[i][num],self.col[j][num],self.diag[d][num] = 0,0,0
                self.origin_board[i][j] = 0
```



위의 코드는 백트래킹을 이용해서, 미리 채워놓은 부분을 제외하고, 나머지 부분을 채워넣는 코드이다.

또한, 정답이 완전 일정하면 재미가 없고 의미도 없으므로, 숫자도 랜덤생성 해주도록 작성하였다.



다음으로는, 실제 sudoku 보드가 정답인지 아닌지를 체크하는 __sudoku_check 함수이다.



```python
import random

class Sudoku():
    ....       
    def __sudoku_check(self, puzzle):
        
        if type(puzzle) != type([]) or len(puzzle) == 0:
            return False

        row_size = len(puzzle)
        col_size = min([len(puzzle[i]) for i in range(0,row_size-1)])

        if col_size != self.SIZE or row_size != self.SIZE:
            return False

        for i in range(0,self.SIZE):
            for j in range(0,self.SIZE):
                if type(puzzle[i][j]) != type(1):
                    return False
        
        for i in range(0,self.SIZE):
            for j in range(0,self.SIZE):
                k = (i//3)*3 + (j//3)
                num = puzzle[i][j]
                self.row[i][num]+=1
                self.col[j][num]+=1
                self.diag[k][num]+=1
        
        for idx in range(0,self.SIZE):
            for num in range(1,self.SIZE+1):
                if self.row[idx][num]!=1 or self.col[idx][num]!=1 or self.diag[idx][num]!=1:
                    return False
        
        return True
```



위의 check함수에서는, 스도쿠가 정답 여부 인지 뿐만이 아니라, 해당 퍼즐에 잘못된 데이터가 들어갔는지도 같이 체크하게 된다.

(Ex: 숫자가 아닌값을 넣은 경우, 보드 사이즈가 맞지 않은 경우, list가 아닌 다른 형식을 넣은 경우.)



마지막으로, 실제 api에 해당하는 public 함수들을 작성해보도록 하겠다.



```python
import random

class Sudoku():
    ....        
    def __generate_puzzle(self):
        self.__clean()
        self.__make_sudoku(0)
        return self.board

    def generate_sudoku(self):
        puzzle = self.__generate_puzzle()
        count = random.randint(15,40)
        remove_coords = {}
        for i in range(0,count):
            coord = random.randint(0,self.SIZE-1),random.randint(0,self.SIZE-1)
            while coord not in remove_coords:
                coord = random.randint(0,self.SIZE-1),random.randint(0,self.SIZE-1)
                remove_coords[coord] = coord
        
        for coord in remove_coords:
            di,dj = coord[0],coord[1]
            puzzle[di][dj] = 0
        return puzzle
    
    def sudoku_check(self,puzzle):
        self.__clean()
        return self.__sudoku_check(puzzle)
```



generate_sudoku 함수에서는, 생성한 sudoku 보드를 받아서, 구멍을 뚤어주는 역할을 수행한다.

(생각해보라, 이미 전부 풀린 sudoku puzzle을 받아봤자 무슨 의미가 있겠는가?)



위의 까지의 코드를 전부 종합해보면 아래와 같다.



```python
import random


class Sudoku():

    def __init__(self):
        self.SIZE = 9
        self.origin_board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.row = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.col = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.diag = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.terminate_flag = False

        self.__board_init()

    def __board_init(self):
        seq_diag = [0,4,8]
        for offset in range(0,9,3):
            seq = [i for i in range(1,self.SIZE+1)]
            random.shuffle(seq)
            for idx in range(0,9):
                i,j = idx//3,idx%3
                self.row[offset+i][seq[idx]] = 1
                self.col[offset+j][seq[idx]] = 1
                k = seq_diag[offset//3]
                self.diag[k][seq[idx]] = 1
                self.origin_board[offset+i][offset+j] = seq[idx]

    def __clean(self):
        self.origin_board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.board = [[0 for j in range(0,self.SIZE)] for i in range(0,self.SIZE)]
        self.row = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.col = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.diag = [[0 for j in range(0,self.SIZE+1)] for i in range(0,self.SIZE+1)]
        self.terminate_flag = False

    def __make_sudoku(self, k):

        if self.terminate_flag == True:
            return True

        board_size = self.SIZE*self.SIZE
        if k >= board_size :
            for i in range(0,self.SIZE):
                for j in range(0,self.SIZE):
                    self.board[i][j] = self.origin_board[i][j]
            self.terminate_flag = True
            return True

        i,j = k//self.SIZE,k%self.SIZE
        start_num = random.randint(1,self.SIZE)

        if self.origin_board[i][j] != 0:
            self.__make_sudoku(k+1)

        for num in range(1,self.SIZE+1):
            num = 1 + (num + start_num)%9
            d = (i//3)*3 + (j//3)
            if self.row[i][num] == 0 and self.col[j][num] == 0 and self.diag[d][num] == 0:
                self.row[i][num],self.col[j][num],self.diag[d][num] = 1,1,1
                self.origin_board[i][j] = num
                self.__make_sudoku(k+1)
                self.row[i][num],self.col[j][num],self.diag[d][num] = 0,0,0
                self.origin_board[i][j] = 0
    
    def __sudoku_check(self, puzzle):
        
        if type(puzzle) != type([]) or len(puzzle) == 0:
            return False

        row_size = len(puzzle)
        col_size = min([len(puzzle[i]) for i in range(0,row_size-1)])

        if col_size != self.SIZE or row_size != self.SIZE:
            return False

        for i in range(0,self.SIZE):
            for j in range(0,self.SIZE):
                if type(puzzle[i][j]) != type(1):
                    return False
        
        for i in range(0,self.SIZE):
            for j in range(0,self.SIZE):
                k = (i//3)*3 + (j//3)
                num = puzzle[i][j]
                self.row[i][num]+=1
                self.col[j][num]+=1
                self.diag[k][num]+=1
        
        for idx in range(0,self.SIZE):
            for num in range(1,self.SIZE+1):
                if self.row[idx][num]!=1 or self.col[idx][num]!=1 or self.diag[idx][num]!=1:
                    return False
        
        return True

    def __generate_puzzle(self):
        self.__clean()
        self.__make_sudoku(0)
        return self.board

    def generate_sudoku(self):
        puzzle = self.__generate_puzzle()
        count = random.randint(15,40)
        remove_coords = {}
        for i in range(0,count):
            coord = random.randint(0,self.SIZE-1),random.randint(0,self.SIZE-1)
            while coord not in remove_coords:
                coord = random.randint(0,self.SIZE-1),random.randint(0,self.SIZE-1)
                remove_coords[coord] = coord
        
        for coord in remove_coords:
            di,dj = coord[0],coord[1]
            puzzle[di][dj] = 0
        return puzzle
    
    def sudoku_check(self,puzzle):
        self.__clean()
        return self.__sudoku_check(puzzle)
```



이제 sudoku api 작성까지 끝났다. 해당 스도쿠 api를 한번 콘솔로 테스트 해보도록 하자.





## 실행



실행결과는 아래와 같다.



```shell
>>> from sudoku.api.sudokus import Sudoku
>>> sudoku = Sudoku()
>>> board = sudoku.generate_sudoku()
[[6, 1, 7, 4, 0, 5, 0, 8, 0], [5, 0, 3, 7, 6, 9, 1, 4, 2], [0, 2, 0, 8, 1, 3, 5, 6, 0], [7, 0, 2, 9, 0, 8, 4, 1, 3], [4, 0, 8, 0, 3, 6, 7, 0, 5], [1, 3, 5, 2, 4, 0, 8, 9, 6], [0, 5, 0, 3, 9, 1, 2, 7, 4], [2, 7, 9, 5, 8, 0, 0, 3, 1], [3, 4, 0, 6, 7, 2, 9, 5, 8]]
>>> sudoku.sudoku_check(board)
False
```



일단 실행을 해보니, 얼추 잘 작동하는것 같다. 

하지만, 우리가 만든 api를 연동하기 전에, 테스트를 먼저 해보아야 한다.



다음 포스팅에서는 python에서 테스트 코드를 어떻게 작성하는지에 대해 다루어 보도록 하겠다.