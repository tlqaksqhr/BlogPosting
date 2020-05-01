## 개요



이번 포스팅에서는 파이썬에서 쉽고 편하게 HTTP(S) 요청을 보내고 받을 수 있는 **Requests** 라이브러리에 대해 알아보도록 하겠다.



## Why Requests?



사실 파이썬에는, http(s) 요청을 보내고 받을 수 있는 urllib이나 httplib과 같은 내장 라이브러리들이 존재한다.

하지만, 파이썬 내장 라이브러리들은 사용하는게 불편하다는 단점이 존재한다.



그래서 파이썬 HTTP 라이브러리로 여러가지가 나왔는데, 그 중에서 가장 많이 쓰고있는것이 바로 이 **Requests** 라이브러리이다.



## 설치



아래 명령어를 이용해서 설치를 수행하면 된다.



```shell
$ pip3 install requests
```

 

일단 사용은 아래와 같이 하면 된다.



```python
import requests

r = requests.post('https://httpbin.org/post', data = {'key':'value'})
```



아래와 같이 다양한 http Method들을 편하게 사용 가능하다.



```python
import requests

r = requests.put('https://httpbin.org/put', data = {'key':'value'})
r = requests.delete('https://httpbin.org/delete')
r = requests.head('https://httpbin.org/get')
r = requests.options('https://httpbin.org/get')
```



또한 json 객체도 json() 함수를 써서 아래와 같이 바로 파싱해서 사용 가능하다.



```python
import requests

r = requests.get('https://api.github.com/events')
print(r.json())
```



HTTP Request 헤더는 아래와 같이 추가가 가능하다.



```python
import requests

url = 'https://api.github.com/some/endpoint'
headers = {'user-agent': 'my-app/0.0.1'}

r = requests.get(url, headers=headers)
```





조금 더 복잡한 예제를 수행해보자, 아래 예제는 메X박스 에서 현재 상영중인 영화를 가져오는 예제이다.



```python
import requests

url = "http://www.megabox.co.kr/DataProvider"
param_data = {
	"siteCode" : 36,
	"startNo" :	0,
	"count"	: 100,
	"playDate" : "2020-01-27",
	"sortBy" : "rank",
	"_command" : "Booking.getBookingMovieListByDate"
}

req = requests.post(url,data = param_data)

print(req.json())
```



실행해보면 json으로 된 현재 상영중인 영화목록을 얻어 올 수 있다.



이제 **Requests** 를 사용해서 사이트 데이터를 조금 더 편하게 긁어올 수 있을것이다.



물론, HTTP 요청만으로 사이트의 모든 내용을 긁어오는데에는 한계가 있다. 

그럴 때에는 **selenium** 이라는 것을 쓰면 되는데, 이는 다음 포스팅에서 다루도록 하겠다.