# Scope Functions

## 개념 및 예시

Scope Functions 는 객체의 context 내에서 블록을 실행할 수 있게 하는 함수이다.

`lambda expression` 이 제공된 객체에서 Scope Function 을 호출하면 해당 함수는 일시적인 범위를 형성하며, 이범이에서는 객체 일므 없이 객체에 접근할 수 있다.

Scope Function 의 일반적인 예시는 다음과 같다.

```kotlin
Person("Alice", 20, "Amsterdam").let {
    println(it)
    it.moveTo("London")
    it.incrementAge()
    println(it)
}

Person(name=Alice, age=20, city=Amsterdam)
Person(name=Alice, age=21, city=London)
```

<br/>

만약 Scope Function 을 사용하지 않는다면 위의 코드는 아래와 같이 작성되한다.

```kotlin
val alice = Person("Alice", 20, "Amsterdam")
println(alice)
alice.moveTo("London")
alice.incrementAge()
println(alice)

# 결과
Person(name=Alice, age=20, city=Amsterdam)
Person(name=Alice, age=21, city=London)

```

Scope Function 은 새로운 기술 기능을 도입하지는 않지만 코드를 보다 간결하고 읽기 쉽게 만들 수 있다. 위의 예시에서 사용된 Scope Function 은 `let` 이다.

Kotlin 에서 제공하는 Scope Function 으로는 `let, run, with, apply` 그리고 `also`가 있다. 기본적으로 이 함수들은 객체의 코드 블럭을 실행한다는 공통점을 갖는다. 차이점은 코드 블럭 내에서 객체에 어떻게 접근하는지와 블럭 내 코드의 결과가 무엇인지이다.

## Context object: this or it

Scope Function 의 lambda expression 내부에서 컨텍스트 객체는 실제 이름 대신 짧은 참조를 통해 사용가능하다. 이 짧은 참조는 lambda receiver 인 `this` 혹은 lambda argument인 `it` 이다.

```kotlin
fun main() {
    val str = "Hello"
    // this
    str.run {
        println("The receiver string length: $length")
        //println("The receiver string length: ${this.length}") // does the same
    }
​
    // it
    str.let {
        println("The receiver string's length is ${it.length}")
    }
}

# 결과
The receiver string length: 5
The receiver string's length is 5
```

### this

`run`, `with` 그리고 `apply` 는 컨텍스트 객체를 `this` 를 통해 사용한다. 그래서 객체를 일반적인 클래스 함수 내부에서 사용하는 것처럼 사용할 수 있다. 대부분의 경우 객체 멤버에 할당을 할 때 `this` 를 생략할 수 있다. 다만 외부 객체나 함수와 구분하기 힘들어지므로 `this` 를 사용하는 것을 권장한다.

```kotlin
val adam = Person("Adam").apply {
    age = 20                       // same as this.age = 20 or adam.age = 20
    city = "London"
}
println(adam)

# 결과
Person(name=Adam, age=20, city=London)
```

### it

`let` 과 `also` 는 컨텍스트 객체를 `lambda argument` 처럼 가지고 있다. 따라서 `it` 을 통해 컨텍스트 객체에 접근할 수 있다. `it` 은 `this` 보다 짧고 `it` 을 사용한 표현식은 보통 읽기 쉽다. 하지만 객체의 함수나 프로퍼티를 호출할 때 `this` 처럼 암시적으로 사용할 수는 없다.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}
​
val i = getRandomInt()

# 결과
INFO: getRandomInt() generated value 49
```

<br/>

또한 context 개체를 인수로 전달할 때 범위 내의 context 개체에 대한 사용자 지정 이름을 제공할 수 있다.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also { value ->
        writeToLog("getRandomInt() generated value $value")
    }
}
​
val i = getRandomInt()

# 결과
INFO: getRandomInt() generated value 77
```

<br/>

## Return value

Scope Function 은 반환하는 결과에 따라 다르다.

- apply 와 also 는 context object 를 반환한다.

- let, run, with 람다 결과를 반환한다.

이 두 가지 옵션을 사용하면 코드에서 다음에 수행하는 작업에 따라 적절한 기능을 선택할 수 있다.

### Context object

`apply` 와 `also` 는 해당 코드 블럭의 컨텍스트 객체를 반환한다. 따라서 이 두 함수는 같은 객체에 대한 `call chain` 을 가질 수 있다.

```kotlin
val numberList = mutableListOf<Double>()
numberList.also { println("Populating the list") }
    .apply {
        add(2.71)
        add(3.14)
        add(1.0)
    }
    .also { println("Sorting the list") }
    .sort()

# 결과
Populating the list
Sorting the list
[1.0, 2.71, 3.14]
```

또한 이것들은 return 문에서도 사용할 수 있다.

```kotlin
fun getRandomInt(): Int {
    return Random.nextInt(100).also {
        writeToLog("getRandomInt() generated value $it")
    }
}
​
val i = getRandomInt()

# 결과
INFO: getRandomInt() generated value 35
```

### Lambda result

`let`, `run` 그리고 `with` 는 `lambda result` 를 반환한다. 따라서 이 두 함수의 결과는 변수에 할당될 수 있다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
val countEndsWithE = numbers.run {
    add("four")
    add("five")
    count { it.endsWith("e") }
}
println("There are $countEndsWithE elements that end with e.")

# 결과
There are 3 elements that end with e.
```

추가적으로 `lambda result` 는 무시될 수 있으며, `Scope Function` 을 변수에 대한 임시 범위를 생성하는데 사용할 수 있다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    val firstItem = first()
    val lastItem = last()
    println("First item: $firstItem, last item: $lastItem")
}

# 결과
First item: one, last item: three
```

<br/>

## Functions

### let

let은 컨텍스트 객체를 argument(it)처럼 사용할 수 있고 lambda result를 반환합니다. let은 호출 체인의 결과에 대해 하나 이상의 함수를 호출하는데 사용할 수 있습니다.

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
val resultList = numbers.map { it.length }.filter { it > 3 }
println(resultList)

# 결과
[5, 4, 4]
```

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let {
    println(it)
    // and more function calls if needed
}

# 결과
[5, 4, 4]
```

만약 코드 블럭이 it을 인자로 가지는 함수 하나만 담고 있다면 lambda expression 대신 ::을 사용할 수 있다.

```kotlin
val numbers = mutableListOf("one", "two", "three", "four", "five")
numbers.map { it.length }.filter { it > 3 }.let(::println)

# 결과
[5, 4, 4]
```

```kotlin
val str: String? = "Hello"
//processNonNullString(str)       // compilation error: str can be null
val length = str?.let {
    println("let() called on $it")
    processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
    it.length
}

# 결과
let() called on Hello
```

`let` 은 종종 `non-null` 객체에 코드 블럭을 생성할 때 사용한다. 또 코드 가독성을 높이기 위해 코드 블럭에 지역 변수를 제공할 때 사용한다.

<br/>

### with

`with` 는 `context` 객체를 `argument` 로 전달하지만 내부에선 `this` 를 통해 사용가능하다. 그리고 `lambda result`를 반환한다. `with` 는 코드 블럭 내에서 `lambda result` 반환없이 함수를 호출할 때 사용할 것을 추천한다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
with(numbers) {
    println("'with' is called with argument $this")
    println("It contains $size elements")
}

# 결과
'with' is called with argument [one, two, three]
It contains 3 elements
```

`with` 은 속성이나 기능이 값을 계산하는 데 사용되는 도우미 객체를 사용하는것이다.

<br/>

### run

`run` 은 컨텍스트 객체를 `this` 를 통해 사용하며, `lambda result` 를 반환한다. `run` 은 코드 블럭 내에 객체 초기화와 반환 값 계산을 둘다 담고 있는 경우 유용하다. `run` 은 `with` 과 동일하지만 `context` 객체의 확장 기능으로 let 을 호출한다.

```kotlin
val service = MultiportService("server", 80)
​
val result = service.run {
    port = 8080
    query(prepareRequest() + " to port $port")
}
​
// the same code written with let() function:
val letResult = service.let {
    it.port = 8080
    it.query(it.prepareRequest() + " to port ${it.port}")
}
# 결과
Result for query 'Default request to port 8080'
Result for query 'Default request to port 8080'
```

수신 객체에서 실행을 호출하는 것 외에도 비홪아 함수로 사용할 수 있다. 비확장 실행을 사용하면, 표현식이 필요한 여러 명령문의 블록을 실행할 수 있다.

<br/>

```kotlin
val hexNumberRegex = run {
    val digits = "0-9"
    val hexDigits = "A-Fa-f"
    val sign = "+-"
​
    Regex("[$sign]?[$digits$hexDigits]+")
}
​
for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
    println(match.value)
}

# 결과
+1234
-FFFF
-a
be
```

<br/>

### apply

`apply` 는 컨텍스트 객체를 `this` 를 통해 사용하며, 컨텍스트 객체를 반환한다. `apply` 는 코드 블럭이 값을 반환하지 않고 주로 컨텍스트 객체의 멤버들로 동작하는 경우 사용한다.

```kotlin
val adam = Person("Adam").apply {
    age = 32
    city = "London"
}
println(adam)

# 결과
Person(name=Adam, age=32, city=London)
```

<br/>

### also

`also` 는 context 객체를 `argument(it)` 처럼 사용할 수 있고, context 객체를 반환한다. `also` 는 context 객체를 인자처럼 사용하는 액션에 유용하다. 또 객체의 프로퍼티나 함수에 대한 참조가 아니라 객체 자체에 대한 참조가 필요한 액션에 유용하다.

```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("The list elements before adding new one: $it") }
    .add("four")

# 결과
The list elements before adding new one: [one, two, three]
```

<br/>

<p align="center">

<img src="https://github.com/KCSGround/TIL/blob/master/assets/scope-functions.PNG" width="800px" height="310px"/>

</p>

<br/>
