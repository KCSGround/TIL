# Fragment ViewPager2

## 개요

프래그먼트는 액티비티내의 사용자 인터페이스 모듈화의 일부분을 나타낸다. 프래그먼트에는 자체 수명 주기가 있고 자체 입력 이벤트를 수신하며 포함하는 액티비티가 실행되는 동안 프래그먼트를 추가하거나 제거할 수 있다.

화면 슬라이드는 하나의 전체 화면에서 다른 전체 화면으로 전환하는 것으로, 설정 마법사 또는 슬라이드쇼와 같은 UI에서 일반적으로 사용된다. 이 과정에서는 지원 라이브러리에서 제공하는 `ViewPager2`로 화면 슬라이드를 실행하는 방법을 보여준다. `ViewPager2` 객체는 화면 슬라이드에 자동으로 애니메이션을 적용할 수 있다.

## Fragment 를 활용하여 ViewPager2 만들기

### 뷰 만들기

```xml
   <!-- fragment_screen_slide_page.xml -->
    <ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/content"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <TextView style="?android:textAppearanceMedium"
            android:padding="16dp"
            android:lineSpacingMultiplier="1.2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/lorem_ipsum" />
    </ScrollView>
```

나중에 프래그먼트 콘텐츠에 사용할 레이아웃 파일을 만든다. 프래그먼트 콘텐츠에 대한 문자열도 정의해야 한다. 다음 예제에는 텍스트를 표시하는 텍스트 뷰가 포함되어 있다.

해당 xml 파일은 `ViewPager2` 를 통해서 각 페이지 마다 들어가게 되는 Component 를 만드는것과 같다. 위의 예제는 `ScrollView` 안에 `TextView` 가 있는 것으로 스크롤 가능한 `TextView` 가 `ViewPager2` 의 한 페이지에 들어가는 것과 같다.

<br/>

### 프래그먼트 만들기

`onCreateView()` 메서드에서 방금 만든 레이아웃을 반환하는 `Fragment` 클래스를 만든다. 그러면 사용자에게 표시할 새 페이지가 필요할 때마다 상위 활동에서 이 프래그먼트의 인스턴스를 만들 수 있다.

```kotlin
   import android.support.v4.app.Fragment

    class ScreenSlidePageFragment : Fragment() {

        override fun onCreateView(
                inflater: LayoutInflater,
                container: ViewGroup?,
                savedInstanceState: Bundle?
        ): View = inflater.inflate(R.layout.fragment_screen_slide_page, container, false)
    }
```

onCreateView 는 프래그먼트가 사용자 인터페이스 보기를 인스턴스화 하도록 하기 위해서 호출된다. 그래픽이 아닌 Fragment 를 사용한다면 null 값을 반환할 수 있다.

`inflater` 는 프래그먼트의 모든 뷰를 확장하는데 사용할 수 있는 `LayoutInflater` 객체이다.

`container` 는 `null` 이 아닌 경우 프래그먼트의 UI가 첨부되어야 하는 상위 뷰이다. 프래그먼트는 뷰 자체를 추가해서는 안되지만 뷰의 `LayoutParams` 를 생성하는 데 사용할 수 있다.

`savedInstanceState` 는 `Activity.onSaveInstanceState(Bundle)`와 마찬가지로 번들에 배치하는 데이터는 구성 변경과 프로세스 중단 및 재생성을 통해 유지된다.

`onCreate(Bundle)`와 `onViewCreated(View, Bundle)` 사이에서 호출된다.

<br/>

### ViewPager2 추가

`ViewPager2` 객체에는 **페이지 간 전환을 위한 스와이프 동작이 내장되어 있으며** 기본적으로 화면 슬라이드 애니메이션을 표시하므로 직접 애니메이션을 만들 필요가 없다. `ViewPager2` 는 표시할 새 페이지의 요소로 `FragmentStateAdapter` 객체를 사용하므로 `FragmentStateAdapter` 는 개발자가 이전에 만든 프래그먼트 클래스를 사용한다.

```xml
    <!-- activity_screen_slide.xml -->
    <androidx.viewpager2.widget.ViewPager2
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
```

다음 ViewPager 를 보여주고자 하는 곳에 ViewPager2 에 대한 코드를 써야한다.

<br/>

### 액티비티 작성

```kotlin
    import androidx.fragment.app.Fragment
    import androidx.fragment.app.FragmentActivity
    ...
    /**
     * The number of pages (wizard steps) to show in this demo.
     */
    private const val NUM_PAGES = 5

    class ScreenSlidePagerActivity : FragmentActivity() {
        private lateinit var viewPager: ViewPager2

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            // 뷰페이저가 있는 xml 파일 <1번과정>
            setContentView(R.layout.activity_screen_slide)

            // ViewPager2 가 선언 된 것에 바인딩
            viewPager = findViewById(R.id.pager)

            // ViewPager2 에 들어가는 데이터를  관리하는 어댑터 <3번과정>
            val pagerAdapter = ScreenSlidePagerAdapter(this)
            viewPager.adapter = pagerAdapter
        }

        override fun onBackPressed() {
            if (viewPager.currentItem == 0) {
                super.onBackPressed()
            } else {
                viewPager.currentItem = viewPager.currentItem - 1
            }
        }

        // ViewPager2 의 데이터를 처리하는 어댑터, getItemCount(), createFragment() <2번과정>
        private inner class ScreenSlidePagerAdapter(fa: FragmentActivity) : FragmentStateAdapter(fa) {
            override fun getItemCount(): Int = NUM_PAGES

            override fun createFragment(position: Int): Fragment = ScreenSlidePageFragment()
        }
    }
```

1. 콘텐츠 뷰를 `ViewPager2`가 있는 레이아웃으로 설정한다.

2. `FragmentStateAdapter` 추상 클래스를 확장하는 클래스를 만들고 `createFragment()` 메서드를 구현하여 `ScreenSlidePageFragment` 의 인스턴스를 새 페이지로 제공한다. 페이저 어댑터는 어댑터에서 만들 페이지 수(예에서는 5개)를 반환하는 `getItemCount()` 메서드도 구현해야 한다.

3. `FragmentStateAdapter` 를 `ViewPager2` 객체에 연결한다.

<br/>

GitHub [ViewPager2 샘플](https://github.com/android/views-widgets-samples/tree/master/ViewPager2)

---

참고문서 - [안드로이드 공식 개발자 페이지](https://developer.android.com/?hl=ko)
