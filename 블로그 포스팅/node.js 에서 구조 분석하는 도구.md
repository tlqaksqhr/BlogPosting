## 개요



일정 규모 이상의 프로젝트를 진행할때에 있어서 모듈간 의존성이나, 코드의 품질을 관리하는것은 매우 중요하다.



그래서 많은 사람들(특히 소프트웨어 공학을 전공한 석,박사들)이 어떤 코드가 나쁜 품질의 코드인지 좋은 품질의 코드인지를 구분하는 많은 기준들을 만들어 냈다.



보통 아래와 같은 기준들이 존재 한다.



- Instability
  - 해당 패키지 또는 모듈이 얼마나 바꾸기 힘든지를 나타내는 지표이다(보통 다른곳에서 많이 쓰고있는 모듈들이 바꾸기 힘드므로.)
  - N(Efferent Coupling) / (N(Afferent Coupling) + N(Efferent Coupling)) 로 계산한다.
  - Efferent Coupling : 다른곳에서 해당 패키지를 사용하고 있다는 의미
  - Afferent Coupling : 해당 패키지가 다른곳의 모듈을 참조하고 있다는 의미
- Abstractness
  - 패키지(또는 모듈) 안에 얼마나 많은 인터페이스나 추상클래스의 비율이 얼마나 되는지를 알려주는 지표이다.
    - 한마디로 해당 패키지가 추상화가 얼마나 되었냐를 알려주는 지표이다.
  - N(AbstractClass) / (N(AbstractClass) + N(ConcreteClass))
- Main Sequence
  - 보통 Core(프레임워크의 핵심 로직)단은 인터페이스가 많아야 하고, 실제 구현체는 Concrete Class가 많아야 한다.
  - 따라서 이러한 기준을 수치화 한것이 "Main Sequence" 이다.
  - Main Sequence = 추상 클래스 개수 + 인터페이스 개수 로 계산 가능하다.
  - 보통 아래와 같이 많이 사용한다.
  - |추상 클래스 개수 + 인터페이스 개수 - 1| / sqrt(2)
- Complexity Metric
  - 코드내에 얼마나 많은 분기문이 존재하는를 알려주는 지표(보통 코드에 분기문이 많을 수록 예외가 많아서 복잡한 코드이다.)
  - "코드 구문 개수" - "분기 개수" + 2 로 계산 가능하다. 



더 자세한 내용은 아래 출처를 참고하면 좋다.



그래서 자바와 같은 경우 **STAN4J** 와 같은 유명한 구조분석 도구들이 존재한다. 하지만, 자바스크립트는 워낙에 파편화가 많이 되어서 이러한 도구가 없는줄 알았다.



하지만 아는 분에게 자바스크립트에도 **arkit** 이라는 구조분석 툴이 있다고 해서 이번 포스팅을 통해 소개 하도록 하겠다.



## 기능



사실 해당 도구가 STAN4J 만큼 많은 기능들을 제공해 주지는 않는다. 그러나, 패키지간의 의존관계를 시각화해서 한눈에 보여주기 때문에, 일단 눈에 보이는 큰 문제들은 제거 가능하다는 장점이 존재한다. (또한 보고서나 제안서를 써서 상대방을 설득 또는 돈을 뜯어 낼 때에도 유리하다 ^^)



## 설치



아래와 같이 설치를 할 수 있다.

```shell
npx arkit
또는

npm install arkit --save-dev
yarn add arkit --dev
```



사용방법은 아래와 같다.



```
semtax@semtaxui-MacBookPro:~$ npx arkit --help
arkit [directory]

Options:
  -d, --directory  Working directory                              [default: "."]
  -c, --config     Config file path (json or js)         [default: "arkit.json"]
  -o, --output     Output path or type (svg, png or puml) [default: "arkit.svg"]
  -f, --first      File patterns to be first in the graph               [string]
  -e, --exclude    File patterns to exclude from the graph
        [default: "test,tests,dist,coverage,**/*.test.*,**/*.spec.*,**/*.min.*"]
  -h, --help       Show help                                           [boolean]
  -v, --version    Show version number                                 [boolean]
```



또한 아래와 같이 json 파일을 통해 자세한 옵션을 설정 할 수 도 있다.



```javascript
{
  "$schema": "https://arkit.pro/schema.json",
  "excludePatterns": ["test/**", "tests/**", "**/*.test.*", "**/*.spec.*"],
  "components": [
    {
      "type": "Dependency",
      "patterns": ["node_modules/*"]
    },
    {
      "type": "Component",
      "patterns": ["**/*.ts", "**/*.tsx"]
    }
  ],
  "output": [
    {
      "path": "arkit.svg",
      "groups": [
        {
          "first": true,
          "components": ["Component"]
        },
        {
          "type": "Dependencies",
          "components": ["Dependency"]
        }
      ]
    }
  ]
}
```







아래는 arkit을 이용해서 express.js 프레임워크의 패키지간 의존성 구조를 분석한 결과를 나타낸 그림이다.



![Express architecture graph](https://github.com/dyatko/arkit/raw/master/test/express/express.svg?sanitize=true)



상당히 간단한것을 확인 할 수 있다.



아래는 reactDOM의 패키지간 의존성과 같은 구조를 분석한 결과이다.



[![ReactDOM architecture graph](https://github.com/dyatko/arkit/raw/master/test/react-dom/arkit.svg?sanitize=true)](https://github.com/dyatko/arkit/blob/master/test/react-dom/arkit.svg?sanitize=true)





상당히 복잡한 것을 알 수 있다.



물론 이러한 지표들이 모든 문제를 해결해주는 만병통치약(Silver Bullet)은 아니지만, 만약 팀 프로젝트나 회사 프로젝트를 진행한다면 이를 이용해서 구조를 분석해서 조금 더 효율적인 프로그램을 짤 수 있지 않을까 싶다.



출처 : 

1. https://architecture101.blog/2013/10/08/architecture_visualization_ii/
2. https://architecture101.blog/2012/05/15/uncle_bob_graph/



