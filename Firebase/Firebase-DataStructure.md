# Firebase Data Structuring

## 데이터를 구조화하는 방법 : JSON 트리

모든 Firebase 실시간 데이터베이스 데이터는 JSON 객체로 저장된다. 데이터베이스를 클라우드 호스팅 JSON 트리라고 생각하면 된다. SQL 데이터베이스와 달리 테이블이나 레코드가 없으며, JSON 트리에 추가된 데이터는 연결된 키를 갖는 기존 JSON 구조의 노드가 된다. 사용자 ID 또는 의미 있는 이름과 같은 고유 키를 직접 지정할 수도 있고, `push()` 를 사용하여 자동으로 지정할 수도 있다.

<br/>

예를 들어 사용자가 기본적인 프로필과 연락처 목록을 저장할 수 있는 채팅 애플리케이션을 가정해 보겠다. 사용자 프로필은 일반적으로 `/users/$uid` 와 같은 경로에 위치한다. 사용자 `alovelace` 의 데이터베이스 항목은 다음과 같이 표시된다.

```json
{
  "users": {
    "alovelace": {
      "name": "Ada Lovelace",
      "contacts": { "ghopper": true },
    },
    "ghopper": { ... },
    "eclarke": { ... }
  }
}
```

데이터베이스는 JSON 트리를 사용하지만 데이터베이스에 저장되는 데이터는 사용 가능한 JSON 유형에 대응하는 특정한 기본 유형으로 표현하여 코드를 보다 쉽게 관리할 수 있다.

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/firebase-example.PNG" width="700px" height="415px"/>

</p>

위의 예시처럼 데이터베이스 인스턴스 밑에 여러개의 키 값을 지정해서 넣을 수 있다. 또한 그 안에 또 다른 key : value 형식으로 JSON 데이터를 넣을 수 있게 되어있다.

<br/>

## 데이터 구조 권장사항

### 데이터 중첩 배제

Firebase 실시간 데이터베이스는 최대 32단계의 데이터 중첩을 허용하므로 중첩을 기본 구조로 도입해도 괜찮다고 생각할 수도 있다. 그러나 **데이터베이스의 특정 위치에서 데이터를 가져오면 모든 하위 노드가 함께 검색된다.** 또한 **사용자에게 데이터베이스의 특정 노드에 대한 읽기 또는 쓰기 권한을 부여하면 해당 노드에 속한 모든 데이터에 대한 권한이 함께 부여된다.** 따라서 실제 구현에서는 데이터 구조를 최대한 평면화하는 것이 좋다. (깊이 있는 트리를 구성하지 않고 너비 있는 트리로 구성하는 것이 좋다.)

```json
{
  // This is a poorly nested data architecture, because iterating the children
  // of the "chats" node to get a list of conversation titles requires
  // potentially downloading hundreds of megabytes of messages
  "chats": {
    "one": {
      "title": "Historical Tech Pioneers",
      "messages": {
        "m1": { "sender": "ghopper", "message": "Relay malfunction found. Cause: moth." },
        "m2": { ... },
        // a very long list of messages
      }
    },
    "two": { ... }
  }
}
```

이 중첩 설계에서는 전체 데이터를 반복하는 것이 어렵다. 예를 들어 채팅 대화 제목을 나열하려면 모든 멤버와 메시지를 포함한 전체 chats 트리를 클라이언트에 다운로드해야 한다.

<br/>

### 데이터 구조 평면화

비정규화를 통해 데이터를 서로 다른 경로로 분할하면 필요에 따라 별도의 호출을 통해 효율적으로 다운로드할 수 있다.

```json
{
  // Chats contains only meta info about each conversation
  // stored under the chats's unique ID
  "chats": {
    "one": {
      "title": "Historical Tech Pioneers",
      "lastMessage": "ghopper: Relay malfunction found. Cause: moth.",
      "timestamp": 1459361875666
    },
    "two": { ... },
    "three": { ... }
  },

  // Conversation members are easily accessible
  // and stored by chat conversation ID
  "members": {
    // we'll talk about indices like this below
    "one": {
      "ghopper": true,
      "alovelace": true,
      "eclarke": true
    },
    "two": { ... },
    "three": { ... }
  },

  // Messages are separate from data we may want to iterate quickly
  // but still easily paginated and queried, and organized by chat
  // conversation ID
  "messages": {
    "one": {
      "m1": {
        "name": "eclarke",
        "message": "The relay seems to be malfunctioning.",
        "timestamp": 1459361875337
      },
      "m2": { ... },
      "m3": { ... }
    },
    "two": { ... },
    "three": { ... }
  }
}
```

이제 대화당 몇 바이트만 다운로드하여 채팅방 목록 전체를 반복하면서 메타데이터를 빠르게 가져와서 UI에 채팅방을 나열하거나 표시할 수 있다. 메시지가 도착하면 별도로 가져와서 표시할 수 있으므로 UI가 빠른 반응 속도를 유지한다.
