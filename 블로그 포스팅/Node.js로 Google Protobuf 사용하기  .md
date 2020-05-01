### Node.js로 Google Protobuf 사용하기



## 개요



구글 protobuf는 XML이나 json과 같이 데이터 교환을 위해 만들어진 포맷으로 구글에서 제작하였다. json과는 다르게 바이너리 형태로 포맷이 이루어져있어서 json보다는 속도가 빠르다. 하지만, 별도의 컴파일러를 이용해서 스키마(데이터를 정의)파일을 컴파일을 해서 사용해야하기 때문에, 데이터 포맷을 변경하기가 json보다는 까다롭다는 단점이 있다. 



또한, 구글에서 제공하는 RPC(Remote Procedure, 주로 서비스간 통신을 주고받을때 많이 사용 함, 비슷한 서비스로 apache thrift가 있다)프레임워크인 gRPC와 같이 쓰이는 경우가 많다.



protobuf는 데이터의 타입을 정의하는 schema파일인 .proto파일과 이를 컴파일해서 사용하는 각 언어별 protobuf 라이브러리로 구성되어있다. 그럼 이제 데이터 타입을 정의하는법을 알아보도록 하자. 



## Protobuf 정의하기



Protobuf를 이용해서 데이터를 정의하려면 아래와 같이 정의하면 된다.



```protobuf
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}
```





## Protobufjs 설치



node.js에서 protobuf를 사용하기 위해 protobufjs라는 라이브러리를 설치해서 사용 해보도록 하겠다.



설치는 아래와 같이 하면 된다.



```shell
npm install protobufjs
```



브라우저에서 사용하려면 아래와 같이 사용하면 된다.



```html
<script src="//cdn.rawgit.com/dcodeIO/protobuf.js/6.X.X/dist/protobuf.min.js"></script>
```



## 예제



 먼저 아래와 같이 protobuf의 스키마(schema) 파일을 작성해준다.



```protobuf
// awesome.proto
package awesomepackage;
syntax = "proto3";

message AwesomeMessage {
    string awesome_field = 1; // becomes awesomeField
}
```



그 다음으로 아래 예제와 같이, protobufjs 를 통해서 스키마 파일을 불러서 protobuf 형식의 데이터를 읽고 쓰면 된다.



```javascript
protobuf.load("awesome.proto", function(err, root) {
    if (err)
        throw err;

    // protobuf 메시지 타입을 얻는다
    var AwesomeMessage = root.lookupType("awesomepackage.AwesomeMessage");

    // protobuf에 넣을 데이터 정의
    var payload = { awesomeField: "AwesomeString" };

    // 선언한 protobuf 에 넣을 데이터가, 우리가 위에서 선언한 protobuf 데이터 schema에 맞는지 확인
    var errMsg = AwesomeMessage.verify(payload);
    if (errMsg)
        throw Error(errMsg);

    // 위에서 선언한 데이터를 protobuf 형식으로 만들어 준다.
    var message = AwesomeMessage.create(payload);

    //  protobuf 형식의 데이터를 Uint8Array(브라우저) 혹은 Buffer(node.js)형식으로 serialize 한다.
    var buffer = AwesomeMessage.encode(message).finish();
    // ... do something with buffer

    // protobuf 형식의 데이터를 Uint8Array(브라우저) 혹은 Buffer(node.js)형식으로 deserialize 한다.
    var message = AwesomeMessage.decode(buffer);
  
  	// 여기 부터는 protobuf 데이터로 적절히 작업을 해줍시다 ㅎㅎ..
  
  	// json 형식으로 변환
    var object = AwesomeMessage.toObject(message, {
        longs: String,
        enums: String,
        bytes: String,
        // ConversionOptions 참조
    });
});
```

