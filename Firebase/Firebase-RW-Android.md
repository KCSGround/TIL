# Firebase Read & Write in Android

`Firebase` 데이터는 `FirebaseDatabase` 참조에 기록되고, 참조에 비동기 리스너를 연결하여 검색할 수 있다. 리스너는 데이터의 초기 상태가 확인될 때 한 번 트리거된 후 데이터가 변경될 때마다 다시 트리거된다.

## DatabaseReference 가져오기

데이터베이스에서 데이터를 읽거나 쓰려면 DatabaseReference 의 인스턴스가 필요하다.

```kotlin

private lateinit var database: DatabaseReference
// ...

// 데이터베이스 레퍼런스 인스턴스 가져오기
database = Firebase.database.reference
```

데이터베이스에서 데이터를 읽거나 쓰려면 `DatabaseReference` 의 인스턴스가 필요하다.

- [Firebase 를 Android 프로젝트와 연동하는 방법 바로가기](https://github.com/KCSGround/TIL/blob/master/Firebase/Firebase-Realtime-Android.md)

<br/>

## 데이터 쓰기

### 기본 쓰기 작업

기본 쓰기 작업은 `setValue()` 코드를 사용하여 지정된 참조에 데이터를 저장하고 해당 경로의 기존 데이터를 모두 바꾼다. 이 메서드의 용도는 다음과 같다.

사용 가능한 JSON 유형에 해당하는 다음과 같은 유형을 전달한다.

- `String, Long, Double, Boolean, Map<String, Object>, List<Object>, 커스텀 자바 객체`

자바 객체를 사용하는 경우 객체의 내용이 하위 위치에 중첩된 형태로 자동으로 매핑된다. 또한 자바 객체를 사용하면 일반적으로 코드가 단순해지고 관리하기도 쉬워진다.

```kotlin
@IgnoreExtraProperties
data class User(val username: String? = null, val email: String? = null) {
    // Null default values create a no-argument default constructor, which is needed
    // for deserialization from a DataSnapshot.
}
```

다음과 같이 `setValue()` 로 사용자를 추가할 수 있다.

```kotlin
fun writeNewUser(userId: String, name: String, email: String) {
    val user = User(name, email)

    database.child("users").child(userId).setValue(user)
}
```

데이터베이스 밑에 최상위 `users` 가 생성되고 그 안에 `userId`, 그리고 그 안에 `user` 객체가 들어간다. `user` 객체는 `name` 과 `email` 이 있으므로 동일선상에 `name`, `email` 이 생성된다.

이 방법으로 `setValue()` 를 사용하면 지정된 위치에서 하위 노드를 포함하여 모든 데이터를 덮어쓴다. 그러나 전체 객체를 다시 쓰지 않고도 하위 항목을 업데이트하는 방법이 있다. 사용자가 프로필을 업데이트할 수 있도록 하려면 다음과 같이 사용자 이름을 업데이트하면 된다.

```kotlin
database.child("users").child(userId).child("username").setValue(name)
```

<br/>

## 데이터 읽기

### 영구 리스너로 데이터 읽기

경로에서 데이터를 읽고 변경사항을 수신 대기하려면 `addValueEventListener()` 메서드를 사용하여 `DatabaseReference` 를 `ValueEventListener` 에 추가해야 한다.

`ValueEventListener` 는 `onDataChange()` 를 콜백으로 가지고 용도는 경로의 전체 내용을 읽고 변경사항을 수신 대기한다.

`onDataChange()` 메서드를 사용하여 이벤트 발생 시점에 특정 경로에 있던 콘텐츠의 정적 스냅샷을 읽을 수 있다. 이 메서드는 리스너가 연결될 때 한 번 트리거된 후 하위 요소를 포함한 데이터가 변경될 때마다 다시 트리거된다. 하위 데이터를 포함하여 해당 위치의 모든 데이터를 포함하는 스냅샷이 이벤트 콜백에 전달된다. 데이터가 없는 경우 스냅샷은 `exists()` 호출 시 `false` 를 반환하고 `getValue()` 호출 시 `null` 을 반환한다.

<br/>

### 데이터 한 번 읽기

**get()을 사용하여 한 번 읽기**

SDK는 앱이 온라인이든 오프라인이든 상관없이 데이터베이스 서버와의 상호작용을 관리하도록 설계되었다.

일반적으로 위에서 설명한 ValueEventListener 기법을 사용하여 데이터를 읽어 백엔드에서 데이터에 대한 업데이트 알림을 수신해야 한다. 리스너 기법은 사용량과 결제를 줄여주며 사용자가 온라인과 오프라인으로 진행할 때 최상의 환경을 제공하도록 최적화되어 있다.

데이터가 한 번만 필요한 경우 `get()` 을 사용하여 데이터베이스에서 데이터의 스냅샷을 가져올 수 있다. 어떠한 이유로든 `get()` 이 서버 값을 반환할 수 없는 경우 클라이언트는 로컬 스토리지 캐시를 프로브하고 값을 여전히 찾을 수 없으면 오류를 반환한다.

불필요한 `get()` 사용은 대역폭 사용을 증가시키고 성능 저하를 유발할 수 있지만 위와 같이 실시간 리스너를 사용하면 이를 방지할 수 있다.

```kotlin
mDatabase.child("users").child(userId).get().addOnSuccessListener {
    Log.i("firebase", "Got value ${it.value}")
}.addOnFailureListener{
    Log.e("firebase", "Error getting data", it)
}
```

<br/>

**리스너를 사용하여 한 번 읽기**

경우에 따라 서버의 업데이트된 값을 확인하는 대신 로컬 캐시의 값을 즉시 반환하고 싶을 수 있다. 이 경우에는 `addListenerForSingleValueEvent` 을 사용하여 로컬 디스크 캐시에서 데이터를 즉시 가져올 수 있다.

이 방법은 한 번 로드된 후 자주 변경되지 않거나 능동적으로 수신 대기할 필요가 없는 데이터에 유용하다.

<br/>

## 데이터 업데이트 또는 삭제

### 특정 필드 업데이트

다른 하위 노드를 덮어쓰지 않고 특정 하위 노드에 동시에 쓰려면 `updateChildren()` 메서드를 사용한다.

`updateChildren()`를 호출할 때 키 경로를 지정하여 더 낮은 수준의 하위 항목 값을 업데이트할 수 있다. 확장성 개선을 위해 데이터를 여러 위치에 저장한 경우 데이터 팬아웃을 사용하여 해당 데이터의 모든 인스턴스를 업데이트할 수 있다.

<br/>

### 완료 콜백 추가

데이터가 커밋되는 시점을 파악하려면 완료 리스너를 추가한다. `setValue()` 및 `updateChildren()`은 모두 쓰기가 데이터베이스에 커밋될 때 호출되는 선택적 완료 리스너를 사용한다. 호출이 실패하면 실패 이유를 나타내는 오류 객체가 리스너로 전달된다.

```kotlin
database.child("users").child(userId).setValue(user)
        .addOnSuccessListener {
            // Write was successful!
            // ...
        }
        .addOnFailureListener {
            // Write failed
            // ...
        }
```

<br/>

### 데이터 삭제

데이터를 삭제하는 가장 간단한 방법은 해당 데이터 위치의 참조에 `removeValue()` 를 호출하는 것입니다.

`setValue()` 또는 `updateChildren()` 등의 다른 쓰기 작업 값으로 `null` 을 지정하여 삭제할 수도 있다. `updateChildren()`에 이 방법을 사용하면 API 호출 한 번으로 여러 하위 항목을 삭제할 수 있다.
