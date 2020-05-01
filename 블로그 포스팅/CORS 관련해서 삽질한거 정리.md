## 개요



이번 포스팅에서는, script 태그를 원격에서 불러올때, 발생 할 수 있는 CORS 문제에 대해 알아보도록 하겠습니다.



회사에서 성능 모니터링 수집도구를 만들다가 발생한 이슈가 있어서 하루종일 삽질을 하다가, 

CORS에 대해 공부하게 되어 그 내용을 정리하였습니다.





## CORS?



먼저 구글에서, CORS의 정의를 찾아보면 아래와 같습니다.



> **Cross-origin resource sharing** (**CORS**) is a mechanism that allows restricted [resources](https://en.wikipedia.org/wiki/Web_resource) on a [web page](https://en.wikipedia.org/wiki/Web_page) to be requested from another [domain](https://en.wikipedia.org/wiki/Domain_name) outside the domain from which the first resource was served.[[1\]](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#cite_note-mozhacks_cors-1)
>
> A web page may freely embed cross-origin images, [stylesheets](https://en.wikipedia.org/wiki/Style_sheet_(web_development)), scripts, [iframes](https://en.wikipedia.org/wiki/HTML_element), and videos.[[2\]](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#cite_note-2) Certain "cross-domain" requests, notably [Ajax](https://en.wikipedia.org/wiki/Ajax_(programming)) requests, are forbidden by default by the [same-origin security policy](https://en.wikipedia.org/wiki/Same-origin_policy). CORS defines a way in which a browser and server can interact to determine whether it is safe to allow the cross-origin request.[[3\]](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#cite_note-nczonline1-3) It allows for more freedom and functionality than purely same-origin requests, but is more secure than simply allowing all cross-origin requests.



즉, 요약하면 브라우저에서 웹에서 자원(웹 페이지, 이미지, 동영상, 자바스크립트 파일, css 등등)을 요청할때 동일 주소 에서만 HTTP로 요청을 할 수 있다는 제약입니다. (보통 Same Origin security) 라고 합니다.



이때, "동일 주소" 라는 개념이 바로 Origin 이라는 개념입니다.



예를 들어, http://whysoseriousnotbegin.com/movie 라는 영화를 등록하는 주소가 있다고 가정합시다. 



이때, 같은  http://whysoseriousnotbegin.com/ 사이트 내에서는 브라우저에서 위의 주소에 HTTP 요청을 보낼 수 있지만,

http://localhost:8080 이나 다른 사이트에서는 HTTP 요청을 보내지 못하게 됩니다.



보통 Origin을 Unique 하게 구분하는 기준은 아래와 같습니다.



1. **HTTP Method**
2. **URL**
3. **포트 번호** 



위의 3개중에 1개라도 다른 경우에는 서로 다른 origin으로 인식을 하게 됩니다.



사실 저 Cross-Origin이 나오게 된 이유는, 다른 사용자가 악의적인 스크립트를 삽입해서 쿠키나 브라우저 내의 정보들을 빼돌린다거나 하는 공격을 수행할 수 있기 때문에 나오게 되었습니다.



하지만, 매쉬업 서비스가 대세가 된 지금, 저 Origin을 제어 할 수 있는 수단이 필요하게 되었습니다.

그게 바로 Cross-Origin 이라는 겁니다.



보통 웹 서버에서는, 저러한 Cross-Origin목록들을 **Access-Control-Allow-Origin** 이라는 HTTP 응답 헤더로 관리하게 됩니다.



저  **Access-Control-Allow-Origin** 헤더에, 보통 허용되는 URL 목록들을 적어주거나, "*" 를 적어서 모두 허용하겠다고 설정 할 수 있습니다.



보통 저  **Access-Control-Allow-Origin** 헤더는 서버에서 설정을 해주게 됩니다.



이제 그러면, 좀 특수한 경우에 대해 알아보도록 합시다.





## 문제(?)

​               						

​                           

​           					                   

그럼 script 태그를 이용해서 원격에서 자바스크립트 파일을 불러오고, 불러온 파일에서 ajax 요청을 보내면 origin이 어떻게 찍히게 될까요? 

​      					                        

바로 원래 자바스크립트 파일이 있는 URL의 도메인이 origin으로 인식 됩니다. 실제로 테스트를 해보면 아래 이미지와 같은 결과가 나옵니다.





​              				           

따라서, 만약 자바스크립트 라이브러리 중에 XMLHttpRequest.prototype을 오버라이딩 하는 라이브러리를 작성하거나 가져다 쓰는 경우에는 해당 문제에 대해 염두에 두어야 합니다.

​                  

​                 

​                                       

## 해결방법(?)

​              

해결 방법은 간단합니다. 아래와 같이 script 태그에, crossorigin 키워드를 붙여주거나,

로컬에서 자바스크립트 파일을 script 태그를 이용해서 import 해주면 됩니다.