# Design Pattern - MVC

## 디자인 패턴?

`디자인 패턴(Design pattern)`은 건축학 및 컴퓨터 과학에서 사용되는 용어로, 설계 문제에 대한 해답을 문서화하기위해 고안된 형식 방법이다. 이 방식은 건축가 크리스토퍼 알렉산더가 건축학 영역에서 고안한 것을 그 시초로 하며, 이후 컴퓨터 과학 등 여러 다른 분야에도 도입되었다.

소프트웨어 개발 방법에서 사용되는 디자인 패턴은 프로그램 개발에서 자주 나타나는 과제를 해결하기 위한 방법 중 하나로, 과거의 소프트웨어 개발 과정에서 발견된 설계의 노하우를 축적하여 이름을 붙여, 이후에 재이용하기 좋은 형태로 특정의 규약을 묶어서 정리한 것이다. 알고리즘과 같이 프로그램 코드로 바로 변환될 수 있는 형태는 아니지만, 특정한 상황에서 구조적인 문제를 해결하는 방식을 설명해 준다.

- 출처 : https://ko.wikipedia.org/wiki/%EB%94%94%EC%9E%90%EC%9D%B8_%ED%8C%A8%ED%84%B4

<br/>

## MVP

MVP 패턴이란 `Model, View, Presenter`의 약자이다. MVP의 핵심 설계는 [MVC](https://github.com/KCSGround/TIL/blob/master/Design/MVC-Pattern.md)와는 다르게 `UI(View)` 와 `비즈니스 로직(Model)` 을 분리하고, 서로 간에 상호작용을 다른 `객체(Presenter)` 에 그 역할을 줌으로써 서로의 영향(의존성)을 최소화하는 것에 있다.

## MVP in Android

- **Model**

  - 프로그램 내부적으로 쓰이는 데이터를 저장하고, 처리하는 역할을 함.(비즈니스 로직)
  - View 또는 Presenter 등 다른 어떤 요소에도 의존적이지 않은 독립적인 영역임.
    - `비즈니스 로직(Business logic)`은 컴퓨터 프로그램의 규칙에 따라 데이터를 생성·표시·저장·변경하는 부분을 말합니다. 데이터베이스, 표시장치 등 프로그램의 다른 부분과 대조되는 개념으로 쓰입니다.

- **View**

  - UI를 담당하며 안드로이드에서는 Activity, Fragment가 대표적인 예.
  - Model에서 처리된 데이터를 Presenter를 통해 받아서 유저에게 보여줌.
  - 유저 액션(Action) 및 액티비티 라이프사이클 상태 변경을 주시하며 Presenter에 보내는 역할임.
  - Presenter를 이용해 데이터를 주고받기 때문에 Presenter에 매우 의존적임.

- **Presenter**
  - Model과 View사이의 매개체.
  - 모델과 뷰를 매개체라는 점에서 Controller와 유사하지만, View에 직접 연결되는 대신 인터페이스를 통해 상호작용 한다는 점이 다름.
  - 인터페이스를 통해 상호작용 하므로 MVC가 가진 테스트 문제와 함께 모듈화/유연성 문제 역시 해결할 수 있음.
  - 뷰에게 표시할 내용(Data)만 전달하며 어떻게 보여줄지는 View가 담당.

출처: https://faith-developer.tistory.com/71 [개발 이야기]

<br>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/MVP-pattern-1.PNG" width="500px" height="260px"/>

<img src="https://github.com/KCSGround/TIL/blob/master/assets/MVP-pattern-2.PNG" width="500px" height="415px"/>

</p>

<br/>

## MVP 장점

`MVC` 와는 다르게 코드가 매우 깔끔 해지며 `MVP` 를 이용해서 이와 같이 Model과 View 간의 결합도를 낮추면, 새로운 기능 추가 및 변경을 해야 할 때 관련된 해당 부분만 코드 수정하면 된다. 확장성이 좋아짐과 동시에 유닛 테스트 시 테스트 코드를 작성하기 편리해지기 때문에 더 쉽게 안전한 코딩이 가능해진다. 그리고 UI, Data 각각 파트를 나누기 때문에 해야 할 일이 명확해지고 그 결과로 쉽고 빠르게 코딩이 가능하다.

<br/>

## MVP 단점

가장 큰 단점은 애플리케이션이 복잡해질수록 `View` 와 `Presenter` 사이의 의존성이 강해지는 단점이 있다. 그리고 `MVC` 의 `Controller`처럼 `Presenter`도 어느 정도 시간이 지남에 따라 추가 비즈니스 로직이 집중되는 경향이 있다.

개발자는 시간이 지난 어느 순간 거대해지며 동시에 다루기도 어렵고, 문제가 발생하기 쉽고, 서로 간의 분리를 하기도 어려운 `Presenter`를 발견하게 된다. 물론 초기에 설계/기획을 잘함과 동시에 유능한 개발자라면 시간의 흐름에 따른 앱의 다양한 변화에 맞춰서 이 문제를 해결해나갈 수 있겠지만 실제로는 쉽지 않다.

---

## 참고문서

- https://salix97.tistory.com/205

- https://velog.io/@jojo_devstory/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%ED%8C%A8%ED%84%B4-MVP%EA%B0%80-%EB%AD%98%EA%B9%8C

- https://beomy.tistory.com/43
