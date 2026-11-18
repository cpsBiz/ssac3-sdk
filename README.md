# ssac3-sdk
싹쓰리 쇼핑 연동 Android SDK

## 개발환경
- 최소 SDK Version : Android 23 (minSdk 23)
- Compile SDK : Android 35 이상
- Build Tool : Android Studio

## 1. 의존성 추가

**settings.gradle.kts** 에 `mavenCentral()` 이 있어야 합니다.

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

**app/build.gradle.kts**

```kotlin
dependencies {
    implementation("io.github.cpsbiz-ssac3-1.1.3")
}
```

## 2. 권한 · 서비스 선언 (AndroidManifest.xml)

SDK 는 권한과 서비스를 **직접 선언하지 않습니다.** 사용하는 기능에 맞춰 앱의 `AndroidManifest.xml` 에 선언해 주세요.
선언이 빠져도 빌드는 통과하지만, 해당 기능이 실행 중에 조용히 동작하지 않습니다.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- 필수 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

</manifest>
```

| 권한 | 용도 | 빠지면 |
|---|---|---|
| `INTERNET` | 서버 통신 · 네트워크 상태 확인 | SDK 화면이 열리지 않습니다 |
| `ACCESS_NETWORK_STATE` | 서버 통신 · 네트워크 상태 확인 | 오프라인 안내 토스트가 뜨지 않습니다.  |


## 3. SDK 초기화 — `init(...)`

```kotlin
fun init(context: Context, appId: String)
```

싹쓰리 개발 서버와 연동 테스트를 하려면 `init` 의 세 번째 인자 `useDev` 에 `true` 를 넘긴다. SDK 가 여는 **웹 화면과 SDK 의 API 호출이 함께** 개발 서버로 바뀐다.

```kotlin
ShopPlusSDK.init(this, appId = "발급받은_appId", useDev = true)
```

**반드시 `Application.onCreate()` 에서 호출해 주세요.**

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        ShopPlusSDK.init(this, "매체사 ID")
    }
}
```

| 파라미터 | 설명 |
|---|---|
| `context` | `Application` 등 Context. SDK 는 `applicationContext` 만 보관합니다 |
| `appId` | 발급받은 매체사 ID. 이 값으로 연결할 서버가 정해집니다. |

- `Activity` 에서 호출하면 앱이 그 Activity 를 거치지 않고 시작될 때(공유하기로 실행, 시스템이 서비스를 다시 띄운 경우 등) SDK 가 초기화되지 않아 오류가 납니다.
- 다른 SDK 함수보다 **먼저** 호출해야 합니다.

## 4. 화면 열기 — `startShop(...)`

```kotlin
fun startShop(
    context: Context?,
    isOnLockScreen: Boolean = false,
    page: ShopPlusPath = ShopPlusPath.HOME
)
```

```kotlin
// 적립 홈
ShopPlusSDK.startShop(context)
```

| 파라미터 | 설명 |
|---|---|
| `context` | 호출하는 화면의 Context. `null` 이면 아무 일도 하지 않습니다 |

- SDK 화면은 한 번에 하나만 열립니다. 이미 열려 있으면 그 화면에서 새 페이지를 엽니다.

## 5. 회원 연동 (쿠키 방식) — `setEncryptedUserId(...)` · `clearEncryptedUserId()`

매체사 앱에서 이미 로그인한 회원을 싹쓰리 웹이 알아볼 수 있도록, **암호화된 회원번호**를 넘깁니다.

```kotlin
fun setEncryptedUserId(encryptedUserId: String)
fun clearEncryptedUserId()
```

```kotlin
// 로그인 완료 시 (init 이후)
ShopPlusSDK.setEncryptedUserId(encryptedUserId)

// 로그아웃 · 회원탈퇴 시
ShopPlusSDK.clearEncryptedUserId()
```

- `setEncryptedUserId` 는 **`init` 이후에** 호출해 주세요.
- 회원번호 암호화 방식은 싹쓰리와 협의한 방식을 따릅니다. 원문 회원번호를 넘기지 마세요.
- 빈 값을 넘기면 아무 일도 하지 않습니다. 연동을 끊을 때는 `clearEncryptedUserId()` 를 호출해 주세요.
- 로그아웃 · 회원탈퇴 시 `clearEncryptedUserId()` 를 꼭 호출해 주세요. 호출하지 않으면 이전 회원 정보가 남습니다.
- `startShop` 등으로 화면을 열기 **전에** 호출해야 그 화면부터 반영됩니다.

**광고 아이디 쿠키 (`setAdId`) — 선택**

광고 아이디(ADID)를 SDK 웹뷰 쿠키(`ssac3_adid`)로 심는다. 방식은 `setEncryptedUserId` 와 같다.

- **Kotlin**

```kotlin
// 광고 아이디를 얻은 뒤 — 웹뷰를 띄우기 전에 호출한다
ShopPlusSDK.setAdId(adId)

// 값을 더 넘기지 않을 때
ShopPlusSDK.clearAdId()
```

- 전제·규칙
- **`init` 이후**, **웹뷰를 띄우기 전에** 호출한다.
- **빈 값을 넘기면 아무 일도 하지 않는다.** 지울 때는 `clearAdId()` 를 쓴다.
