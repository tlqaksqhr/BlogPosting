## 개요



이번 포스팅에서는, javascript 에서 window.location.host가 빈 값이 나오는 경우에 대해서 알아보도록 하겠다.





## window.location 객체?



MDN 페이지의 설명에 따르면,  window.location 객체는 웹 브라우저에서 HTML Document의 위치를 알려주는 객체이다

보통 location에 있는 속성은 URI 스키마를 따르는데 대략적으로 아래 URL에 있는 그림과 같다.



https://developer.mozilla.org/en-US/docs/Web/API/Location



이때, 위 URL의 내용을 참고하면 window.location.host 는 현재 html이 있는 사이트의 호스트명이 된다.

과연 어떻게 하면 빈값이 나오게 되는것일까?



## 로컬 html에서 & host



사실 위 제목을 보면 바로 맞출수 있겠지만, 로컬 html파일을 실행시킨뒤, window.location.host를 찍어보면 빈값이 나오게 된다.



어떻게 생각해보면 직관적인것이, html파일을 실행시킨뒤, URL을 잘 보면 host와 hostname에 매칭되는 문자열이 전혀 보이지 않기 때문이다.



따라서, 빈 문자열을 리턴하게 되는것이다.







