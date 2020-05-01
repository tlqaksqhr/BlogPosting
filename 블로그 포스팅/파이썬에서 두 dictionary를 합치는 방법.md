## 개요



파이썬으로 함수형 프로그래밍을 하다보니, 두 dict 자료형을 합쳐서 합쳐진 결과를 반환하는 함수가 없나 찾아보았다.



다행히도 파이썬 3.5 이상부터 이러한 연산을 지원해주는 내장 연산자가 있어서 소개를 해보려고 한다.



## 사용법



해당 연산자의 사용법은 아래 코드 와 같다.



```python3
d1 = {'a' : 1, 'b' : 2}
d2 = {'c' : 3, 'd' : 4}

result = {**d1, **d2}
print(result) # {'a' : 1, 'b' : 2, 'c' : 3, 'd' : 4}
```



## 활용



파이썬으로 함수형 프로그래밍 수행시에, dictionary의 리스트를 인자로 받아서 해당 리스트에 특정 값을 추가 할 때 사용 할 수 있다.



## 출처

1. https://stackoverflow.com/questions/38987/how-do-i-merge-two-dictionaries-in-a-single-expression