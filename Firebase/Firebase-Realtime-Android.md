# Firebase Realtime Database : Android

## Android 에서 설치 및 설정

### Firebase 에 앱 연결

Android 프로젝트에 Firebase 추가는 다음 링크를 따라 수행한다.

[Android 프로젝트에 Fiebase 추가하기](https://firebase.google.com/docs/android/setup?hl=ko)

<br/>

### 데이터베이스 만들기

1. Firebase Console의 실시간 데이터베이스 섹션으로 이동한다. 기존 Firebase 프로젝트를 선택하라는 메시지가 표시된다. 데이터베이스 만들기 워크플로를 따른다.

2. Firebase 보안 규칙의 시작 모드를 선택합니다.

   - 테스트 모드
     모바일과 웹 클라이언트 라이브러리를 시작할 때 유용하지만 모든 사용자가 데이터를 읽고 덮어쓸 수 있습니다. 테스트 완료 후 Firebase 실시간 데이터베이스 규칙 이해 섹션을 검토해야 한다.
   - 잠금 모드
     모바일과 웹 클라이언트의 모든 읽기와 쓰기를 거부합니다. 인증된 애플리케이션 서버에서는 사용자의 데이터베이스에 계속 액세스할 수 있다.

```
참고: 테스트 모드에서 데이터베이스를 만들고 체험 기간 내에
기본값 공개 읽기와 공개 쓰기 규칙을 변경하지 않을 경우

이메일로 알림을 받게 되며 데이터베이스 규칙이 모든 요청을 거부한다.
Firebase Console 설정 과정에서 만료일을 확인해야 한다.
웹, iOS 또는 Android SDK를 시작하려면 테스트 모드를 선택하라.
```

3. 데이터베이스의 리전을 선택한다. 선택한 리전에 따라 데이터베이스 네임스페이스는 `<databaseName>.firebaseio.com` 또는 `<databaseName>.<region>.firebasedatabase.app` 형식이 된다.

4. 완료를 클릭한다.

<br/>

### 앱에 실시간 데이터베이스 SDK 추가

`Firebase Android BoM`을 사용하여 모듈(앱 수준) `Gradle` 파일(일반적으로 `app/build.gradle`)에서 실시간 데이터베이스 Android 라이브러리의 종속 항목을 선언한다.

```s
dependencies {
    // Import the BoM for the Firebase platform
    implementation platform('com.google.firebase:firebase-bom:28.3.0')

    // Declare the dependency for the Realtime Database library
    // When using the BoM, you don't specify versions in Firebase library dependencies
    implementation 'com.google.firebase:firebase-database-ktx'
}
```

<br/>

### 실시간 데이터베이스 규칙 구성

실시간 데이터베이스가 제공하는 선언적 규칙 언어로 데이터의 구조, 색인 생성 방법, 데이터를 읽고 쓸 수 있는 조건을 정의할 수 있다.

기본적으로 인증된 사용자만 데이터를 읽고 쓸 수 있도록 데이터베이스에 대한 읽기와 쓰기 액세스 권한이 제한된다. 공개 액세스 규칙을 구성하면 인증을 설정하지 않고 시작할 수 있다. 다만 이렇게 하면 앱을 사용하지 않는 사람을 포함하여 모두에게 데이터베이스가 공개되므로 인증을 설정할 때 데이터베이스를 다시 제한해야 한다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/realtime-database-rules.PNG" width="700px" height="350px"/>

</p>

<br/>
