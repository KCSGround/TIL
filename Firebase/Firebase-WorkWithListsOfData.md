# Firebase Work with Lists of Data on Android(안드로이드에서 데이터 목록 다루기)

기본적으로 인증된 사용자만 데이터를 읽고 쓸 수 있도록 데이터베이스에 대한 읽기 및 쓰기 액세스 권한이 제한된다. 공개 액세스 규칙을 구성하면 인증을 설정하지 않고 시작할 수 있다. 다만 이렇게 하면 앱을 사용하지 않는 사람을 포함하여 모두에게 데이터베이스가 공개되므로 인증을 설정할 때 데이터베이스를 다시 제한해야 한다. **보안 규칙에 따라서 사용할 수 있는 사용자가 나누어진다.**

<br/>

## DatabaseReference 가져오기

데이터베이스에서 데이터를 읽고 쓰려면 `DatabaseReference` 인스턴스가 필요하다.

```kotlin
private lateinit var database: DatabaseReference
// ...
database = Firebase.database.reference
```

<br/>

## 목록 읽기 및 쓰기

### 데이터 목록에 추가

멀티 사용자 애플리케이션에서 `push()` 메서드를 사용하여 목록에 데이터를 추가한다. `push()` 메서드는 지정된 Firebase 참조에 새 하위 요소가 추가될 때마다 고유 키를 생성한다. 목록의 새 요소마다 이러한 자동 생성 키를 사용하면 여러 클라이언트에서 쓰기 충돌 없이 동시에 같은 위치에 하위 요소를 추가할 수 있다. `push()`가 생성하는 고유 키는 타임스탬프에 기반하므로 목록 항목은 시간순으로 자동 정렬된다.

`push()` 메서드가 반환하는 새 데이터에 대한 참조를 사용하여 하위 요소의 자동 생성 키 값을 가져오거나 하위 데이터를 설정할 수 있다. `push()` 참조에 대해 `getKey()`를 호출하면 자동 생성 키 값이 반환된다.

<br/>

### 하위 이벤트 수신 대기

목록을 다루는 애플리케이션은 단일 객체에 사용되는 값 이벤트가 아닌 하위 이벤트를 수신 대기해야 한다.

하위 이벤트는 노드의 하위 요소에서 발생하는 특정 작업에 대응하여 트리거된다. 하위 요소가 `push()` 메서드를 통해 새로 추가되거나 `updateChildren()` 메서드를 통해 업데이트되는 경우가 그 예시이다. 이러한 메서드는 데이터베이스의 특정 노드에 대한 변경사항을 수신 대기하는 데 유용할 수 있다.

`DatabaseReference` 의 하위 이벤트를 수신 대기하려면 `ChildEventListener` 를 연결한다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/childEventListener.PNG" width="700px" height="300px"/>

</p>

<br/>

예시로 소셜 블로깅 앱은 아래와 같이 이러한 메서드를 함께 사용하여 게시물의 댓글 활동을 모니터링할 수 있다.

```kotlin
val childEventListener = object : ChildEventListener {
    override fun onChildAdded(dataSnapshot: DataSnapshot, previousChildName: String?) {
        Log.d(TAG, "onChildAdded:" + dataSnapshot.key!!)

        // A new comment has been added, add it to the displayed list
        val comment = dataSnapshot.getValue<Comment>()

        // ...
    }

    override fun onChildChanged(dataSnapshot: DataSnapshot, previousChildName: String?) {
        Log.d(TAG, "onChildChanged: ${dataSnapshot.key}")

        // A comment has changed, use the key to determine if we are displaying this
        // comment and if so displayed the changed comment.
        val newComment = dataSnapshot.getValue<Comment>()
        val commentKey = dataSnapshot.key

        // ...
    }

    override fun onChildRemoved(dataSnapshot: DataSnapshot) {
        Log.d(TAG, "onChildRemoved:" + dataSnapshot.key!!)

        // A comment has changed, use the key to determine if we are displaying this
        // comment and if so remove it.
        val commentKey = dataSnapshot.key

        // ...
    }

    override fun onChildMoved(dataSnapshot: DataSnapshot, previousChildName: String?) {
        Log.d(TAG, "onChildMoved:" + dataSnapshot.key!!)

        // A comment has changed position, use the key to determine if we are
        // displaying this comment and if so move it.
        val movedComment = dataSnapshot.getValue<Comment>()
        val commentKey = dataSnapshot.key

        // ...
    }

    override fun onCancelled(databaseError: DatabaseError) {
        Log.w(TAG, "postComments:onCancelled", databaseError.toException())
        Toast.makeText(context, "Failed to load comments.",
                Toast.LENGTH_SHORT).show()
    }
}
databaseReference.addChildEventListener(childEventListener)
```

<br/>

### 값 이벤트 수신 대기

데이터 목록을 읽을 때 `ChildEventListener` 를 사용하는 것이 좋지만 목록 참조에 `ValueEventListener` 를 연결하는 것이 유용한 상황도 있다.

데이터 목록에 `ValueEventListener` 를 연결하면 전체 데이터 목록이 단일 `DataSnapshot` 으로 반환되며 이를 루프 처리하여 개별 하위 요소에 액세스할 수 있다.

쿼리에 일치하는 항목이 단 한 개인 경우에도 스냅샷은 목록으로 표시된다. 단지 항목이 한 개 있을 뿐이다. 이 항목에 액세스하려면 결과를 루프 처리해야 한다.

```kotlin
val myTopPostsQuery = databaseReference.child("user-posts").child(myUserId)
            .orderByChild("starCount")
// My top posts by number of stars
myTopPostsQuery.addValueEventListener(object : ValueEventListener {
    override fun onDataChange(dataSnapshot: DataSnapshot) {
        for (postSnapshot in dataSnapshot.children) {
            // TODO: handle the post
        }
    }

    override fun onCancelled(databaseError: DatabaseError) {
        // Getting Post failed, log a message
        Log.w(TAG, "loadPost:onCancelled", databaseError.toException())
        // ...
    }
})
```

`onChildAdded` 이벤트를 추가로 수신 대기하는 대신 한 번의 작업으로 목록의 모든 하위 요소를 가져오려는 경우 이 패턴이 유용할 수 있다.

<br/>

## 리스너 분리

Firebase 데이터베이스 참조에 대해 `removeEventListener()` 메서드를 호출하면 콜백이 삭제된다.

한 데이터 위치에 리스너를 여러 번 추가하면 각 이벤트가 발생할 때마다 리스너가 여러 번 호출되며 리스너를 완전히 삭제하려면 동일한 횟수만큼 리스너를 분리해야 한다.

상위 리스너에 `removeEventListener()`를 호출해도 하위 노드에 등록된 리스너가 자동으로 삭제되지 않는다. 하위 리스너에도 `removeEventListener()`를 호출하여 콜백을 삭제해야 한다.

<br/>

## 데이터 정렬 및 필터링

실시간 데이터베이스의 `Query` 클래스를 사용하여 키, 값 또는 하위 요소 값으로 정렬된 데이터를 검색할 수 있다. 정렬된 결과를 특정 개수로 필터링하거나 키 또는 값 범위에 따라 필터링할 수도 있다.

<br/>

### 데이터 정렬

정렬된 데이터를 검색하려면 우선 정렬 기준 메서드 중 하나를 지정하여 결과를 어떤 순서로 정렬할지 결정한다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/dataSortMethod.PNG" width="700px" height="150px"/>

</p>

<br/>

정렬 기준 메서드는 한번에 하나만 사용할 수 있다. 동일한 쿼리에서 정렬 기준 메서드를 여러 번 호출하면 오류가 발생한다.

다음 예시에서는 별표 수를 기준으로 사용자의 최상위 게시물 목록을 검색하는 방법을 보여준다.

```kotlin
// My top posts by number of stars
val myUserId = uid
val myTopPostsQuery = databaseReference.child("user-posts").child(myUserId)
    .orderByChild("starCount")

myTopPostsQuery.addChildEventListener(object : ChildEventListener {
    // TODO: implement the ChildEventListener methods as documented above
    // ...
})
```

여기에서는 쿼리를 하위 리스너와 함께 사용하여 사용자 ID를 기준으로 데이터베이스의 특정 경로에 있는 사용자의 게시물을 각 게시물이 받는 별표 수에 따라 정렬한 결과를 클라이언트와 동기화하도록 쿼리를 정의한다. 이와 같이 ID를 색인 키로 사용하는 기법을 데이터 팬아웃이라고 한다.

`orderByChild()` 메서드를 호출할 때는 결과를 정렬하는 기준이 될 하위 키를 지정한다. 이 예시에서는 `orderByChild()` 호출에서 중첩된 하위 요소의 상대 경로를 지정하여 `metrics` 키 아래에 중첩된 값에 따라 목록 요소를 정렬할 수 있다.

```kotlin
// Most viewed posts
val myMostViewedPostsQuery = databaseReference.child("posts")
        .orderByChild("metrics/views")
myMostViewedPostsQuery.addChildEventListener(object : ChildEventListener {
    // TODO: implement the ChildEventListener methods as documented above
    // ...
})
```

<br/>

### 데이터 필터링

데이터 필터링하려면 제한 또는 범위 메서드를 정렬 기준 메서드와 조합하여 쿼리를 작성한다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/dataFiltering.PNG" width="700px" height="300px"/>

</p>

<br/>

정렬 기준 메서드와 달리 여러 개의 제한 또는 범위 함수를 조합할 수 있다. 예를 들어 `startAt()` 및 `endAt()` 메서드를 조합하여 결과를 지정된 값 범위로 제한할 수 있다.

쿼리에 일치하는 항목이 단 한 개인 경우에도 스냅샷은 목록으로 표시된다. 단지 항목이 한 개 있을 뿐이다. 이 항목에 액세스하려면 결과를 루프 처리해야 한다.

```kotlin
// My top posts by number of stars
myTopPostsQuery.addValueEventListener(object : ValueEventListener {
    override fun onDataChange(dataSnapshot: DataSnapshot) {
        for (postSnapshot in dataSnapshot.children) {
            // TODO: handle the post
        }
    }

    override fun onCancelled(databaseError: DatabaseError) {
        // Getting Post failed, log a message
        Log.w(TAG, "loadPost:onCancelled", databaseError.toException())
        // ...
    }
})
```

<br/>

### 키 또는 값으로 필터링

`startAt(), startAfter(), endAt(), endBefore(), equalTo()`를 사용하여 쿼리의 시작, 종료, 동일 지점을 임의로 선택할 수 있다. 이 메서드는 특정 값을 갖는 하위 요소로 데이터를 페이지로 나누거나 항목을 찾는 데 유용하다.
