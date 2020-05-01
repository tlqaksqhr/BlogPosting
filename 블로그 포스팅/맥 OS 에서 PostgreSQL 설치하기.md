# 맥 OS 에서 PostgreSQL 설치하기



## 1. Postgres SQL 설치



아래 명령어를 이용해서 PostgreSQL을 설치한다.



```shell
brew install postgresql
```



설치가 완료 되었으면 아래 명령어를 이용해 postgresql 서비스를 시작한다.





```shell
pg_ctl -D /usr/local/var/postgres start && brew services start postgresql
```



서비스가 정상적으로 실행됬는지 확인하기 위해서, 아래 명령어를 이용해서 확인 해준다.



```shell
postgres -V
```



##2. Postgres SQL 설정하기



### 개요



PostgresSQL을 설치하고 나면, 기본적으로 postgres 유저가 자동으로 생성이 된다. 또한, PostgresSQL 에서는 postgresSQL 연결을 위해서 psql이라는 쉘을 제공 한다. 이를 이용해서 postgresSQL 설정을 해보도록 하겠다.



### 사용자 권한 생성 및 사용자 생성



우선 아래 명령어로 postgresSQL에 접속을 한다.



```shell
psql postgres
```



위 명령어를 입력해서 PostgresSQL에 접속을 한뒤, 아래 명령어를 입력한다.



```shell
postgres=# \du
```



해당 명령어를 입력하면, 아래 그림처럼 미리 정의된 권한을 확인 할 수 있다. 사실, 아래 그림에서 주어진 권한을 써도 되지않냐는 의문이 들 수도 있지만, 데이터베이스를 이용해서 실제 서비스를 제작할때, 아래 그림과 같이 너무 많은 권한이 주어진 데이터베이스 계정을 쓰는 경우 많은 문제가 발생 할 수 있기 때문에, 최소한도의 필요한 권한만을 가져야 한다.



![postgresql server](https://s3.amazonaws.com/codementor_content/2016-Oct/postgres1/psql-du-output.png)



우선, 권한을 설정해주기 전에, 기본으로 생성된 postgres 유저의 비밀번호를 설정해줘야 한다. 



아래 명령어를 이용해서 패스워드를 설정해준다.

```shell
postgres=# \password postgres
```



다음으로, 아래 명령어를 이용해 새로운 권한을 생성하고 권한이 제대로 생성되었는지를 확인 한다.

```shell
postgres=# CREATE ROLE testdb WITH LOGIN PASSWORD 'testdb'
postgres=# \du
```



위 명령어를 실행하면, 권한을 생성했음에도 불구하고 실제 권한은 주어지지않았다는 것을 알 수 있다. 

사실 PostgreSQL에서는 보안적인 이유때문에 새로 생성된 권한에는 기본적으로 읽기권한밖에 주어지지 않는다

아래 명령어를 이용해서 해당 권한에 추가적인 권한을 부여한다.



```shell
postgres=# ALTER ROLE testdb CREATEDB; 
postgres=# \du 
postgres=# \q # quits
```



위 명령어를 입력한 경우, 정상적으로 데이터베이스 생성 권한이 추가된것을 알 수 있다.



아래 명령어를 이용해서, 새 유저를 생성한다.



```shell
createuser testdb --createdb
```



PostgreSQL 에서는 "createuser" 이외에도 다른 여러가지 명령어들을 제공한다. 자세한 내용은 아래 링크를 참조하면 된다.

https://www.postgresql.org/docs/9.5/static/reference-client.html



우선 아래 명령어를 이용해서 새로 생성한 유저로 postgreSQL에 접근한다.



```shell
psql postgres -U testdb
```



접속하면 아래 명령어를 이용해서 새로운 데이터베이스를 생성한다.



```shell
postgres=> CREATE DATABASE testdb;
```



생성이 완료되면, 생성된 데이터 베이스에 접근가능한 유저를 추가하고 권한을 설정 해준다.



```shell
postgres=> GRANT ALL PRIVILEGES ON DATABASE testdb TO testdb; 
```



권한 설정이 완료되면 제대로 권한이 설정되었는지 아래 명령어를 이용해 확인 한다.

```shell
postgres=> \list
postgres=> \connect super_awesome_application 
postgres=> \dt 
postgres=> \q
```



이제 해당 데이터베이스와 개발을 이용해서 postgresSQL에 접근이 가능하다.



## 3. Mac OS에서 유용한 Postgres SQL 관리 도구



맥 OS X 에서는 아래와 같은 Postgres SQL 관리 도구가 존재 한다.

- Postico (https://eggerapps.at/postico/)
- pgAdmin (https://www.pgadmin.org/)
- Navicat (https://www.navicat.com/products/navicat-for-postgresql)



상황에 따라서 적절하게 이용해 주면 된다.



출처 : https://www.codementor.io/@engineerapart/getting-started-with-postgresql-on-mac-osx-are8jcopb