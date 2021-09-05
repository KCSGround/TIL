# Retrofit

## retrofit 이란?

[retrofit](https://en.wikipedia.org/wiki/Retrofitting) 은 다음과 같은 뜻을 가진다.

- 기존에 사용할 수 없었던, 필요하다고 갖누되는 새 부품이나 개조된 장비를 갖추다.

- 이전에 제조되거나 건설된 것에 설치하다.

- 새로운 목적이나 필요에 순응하다.

retrofit은 `개조`라는 단어로 번역 가능하다. 원래 없던 부품을 새로 장작/제공한다는 의미를 가진다.

Android 에서 retrofit 은 애플리케이션 통신 기능에 사용하는 코드를 사용하기 쉽게 만들어 놓은 라이브러리이다.

REST 기반의 웹 서비스를 통해 JSON 구조의 데이터를 쉽게 가져오고 업로드할 수 있다.

<br/>

\*\* [REST API 란?](https://www.redhat.com/ko/topics/api/what-is-a-rest-api)

<br/>

Retrofit 은 Square 에서 개발했으며 공식 페이지는 다음과 같다.

https://square.github.io/retrofit/

square 는 다음과 같이 retrofit 을 소개한다.

> Retrofit은 Java 및 Android 용 REST 클라이언트이다. REST 기반 웹 서비스를 통해 JSON (또는 기타 구조화 된 데이터)을 검색하고 업로드하는 것이 비교적 쉽다. Retrofit에서는 데이터 직렬화에 사용되는 변환기를 구성한다. 일반적으로 JSON의 경우 GSon을 사용하지만 XML 또는 기타 프로토콜을 처리하기 위해 사용자 지정 변환기를 추가 할 수 있다. Retrofit은 HTTP 요청에 OkHttp 라이브러리를 사용한다.

retrofit 은 안드로이드 앱에서 필요한 데이터를 서버로부터 가져오고 서버에 데이터를 전송하기 위한 코드를 작성할 때 사용하는 라이브러리이다.

앱-서버 통신을 Okhttp 라이브러리로 AsyncTask를 사용하여 구현했다고 할 수도 있다. 하지만 AsyncTask 로 서버와 통신을 구현하는 것은 어렵고 시간이 많이 들 뿐만 아니라(비동기 처리 코드를 개발자가 하나하나 작성), AsyncTask 자체가 안드로이드에서 분리 되었다. 그래서 이제 AsyncTask 를 사용하여 서버 통신을 구현하는 것은 좋은 생각이 아니다.

<br/>

## retrofit 사용하기

Retrofit 을 사용하기 위해서 다음 세 가지 클래스가 필요하다.

1. JSON 형태의 모델 클래스 (kotlin 에서는 data class 를 사용)

2. HTTP 작업을 정의하는(onSuccess/onFail) 인터페이스

3. Retrofit.Builder를 선언한 클래스 (baseUrl과 Converter등을 선언한다. Interceptor를 추가하여 응답을 가공할수도 있다.)

<br/>

- Retrofit 라이브러리를 Gradle 에 추가한다.

```gradle
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

Gson(구글 Gson, Google Gson)은 JSON의 자바 오브젝트의 직렬화, 역직렬화를 해주는 오픈 소스 자바 라이브러리이다.

<br/>

- Manifest 파일에 INTERNET permossion 을 추가한다.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

<br/>

- JSON 형태의 모델 클래스를 작성한다.

**HouseModel.kt**

```kotlin
data class HouseModel(
    val id: Int,            // 숙소에 대한 ID
    val title: String,      // 숙소 홍보 제목
    val price: String,      // 숙소 가격
    val imgUrl: String,     // 숙소 이미지
    val lat: Double,        // 위도
    val lng: Double         // 경도
)
```

**HouseDto.kt**

```kotlin
/**
 * 숙소에 대한 정보를 가지는 데이터 객체를 담는 List
 */
data class HouseDto(
    val items: List<HouseModel>
)
```

**서버에 저장된 JSON 값**

```json
{
  "items": [
    {
      "id": 1,
      "title": "강원대!! 최저가!! 레지던스!!",
      "price": "23,000원",
      "lat": 37.869117090350585,
      "lng": 127.74417839647492,
      "imgUrl": "https://picsum.photos/200/200"
    },
    {
      "id": 2,
      "title": "강원대!! 이게 맞나?",
      "price": "33,000원",
      "lat": 37.869276,
      "lng": 127.735477,
      "imgUrl": "https://picsum.photos/200/200"
    },
    {
      "id": 3,
      "title": "강원대!! 최저가!! 공쪽입니다.",
      "price": "43,000원",
      "lat": 37.869572,
      "lng": 127.736646,
      "imgUrl": "https://picsum.photos/200/200"
    }
  ]
}
```

서버 통신시 request body 또는 response body 에서 사용할 JSON 형태의 모델 클래스를 작성하는 것이다.

Java 에서는 모델 클래스를 작성하기 위해 따로 getter, setter 등 여러 추가적인 코드를 작성해야 하지만, Kotlin 의 경우 `data class` 형태로 프로퍼티의 이름과 타입을 지정하기만 하면 된다.

`data class` 로 모델 클래스를 작성할 경우, 프로퍼티의 변수명은 실제 서버에서 사용하는 값과 똑같이 작성해야 한다. `HouseModel.kt` 파일과 같이 서버에 저장된 JSON 값들과 같이 작성했음.

앱내에서는 변수명을 다르게 사용하고 싶은 경우 `@SerializedName` 어노테이션을 사용하면 된다.

<br/>

- RetrofitService Interface 를 작성한다.

```kotlin
import retrofit2.Call
import retrofit2.http.GET

/**
 * retrofit2 를 사용하기 위한 인터페이스
 */
interface HouseService {
    /**
     * HTTP GET 을 통해서 해당 주소에 있는 JSON 데이터를 가져오는 함수
     * getHouseList 는 GET 처리를 통해서 HouseDto 를 Call 해서 가져온다.
     * HouseDto 는 val items: List<HouseModel> 값을 가진다.
     */
    @GET("/v3/e88aa115-d2cf-45ea-80a3-feffdcded4cb")
    fun getHouseList(): Call<HouseDto>

}
```

<br/>

- Retrofit.Builder 로 Retrofit 객체를 초기화하고 Retrofit 객체에 enqueue() 를 사용하여 서버와 통신했을때 콜백을 작성하여 사용한다.

```kotlin
// Retrofit Buiolder 를 이용해서 Retrofit 객체 초기화
val retrofit = Retrofit.Builder()
    .baseUrl("https://run.mocky.io/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

// Retrofit interface 를 사용한다.
retrofit.create(HouseService::class.java).also {
    // enqueue() 를 통해서 서버와 통신했을 때 콜백을 작성한다.
    it.getHouseList().enqueue(object :Callback<HouseDto> {
        override fun onResponse(call: Call<HouseDto>, response: Response<HouseDto>) {
            if (response.isSuccessful.not()) {
                return
            }
            response.body()?.let { dto ->
                updateMarker(dto.items)
                viewPagerAdapter.submitList(dto.items)
                recyclerAdapter.submitList(dto.items)
                bottomSheetTitleTextView.text = "${dto.items.size} 개의 숙소"
            }
        }
    override fun onFailure(call: Call<HouseDto>, t: Throwable) {}
    })
}
```

`object : Callback<HouseDto>` 콜백 오브젝트를 넣어서 서버와 통신했을때, `onResponse(응답한 경우)`, `onFailure(실패한 경우)` 를 각각 작성하여 처리한다.

---

참고 사이트

- https://salix97.tistory.com/204
- https://medium.com/mindorks/understand-how-does-retrofit-work-c9e264131f4a
- https://flymogi.tistory.com/entry/Retrofit%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90-v202
- https://www.vogella.com/tutorials/Retrofit/article.html

<br/>

- [retrofit 공식사이트](https://square.github.io/retrofit/)
