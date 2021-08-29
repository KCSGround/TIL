# Room

`Room`은 `SQLite`에 대한 추상화 레이어를 제공하여 원활한 데이터베이스 액세스를 지원하는 동시에 `SQLite`를 완벽히 활용한다.

상당한 양의 구조화된 데이터를 처리하는 앱은 데이터를 로컬로 유지하여 대단한 이점을 얻을 수 있다. 가장 일반적인 사용 사례는 관련 데이터를 캐싱하는 것이다. 이런 방식으로 기기가 네트워크에 액세스할 수 없을 때 오프라인 상태인 동안에도 사용자가 여전히 콘텐츠를 탐색할 수 있다. 나중에 기기가 다시 온라인 상태가 되면 사용자가 시작한 콘텐츠 변경사항이 서버에 동기화된다.

`Room`은 이러한 문제를 자동으로 처리하므로 `SQLite` 대신 `Room`을 사용할 것을 적극적으로 권장한다.

앱에서 `Room`을 사용하려면 앱의 `build.gradle` 파일에 다음 종속 항목을 추가한다.

```s
dependencies {
    def room_version = "2.3.0"

    implementation("androidx.room:room-runtime:$room_version")
    annotationProcessor "androidx.room:room-compiler:$room_version"

    // To use Kotlin annotation processing tool (kapt)
    kapt("androidx.room:room-compiler:$room_version")
    // To use Kotlin Symbolic Processing (KSP)
    ksp("androidx.room:room-compiler:$room_version")

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation("androidx.room:room-ktx:$room_version")

    // optional - RxJava2 support for Room
    implementation "androidx.room:room-rxjava2:$room_version"

    // optional - RxJava3 support for Room
    implementation "androidx.room:room-rxjava3:$room_version"

    // optional - Guava support for Room, including Optional and ListenableFuture
    implementation "androidx.room:room-guava:$room_version"

    // optional - Test helpers
    testImplementation("androidx.room:room-testing:$room_version")

    // optional - Paging 3 Integration
    implementation("androidx.room:room-paging:2.4.0-alpha04")
}
```

<br/>

Room에는 다음과 같은 세 가지 주요 구성요소가 있다.

- 데이터베이스 : 데이터베이스 홀더를 포험하며 앱의 지속적인 관계형 데이터의 기본 연결을 위한 기본 엑세스 포인트 역할을 한다.

  `@Database` 로 주석이 지정된 클래스는 다음 조건을 충족해야 합니다.

  - `RoomDatabase` 를 확장하는 추상 클래스여야 한다.
  - 주석 내에 데이터베이스와 연결된 항목의 목록을 포함해야 한다.
  - 인수가 0개이며 `@Dao` 로 주석이 지정된 클래스를 반환하는 추상 메서드를 포함해야 한다.

- 항목: 데이터베이스 내의 테이블을 나타낸다.

- DAO: 데이터베이스에 액세스하는 데 사용되는 메서드가 포함되어 있다.

앱은 `Room` 데이터베이스를 사용하여 데이터베이스와 연결된 데이터 액세스 개체 또는 `DAO`를 가져온다. 그런 다음 앱은 각 `DAO`를 사용하여 데이터베이스에서 항목을 가져오고 항목의 변경사항을 다시 데이터베이스에 저장한다. 마지막으로 앱은 항목을 사용하여 데이터베이스 내의 테이블 열에 해당하는 값을 가져오고 설정한다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/room_architecture.PNG" width="600px" height="540px"/>

</p>

**Room 아키텍처 다이어그램**

<br>

다음 코드 스니펫에는 하나의 항목과 하나의 DAO가 있는 데이터베이스 구성 샘플이 포함되어 있다.

**User**

```kotlin
 @Entity
    data class User(
        @PrimaryKey val uid: Int,
        @ColumnInfo(name = "first_name") val firstName: String?,
        @ColumnInfo(name = "last_name") val lastName: String?
    )
```

<br/>

**UserDao**

```kotlin
    @Dao
    interface UserDao {
        @Query("SELECT * FROM user")
        fun getAll(): List<User>

        @Query("SELECT * FROM user WHERE uid IN (:userIds)")
        fun loadAllByIds(userIds: IntArray): List<User>

        @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
               "last_name LIKE :last LIMIT 1")
        fun findByName(first: String, last: String): User

        @Insert
        fun insertAll(vararg users: User)

        @Delete
        fun delete(user: User)
    }

```

<br/>

**AppDatabase**

```kotlin
   @Database(entities = arrayOf(User::class), version = 1)
    abstract class AppDatabase : RoomDatabase() {
        abstract fun userDao(): UserDao
    }
```

<br/>

위 파일들을 생성한 후에 다음 코드를 사용해서 생성한 데이터베이스 인스턴스를 가져온다.

```kotlin
    val db = Room.databaseBuilder(
                applicationContext,
                AppDatabase::class.java, "database-name"
            ).build()
```

앱이 단일 프로세스에서 실행되면 AppDatabase 객체를 인스턴스화할 때 싱글톤 디자인 패턴을 따라야 한다. 각 RoomDatabase 인스턴스는 리소스를 상당히 많이 소비한다. 그리고 단일 프로세스 내에서 여러 인스턴스에 액세스할 필요가 거의 없다.
