## 소개



이번 시간에는 pwnable.kr 에 있는 fd 라는 문제를 풀어보도록 하겠습니다.





## 풀이



먼저, 아래 주소에 ssh를 이용해서 접속해줍니다.



```shell
ssh fd@pwnable.kr -p2222
```





접속해서 파일들을 살펴보면 아래 파일들이 있는것을 볼 수 있습니다.



```shell
fd fd.c flag
```



먼저 문제 코드를 읽어 보도록 하겠습니다. 

문제 코드는 아래와 같습니다.



```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

char buf[32];

int main(int argc, char* argv[], char* envp[]){
  
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;
  
}
```



우리의 목표는, 저 system 함수를 실행시키는 겁니다.

과연 어떻게 하면 될까요?



자, 시스템 프로그래밍 시간때 배운 내용을 떠올려 봅시다.

리눅스에서는 기본적으로 아래 3개의 파일 디스크립터를 열어 놓습니다.



| 파일 디스크립터 번호 | 분류      |
| -------------------- | --------- |
| 0                    | 표준 입력 |
| 1                    | 표준 출력 |
| 2                    | 표준 에러 |



그리고, read 함수는 지정된 파일 디스크립터에서 내용을 읽어 들일수 있습니다.



이 사실들을 종합해보면 저 fd를 0으로 맞춰 주고, "strcmp"을 이용해서 비교하는 값을 넣어주면 문제가 풀리게 되는겁니다.

아래와 같이 실행해줍시다



```shell
./fd 4660
LETMEWIN
```



이렇게 하면 정답을 얻을 수 있습니다.