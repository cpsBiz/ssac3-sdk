# ssac3-sdk
싹쓰리 쇼핑 연동 Android SDK

## 개발환경
- 최소 SDK Version : Android 23
- Compile SDK : Android 35 이상
- Build Tool : Android Studio 
- Application Context 를 요구

**  project 의 settings.gradle.kts 에 아래와 같이 주소가 추가되야 합니다.
  ```kotlin      
    repositories { 
       ...
      mavenCentral()
       ...
    }
    
```


**  App 의 build.gradle 에 아래와 같이 주소가 추가되야 합니다.
 ```XML
dependencies {
  ...
  implementation("io.github.cpsbiz:ssac3:1.0.4")
  ...
}
```


##  SSAC3 SDK 선언
** App 의 Application 에 아래와 같이 코드를 넣어주세요
  ```kotlin
 override fun onCreate(owner: LifecycleOwner) {
            super.onCreate(owner)
   ...
   ShopPlusSDK.init(getContext(),"서비스 이름")
   ...
}
    
```

##  SSAC3 SDK 호출
** SDK 실행 시점에 아래와 같이 코드를 넣어주세요

 ```kotlin

  fun openSDK(){
   ...
     ShopPlusSDK.startShop(context)
   ...
}
    
```
