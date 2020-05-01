## Gradle Task 변수 & 안드로이드 & BuildVariants





## 개요

​       

회사에서 Gradle 과 관련된 솔루션 버그를 잡다가 막힌 내용이 있었는데, 까먹기전에 그 내용을 간략하게 정리해서 올리려고 한다.

​            

​      

​        

## 내용

​    				

​     				

   					 

Gradle은, groovy 또는 코틀린을 이용해서 프로그래밍 하듯이 빌드 스크립트를 코딩해서 제작이 가능하다.

따라서, 빌드할 내용이 복잡해지면 마치 코드 리팩토링을 하는것처럼 빌드 스크립트도 리팩토링을 해야하는 일이 생긴다.

​	            	      						                  

​                   		                  		

​                             	

​                     		

또한, 일반적인 build.gradle 파일 1개만으로 감당이 안되는 복잡한 작업을

(예를 들어, 코드 난독화 솔루션이나, 컴파일된 클래스를 변경해서 성능 모니터링을 하는 솔루션) 하는 경우를 대비해서, 

Gradle은 플러그인을 만들 수 있는 기능을 제공한다.

​                         

​                 

​                    

보통 저러한 Gradle 플러그인 들을 사용 할때 현재 프로젝트에 설정되어 있는 빌드 관련 정보(라이브러리 위치, 컴파일된 클래스 파일, 리소스데이터들의 위치, 난독화 여부, SDK 위치 등등)들을 가져와야 하는 경우가 많다.

​                        

​                   

​               

안드로이드와 같은 경우에는, 이러한 빌드 관련정보를 buildVariants 라는 변수로 관리하게 된다.

(보통 buildVariants는 applicationVariants, libraryVariants 등이 존재한다).

​                   

​                   

보통 아래와 같은 방식으로 접근을 해서 사용한다.

​                                    

```groovy
android.applicationVariants.all { variant ->
  // 여기서 하고싶은거 하기
  // variants.javaCompile
}
```

​                                

또한 아래와 같은 코드로 실제 안드로이드 빌드 과정에 간섭 / 빌드 도중에 정보를 가져오는 코드도 작성 할 수 있다.                    

​                                    

```groovy
android.applicationVariants.all { variant ->
  variants.javaCompile.doLast { task ->
    // 실제 빌드 과정에 간섭하는 코드를 여기에 작성
    // codeObfuscation(task);
  }
}
```

​                   

이제, 대략적인 개요 설명은 끝났으니, 내가 삽질했던 버그의 내용을 정리해보도록 하겠다.

​                

​               

## 버그

​                                            

다시 한번 아래의 코드를 보자.

```groovy
applicationVariants.all { variant ->
  variants.javaCompile.doLast { task ->
    // 실제 빌드 과정에 간섭하는 코드를 여기에 작성
    // codeObfuscation(task);
    
  }
}
```

​                              

위의 코드에 있는 task 라는 변수의 타입을 출력하면 보통 Task 클래스나, 그 파생 클래스의 타입이 나오게 된다.

​                                    

하지만, 아래와 같이 별다른 변수이름을 적어주지 않고 타입을 출력 해주면 "applicationVariants.all" 보다 상위 scope에 있는 task 변수의 타입이 출력되서 엉뚱한 결과가 나오게 된다.

​                          

```groovy
applicationVariants.all { variant ->
  variants.javaCompile.doLast { 
    // 실제 빌드 과정에 간섭하는 코드를 여기에 작성
    // codeObfuscation(task);
    
  }
}
```

​                          

따라서, 위와 같은 코드를 작성할시에는 무조건 인자명을 제대로 적어주도록 하자. 그렇지 않은 경우에는 예상치 못한 결과가 나올 수도 있다.

