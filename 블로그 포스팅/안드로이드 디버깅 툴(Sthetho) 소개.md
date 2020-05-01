## 개요



이번 포스팅에서는 크롬 브라우저에서 안드로이드 디버깅을 할 수 있는 stetho에 대해서 알아보도록 하겠습니다.



## Sthetho?



Sthetho는 페이스북에서 개발한 크롬 브라우저에서 안드로이드를 디버깅 할 수 있게 해주는 안드로이드 라이브러리입니다.



보통 이러한 것들을 보고 디버그 브릿지(Debug Bridge) 라고 합니다.



## 설치 & 사용방법



먼저 안드로이드 gradle 파일에 아래 내용을 넣어 줍니다.



```groovy
implementation 'com.facebook.stetho:stetho:1.5.1'
```



만약 retrofit이나 okhttp 등의 http 통신 내용도 보고싶은 경우 아래내용을 추가해줍니다.



```groovy
implementation 'com.facebook.stetho:stetho-okhttp3:1.5.1'
```

 

그런 뒤, 안드로이드 어플리케이션 파일에 아래와 같이 코드를 적어줍니다.



```java
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    Stetho.initializeWithDefaults(this);
  }
}
```



그러고 나서 안드로이드 Manifest.xml 파일에 아래 내용을 추가해줍시다.



```xml
<manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        ...>
        <application
                android:name="MyApplication"
                ...>
         </application>
</manifest>            
```



okhttp 등의 http 통신 내용도 보고싶은 경우 아래와 같이 작성해 줍니다.



```java
new OkHttpClient.Builder()
    .addNetworkInterceptor(new StethoInterceptor())
    .build()
```



그러면 아래와 같이 크롬 브라우저를 통해 디버깅을 할 수 있습니다.



![Network Inspection](http://facebook.github.io/stetho/static/images/inspector-network.png)



![Database Inspection](http://facebook.github.io/stetho/static/images/inspector-sqlite.png)