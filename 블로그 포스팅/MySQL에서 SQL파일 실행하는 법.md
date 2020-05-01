## 개요



예전에 작업하는 도중에, 솔루션 인프라 구성을 하기 위해 SQL파일을 실행해야 하는 일이 생겼다.

이때, MySQL에서 SQL을 실행하는법을 찾는데 시간이 좀 걸렸었는데, 까먹을 수도 있어서 블로그에 정리를 하려 한다.





## 실행하는 법



### 방법 1



mysql 에 로그인을 한 뒤 "source <SQL파일 경로>" 를 쳐주면 된다.



ex)

```shell
mysql> source \home\user\Desktop\test.sql;
```

 

### 방법 2



mysql 로그인시 아래 명령어를 이용해서 SQL 파일에 있는 SQL들을 바로 실행 할 수 있다.



ex)

```shell
mysql -h hostname -u user database < path/to/test.sql
```

