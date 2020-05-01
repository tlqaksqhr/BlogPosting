# PostgreSQL에서 AUTO INCREMENT 사용하기



## 개요



이번 문서에서는 mysql에 있는 AUTO INCREMENT와 유사한 기능을 PostgreSQL에서는 어떻게 사용하는지 알아 보도록 하겠다.



## PostgreSQL Serial Type



PostgreSQL에서는 auto increment가 지원이 되는 Sequence 라는 타입을 지원하고 있다.



테이블 생성시 아래와 같이 SERIAL 키워드를 이용해서 컬럼의 타입을 정의함으로써,  Sequence 타입을 사용 할 수 있다.



```plsql
CREATE TABLE table_name(
    id SERIAL
);
```



SERIAL 타입은 postgreSQL에서 지원하는 타입이므로, 내부적으로 실행될때는 아래와 같은 SQL로 바뀌게 된다.



```plsql
CREATE SEQUENCE table_name_id_seq;
 
CREATE TABLE table_name (
    id integer NOT NULL DEFAULT nextval('table_name_id_seq')
);
 
ALTER SEQUENCE table_name_id_seq
OWNED BY table_name.id;
```



보통 auto increment와 같은 기능은 primary key에 주로 사용되므로 보통 아래와 같은 방식으로 많이 사용이 된다.



```plsql
CREATE TABLE fruits(
   id SERIAL PRIMARY KEY,
   name VARCHAR NOT NULL
);
```



데이터를 삽입할때는 2가지 방식으로 삽입이 가능하다.



```plsql
INSERT INTO fruits(name) VALUES('Orange');
```



```plsql
INSERT INTO fruits(id,name) VALUES(DEFAULT,'Apple');
```





PostgreSQL에서 SERIAL 타입은 아래 표와 같이 다양한 크기의 키 사이즈를 지원한다.

| 타입명      | 크기   | 범위                          |
| ----------- | ------ | ----------------------------- |
| SMALLSERIAL | 2 byte | 1 ~ 32,767                    |
| SERIAL      | 4 byte | 1 ~ 2,147,483,647             |
| BIGSERIAL   | 8 byte | 1 ~ 9,223,372,036,854,775,807 |



