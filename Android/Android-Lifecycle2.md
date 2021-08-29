# Activity Lifecycle2

## Activity 상태 및 메모리에서 제거

시스템은 RAM에 여유 공간이 필요할 때 프로세스를 종료한다. 시스템이 특정 프로세스를 종료할 가능성은 그 시점의 프로세스 상태에 따라 달라진다. 그리고 프로세스 상태는 프로세스에서 실행되는 활동 상태에 따라 달라진다.

<br>

<p align="center">

<img src="https://github.com/dudwns9331/2021-Summer-Kotlin/blob/master/Summer-Technical-Note/assets/activity-memory.PNG" width="800px" height="300px"/>

</p>

**프로세스 수명 주기와 액티비티 상태 간의 관계**

<br>

시스템은 메모리 공간을 확보하기 위해 절대 활동을 직접 종료하지 않는다. 그 대신, 활동을 실행하는 프로세스를 종료하여 활동뿐만 아니라 프로세스에서 실행되는 다른 모든 작업을 함께 소멸시킨다.

<br/>

## 임시 UI 상태 저장 및 복원

사용자는 활동의 UI 상태가 회전 또는 멀티 윈도우 모드로의 전환과 같은 구성 변경사항이 발생하더라도 동일하게 유지될 것으로 기대한다. 그러나 시스템은 이런 구성 변경이 발생하면 기본적으로 활동을 소멸시켜 활동 인스턴스에 저장된 모든 UI 상태를 제거한다. 마찬가지로 사용자는 일시적으로 앱에서 다른 앱으로 전환했다가 다시 앱으로 돌아왔을 때도 UI 상태가 그대로 유지되기를 기대한다. 그러나 사용자가 나가서 활동이 중단되면 시스템은 애플리케이션의 프로세스를 소멸시킬 수 있다.

시스템 제약으로 인해 활동이 소멸되면 `ViewModel, onSaveInstanceState()` 및/또는 로컬 저장소를 결합하여 사용자의 임시 UI 상태를 보존해야 한다.

원시 데이터 유형이거나 간단한 객체(예: 문자열)와 같이 UI 데이터가 간단하고 가벼울 경우, `onSaveInstanceState()`만으로도 모든 구성 변경 및 시스템이 시작한 프로세스가 종료된 상황에서 UI 상태를 보존할 수 있다. 그러나 `onSaveInstanceState()`가 직렬화/역직렬화 비용을 발생시키기 때문에 대부분의 경우에는 `ViewModel`과 `onSaveInstanceState()`를 모두 사용해야 한다.

### 인스턴스 상태

정상적인 앱 동작으로 인해 활동이 소멸되는 시나리오는 몇 가지가 있다. 예를 들어 사용자가 뒤로 버튼을 누르거나 활동이 `finish()` 메서드를 호출하여 자체적인 소멸 신호를 보내는 경우이다. 사용자가 뒤로 버튼을 누르거나 활동이 자체적으로 종료되어 활동이 소멸되는 경우 해당 `Activity` 인스턴스에 관한 시스템과 사용자의 콘셉트가 모두 영구적으로 사라진다. 이 시나리오에서 사용자의 기대가 시스템의 동작과 일치하므로 추가적인 작업이 필요하지 않다.

그러나 시스템이 시스템 제약(예: 구성 변경 또는 메모리 부족)으로 인해 활동을 소멸시킬 경우, 실제 `Activity` 인스턴스는 사라지더라도 시스템에 존재했다는 정보는 남아 있다. 사용자가 활동으로 다시 돌아가려고 시도하면 시스템은 소멸 당시 활동의 상태를 설명하는 저장된 데이터 세트를 사용하여 해당 활동의 새로운 인스턴스를 생성한다.

시스템이 이전 상태를 복원하기 위해 사용하는 저장된 데이터를 인스턴스 상태라고 하며, 이는 `Bundle` 객체에 저장된 키-값 쌍의 컬렉션이다. 기본적으로 시스템은 `Bundle` 인스턴스 상태를 사용하여 활동 레이아웃의 각 `View` 객체 관련 정보를 저장한다(예: `EditText` 위젯에 입력된 텍스트 값). 따라서 활동 인스턴스가 소멸되고 재생성된 경우, 레이아웃의 상태는 별도의 코드 요청 없이 이전 상태로 복원된다. 하지만 활동에서 사용자 진행 상태를 추적하는 멤버 변수처럼 활동에 복원하고자 하는 상태 정보가 더 많이 있는 경우도 있다.

`Bundle` 객체는 메인 스레드에서 직렬화되어야 하고 시스템 프로세스 메모리를 사용하므로 소량의 데이터를 보존하는 데만 적합하다. 극소량 이상의 데이터를 보존하려면 영구 로컬 저장소, `onSaveInstanceState()` 메서드, `ViewModel` 클래스로 데이터를 보존하는 복합적인 방법을 사용해야 한다.

### `onSaveInstanceState()`를 사용하여 간단하고 가벼운 UI 상태 저장

액티비티가 정지되기 시작하면 인스턴스 상태 번들에 상태 정보를 저장할 수 있도록 시스템이 `onSaveInstanceState()` 메서드를 호출한다. 이 메서드의 기본 구현은 `EditText` 위젯 내 텍스트 또는 `ListView` 위젯의 스크롤 위치와 같은 활동의 뷰 계층 구조에 대한 임시 정보를 저장한다.

액티비티의 추가적인 인스턴스 상태 정보를 저장하려면 `onSaveInstanceState()`를 재정의하고, 액티비티가 예상치 못하게 소멸될 경우 저장되는 `Bundle` 객체에 키-값 쌍을 추가해야 한다. `onSaveInstanceState()`를 재정의할 경우 기본 구현에서 뷰 계층 구조의 상태를 저장하고자 한다면 상위 클래스 구현을 호출해야 한다.

다음은 그 예시이다.

```kotlin
override fun onSaveInstanceState(outState: Bundle?) {
    // Save the user's current game state
    outState?.run {
        putInt(STATE_SCORE, currentScore)
        putInt(STATE_LEVEL, currentLevel)
    }

    // Always call the superclass so it can save the view hierarchy state
    super.onSaveInstanceState(outState)
}

companion object {
    val STATE_SCORE = "playerScore"
    val STATE_LEVEL = "playerLevel"
}
```

<br/>

### 저장된 인스턴스 상태를 사용하여 활동 UI 상태 복원

액티비티가 이전에 소멸된 후 재생성되면, 시스템이 활동에 전달하는 `Bundle` 로부터 저장된 인스턴스 상태를 복구할 수 있다. `onCreate()` 및 `onRestoreInstanceState()` 콜백 메서드 둘 다 인스턴스 상태 정보를 포함하는 동일한 `Bundle` 을 수신한다.

`onCreate()` 메서드는 시스템이 활동의 새 인스턴스를 생성하든, 이전 인스턴스를 재생성하든 상관없이 호출되므로 읽기를 시도하기 전에 번들 상태가 `null` 인지 반드시 확인해야 한다. `null` 일 경우, 시스템은 이전에 소멸된 활동의 인스턴스를 복원하지 않고 새 인스턴스를 생성한다.

예를 들어 다음 코드 스니펫은 `onCreate()`에서 일부 상태 데이터를 복원하는 방법을 보여준다.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState) // Always call the superclass first

    // Check whether we're recreating a previously destroyed instance
    if (savedInstanceState != null) {
        with(savedInstanceState) {
            // Restore value of members from saved state
            currentScore = getInt(STATE_SCORE)
            currentLevel = getInt(STATE_LEVEL)
        }
    } else {
        // Probably initialize members with default values for a new instance
    }
    // ...
}
```

`onCreate()` 중에 상태를 복원하는 대신, `onRestoreInstanceState()`를 구현하는 방법을 선택할 수 있다. 이는 시스템이 `onStart()` 메서드 다음에 호출한다. 시스템은 복원할 저장 상태가 있을 경우에만 `onRestoreInstanceState()`를 호출한다. 따라서 `Bundle`이 `null` 인지 확인할 필요가 없다.

<br/>

## 활동 간 이동

앱은 수명 주기 동안 활동에 여러 번 들어가고 나올 가능성이 크다. 예를 들어 사용자가 기기의 뒤로 버튼을 누르거나 활동에서 다른 활동을 시작할 수 있다.

<br/>

### 다른 활동에서 활동시작하기

액티비티는 특정 시점에서 다른 액티비티를 시작해야 하는 경우가 많다. 예를 들어 앱이 현재 화면에서 새로운 화면으로 옮겨갈 때가 이에 해당한다.

액티비티가 새로 시작하려는 액티비티에서 결과를 받기를 원하는지 여부에 따라 `startActivity()` 나 `startActivityForResult()` 메서드 중 하나를 사용하여 새 액티비티를 시작합니다. 두 가지 경우 모두 `Intent` 객체를 전달한다.

`Intent` 객체는 시작하고자 하는 액티비티를 정확히 나타내거나, 실행하고자 하는 작업의 유형을 설명한다(시스템이 적절한 액티비티를 선택하며, 이는 다른 애플리케이션에서 가져온 것일 수도 있다). `Intent` 객체는 시작된 액티비티에서 사용할 소량의 데이터를 포함할 수 있다.

<br/>

### startActivity()

새로 시작된 액티비티가 결과를 반환할 필요가 없을 경우 현재 액티비티가 `startActivity()` 메서드를 호출하면 이를 시작할 수 있다.

본인의 애플리케이션 내에서 작업하는 경우에는 알려진 액티비티를 시작하기만 하면 되는 경우가 많다.

또한 애플리케이션에서 다른 몇 가지 작업을 수행하고자 할 수도 있다. 예를 들어 이메일 보내기, SMS 보내기 또는 상태 업데이트 등의 작업을 활동의 데이터를 사용하여 수행할 수 있다. 이 경우, 본인의 애플리케이션에 그러한 동작을 실행할 자체 액티비티가 없을 수도 있다. 따라서 기기에 있는 다른 애플리케이션이 제공하는 활동을 대신 활용하여 동작을 실행하게 할 수 있다. 바로 이 부분에서 인텐트의 진가가 발휘된다. 실행하고자 하는 동작을 설명하는 인텐트를 생성하면 시스템이 적절한 활동을 다른 애플리케이션에서 시작한다. 인텐트를 처리할 수 있는 액티비티가 여러 개 있는 경우, 사용자는 어느 것을 사용할지 선택할 수 있다.

다음은 이메일 전송에 대한 예시이다.

```kotlin
val intent = Intent(Intent.ACTION_SEND).apply {
    putExtra(Intent.EXTRA_EMAIL, recipientArray)
}
startActivity(intent)
```

<br/>

### startActivityForResult()

액티비티가 종료될 때 결과를 반환받고자 할 수도 있다. 예를 들어 사용자가 연락처 목록에서 어떤 사람을 선택할 수 있도록 하는 액티비티를 시작할 수 있다. 이 액티비티가 종료되면 선택한 사람을 반환한다. 이렇게 하려면 `startActivityForResult(Intent, int)` 메서드를 호출해야 한다. 이 메서드에서는 정수 매개변수가 호출을 식별한다. 이 식별자는 동일한 액티비티에서 여러 개의 `startActivityForResult(Intent, int)` 호출을 명확히 구분하는 역할을 한다. 이는 글로벌 식별자가 아니며, 다른 앱이나 액티비티와 충돌할 위험이 없다. 결과는 `onActivityResult(int, int, Intent)` 메서드를 통해 반환된다.

하위 액티비티가 존재하면 `setResult(int)`를 호출하여 상위 활동으로 데이터를 반환할 수 있다. 하위 액티비티는 항상 결과 코드를 제공한다. 이는 표준 결과인 `RESULT_CANCELED, RESULT_OK`가 되거나 `RESULT_FIRST_USER`로 시작하는 임의의 맞춤 값이 될 수도 있다. 또한 하위 액티비티는 원하는 모든 추가 데이터가 포함된 `Intent` 객체를 반환할 수도 있다. 상위 액티비티는 원래 제공했던 정수 식별자와 함께 `onActivityResult(int, int, Intent)` 메서드를 사용하여 정보를 수신한다.

어떤 이유(예: 비정상 종료)로든 하위 액티비티가 실행되지 않으면 상위 액티비티는 `RESULT_CANCELED` 코드가 포함된 결과를 수신한다.

```kotlin
class MyActivity : Activity() {
    // ...

    override fun onKeyDown(keyCode: Int, event: KeyEvent?): Boolean {
        if (keyCode == KeyEvent.KEYCODE_DPAD_CENTER) {
            // When the user center presses, let them pick a contact.
            startActivityForResult(
                    Intent(Intent.ACTION_PICK,Uri.parse("content://contacts")),
                    PICK_CONTACT_REQUEST)
            return true
        }
        return false
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, intent: Intent?) {
        when (requestCode) {
            PICK_CONTACT_REQUEST ->
                if (resultCode == RESULT_OK) {
                    startActivity(Intent(Intent.ACTION_VIEW, intent?.data))
                }
        }
    }

    companion object {
        internal val PICK_CONTACT_REQUEST = 0
    }
}
```
