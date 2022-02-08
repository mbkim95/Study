# 아이템 8: 적절하게 null을 처리하라

- `null`은 값이 부족하다(lack of value)를 의미함
- 함수가 `null`을 리턴한다는 것은 여러 의미를 가질 수 있음
  - `String.toIntOrNull()`은 `Int`로 변환할 수 없을 때 `null`을 반환
  - `Iterable<T>.firstOrNull(() -> Boolean)`은 주어진 조건에 맞는 요소가 없을 때 `null`을 반환

- API 사용자가 nullable 값을 처리해야하기 때문에, 최대한 명확한 의미를 담는 것이 좋음

```kotlin
val printer: Printer? = getPrinter()
printer.print() // 컴파일 오류

printer?.print() // 안전 호출
if (printer != null) printer.print() // 스마트 캐스팅
printer!!.print() // not-null assertion
```

- 기본적으로 nullable 타입은 세 가지 방법으로 처리함
  - ?., 스마트 캐스팅, Elvis 연산자 등을 활용
  - 오류를 `throw`
  - 함수 또는 프로퍼티를 리팩터링해서 nullable 타입이 나오지 않게 바꿈

## null을 안전하게 처리하기

- `null`을 안전하게 처리하는 방법 중 널리 사용되는 방법으로는 안전 호출(safe call)과 스마트 캐스팅(smart casting)이 있음

```kotlin
printer?.print() // safe call
if (printer != null) printer.print() // smart casting
```

- Elvis 연산자를 사용하는 방법도 있음

```kotlin
val printerName1 = printer?.name ?: "Unnamed"
val printerName2 = printer?.name ?: return
val printerName3 = printer?.name ?: throw Error("Printer must be named")
```

- 스마트 캐스팅은 코틀린의 규약 기능(contracts feature)을 지원하므로 이 기능을 사용하면 스마트 캐스팅을 할 수 있음

```kotlin
println("What is your name?")
val name = readLine()
if (!name.isNullOrBlank()) {
    println("Hello ${name.toUpperCase()}")
}

val news: List<News>? = getNews()
if (!news.isNullOrEmpty()) {
    news.forEach { notifyUser(it) }
}
```

## 오류 throw 하기

- 다른 개발자가 어떤 코드를 보고 선입견처럼 '당연히 그럴 것이다'라고 생각하게 되는 부분이 있고, 그 부분에서 문제가 발생할 경우에는 개발자에게 오류를 강제로 발생시켜주는 것이 좋음
- 오류를 강제로 발생시킬 때는 `throw`, `!!`, `requireNotNull`, `checkNotNull` 등을 활용함

```kotlin
fun process(user: User) {
    requireNotNull(user.name)
    val context = checkNotNull(context)
    val networkService = getNetworkService(context) ?: throw NoInternetConnection()
    networkService.getData { data, userData ->
        show(data!!, userData!!)
    }
}
```

## not-null assertion(!!)과 관련된 문제

- nullable을 처리하는 가장 간단한 방법은 not-null assertion(!!)을 사용하는 방법인데, 이를 사용하면 자바에서 nullable을 처리할 때 발생할 수 있는 문제가 똑같이 발생함
- nullability와 관련된 정보는 숨겨져 있으므로 놓치기 쉬움

```kotlin
fun largestOf(vararg nums: Int): Int = nums.max()!!

largestOf() // NPE
```

- 변수를 `null`로 설정하고, 이후에 `!!` 연산자를 사용하는 방법은 이후에 프로퍼티를 계속해서 언팩(unpack)해야 하고, 해당 프로퍼티가 이후에 의미 있는 `null`값을 가질 가능성을 차단하므로 좋지 않음
  - `lateinit` 또는 `Delegates.notNull`을 사용하는 것이 좋음
- 일반적으로 **`!!` 연산자 사용을 피해야함**

## 의미 없는 nullability 피하기

- nullability는 어떻게든 적절히 처리해야 하므로, 필요한 경우가 아니면 nullability 자체를 피하는 것이 좋음

> nullability를 피하는 방법
>
> - 클래스에서 nullability에 따라 여러 함수를 만들어서 제공하는 방법
>   - ex) `List<T>`의 `get`과 `getOrNull` 함수
> - 어떤 값이 클래스 생성 이후에 확실하게 설정되었다는 보장이 있다면 `lateinit` 프로퍼티와 notNull 델리게이트를 사용하는 방법
> - nullable enum과 None enum 값은 완전히 다른 의미
>   - `null` enum은 별도로 처리해야 하지만, None enum은 정의에 없으므로 필요한 경우에 사용하는 쪽에서 추가해서 활용할 수 있음

## lateinit 프로퍼티와 notNull 델리게이트

- `lateinit` 한정자는 프로퍼티가 이후에 설정될 것임을 명시하는 한정자
  - 초기화 전에 값을 사용하려고 하면 예외가 발생함
  - 프로퍼티를 처음 사용하기 전에 반드시 초기화될 것이라고 예상되는 상황에 활용

```kotlin
class UserControllerTest {
    private lateinit var dao: UserDao
    private lateinit var controller: UserController

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao)
    }

    @Test
    fun test() {
        controller.doSomething()
    }
}
```

- JVM에서 `Int`, `Long`, `Double`, `Boolean`과 같은 기본 타입은 `Delegates.notNull`을 사용해서 초기화해야함

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean by Delegates.notNull()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extra.getBoolean(FROM_NOTIFICATION_ARG)
    }
}
