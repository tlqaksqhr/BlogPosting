### 1. ASM을 어디에다 쓰는가?

1. 난독화 도구 제작등에도 사용 됨.
2. JIT만들때도 활용이 가능 함.
3. 그 외 기존 class 파일의 바이트 코드 패치를 할때 사용 됨.
   1. AspectJ 같은데서도 쓰인다고 함

### 2. ASM 라이브러리의 전체적인 구조

#### 2-1. 전체 구성

- Tree Based API와 Event Based API를 제공함
- Tree Based API는 Top-Down으로 자바 바이트 코드를 파싱하여 생성된 AST를 순회하는 방식으로 동작
- Event Based API는 순차적으로 바이트코드를 읽어들어서 특정 구문이 발견되었을때(Ex: 클래스 정의, 변수선언 등) 해당 구문이 발생하면 이벤트 핸들러가 실행되는 방식임

#### 2-2. 주요 모듈

- org.objectweb.asm : 이벤트 based API를 제공
- org.objectweb.asm.util : 기타 함수들을 제공
- org.objectweb.asm.commons : 미리 정의된 Listener(Filter)들을 제공
- org.objectweb.asm.tree : 트리 based API를 제공
- rg.objectweb.asm.tree.analysis : 트리기반의 미리 정의된 Filter(AST Handler) 들을 제공

#### 2-3. 주요 프로그래밍 방식

- Asm의 객체들은 크게 아래 3가지 형태로 나누어짐
  - Event Producer : 클래스 파일을 읽어들이고 파싱하는 역할
  - Event Consumer : 클래스 파일을 생성하는 역할
  - Event Filter : 클래스 파일에서 특정 요소들을 필터링하는 역할
- 보통 아래와 같은 단계를 통해 라이브러리를 사용하고 진행이 됨
  1. 목적에 맞게 Event Producer와 Event Consumer, Event Filter들을 조합 함.
  2. 이를 통해  클래스파일이 저 조합된 필터들을 통해서 변형 또는 생성되어 출력 됨.