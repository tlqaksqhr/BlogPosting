## 개요



평소에, rabbitMQ에서 계정을 생성하거나 패스워드를 변경하고 나서, 제대로 생성되었는지 확인하는법을 몰라서 rabbitMQ 라이브러리를 이용해서 일일히 프로그래밍을 해서 확인해야 하는 불편한 점이 있었다.



그러던 중, rabbitMQ에서 제공하는 REST API로 바로 확인하는 법이 있어서 공유를 하려고 한다.



## 확인방법



아래와 같은 명령어를 이용해서 확인하면 된다.

```shell
curl -i-u <계정명>:<비밀번호> http://localhost:15672/api/whoami
```



## 기타



그 외에도 rabbitMQ 에서는 다양한 REST API를 제공한다.

더 자세한 내용은 https://pulse.mozilla.org/api/를 보면 된다.



## 출처

1. https://pulse.mozilla.org/api/
2. https://stackoverflow.com/questions/17148683/verify-rabbitmq-credentials-are-valid