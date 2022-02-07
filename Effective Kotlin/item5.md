# 아이템 5: 예외를 활용해 코드에 제한을 걸어라

- 확실하게 어떤 형태로 동작해야 하는 코드가 있다면, 예외를 활용해 제한을 걸어주는 것이 좋음

```kotlin
// Stack<T>의 일부
fun pop(num: Int = 1): List<T> {
    require(num <= size) {
        "Cannot remove more elements than current size"
    }
    check(isOpen) { "Cannot pop from closed stack" }
    val ret = collection.take(num)
    collection = collection.drop(num)
    assert(ret.size == num)
    return ret
}
```

- 제한을 걸어주면 다양한 장점이 있음

1. 문서를 읽지 않은 개발자도 문제를 확인할 수 있음
2. 문제가 있을 경우에 함수가 예외를 throw함.
3. 코드가 어느 정도 자체적으로 검사됨. 이와 관련된 단위 테스트를 줄일 수 있음
4. 스마트 캐스트 기능을 활용할 수 있게 되므로, 캐스트(타입 변환)를 적게 할 수 있음

## 아규먼트

- 함수를 정의할 때 타입 시스템을 활용해서 아규먼트(argument)에 제한을 거는 코드를 많이 사용함
  - 일반적으로 제한을 걸 때 `require` 함수를 사용함

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    return if (n <= 1) 1 else factorial(n - 1) * n
}

fun findClusters(points: List<Point>): List<Cluster> {
    require(points.isNotEmpty())
    // ...
}

fun sendEmail(user: User, message: String) {
    requireNotNull(user.email)
    require(isValidEmail(user.email))
    // ...
}
```

- 이러한 형태의 입력 유효성 검사 코드는 함수의 가장 앞부분에 배치되므로 읽는 사람도 쉽게 확인할 수 있음
- `require` 함수는 조건을 만족하지 못하면 `IllegalArgumentException`을 발생시킴
  - 문서에 관련된 내용을 반드시 명시해 두어야함
  - 람다를 활용해서 지연 메시지를 정의할 수도 있음

## 상태

 - 상태와 관련된 제한을 걸 때는 일반적으로 `check` 함수를 사용함

```kotlin
fun speak(text: String) {
    check(isInitialized)
    // ...
}

fun getUserInfo(): UserInfo {
    checkNotNull(token)
    // ...
}

fun next(): T {
    check(isOpen)
    // ...
}
```

- `check`함수는 지정된 예측을 만족하지 못할 때, `IllegalStateException`을 throw함
  - 예외 메시지는 지연 메시지를 사용해서 변경할 수 있음
- 함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 `require` 함수 뒤에 배치함

## Assert 계열 함수 사용

- 함수가 올바르게 구현되었다면, 확실하게 참을 낼 수 있는 코드들이 있음
- 그런데 함수가 올바르게 구현되어 있지 않을 수도 있으므로, 이러한 문제로 발생할 수 있는 추가적인 문제를 예방하려면 단위 테스트를 사용하는 것이 좋음

```kotlin
class StackTest {

    @Test
    fun 'stack pops correct number of elements'() {
        val stack = Stack(20) { it }
        val ret = stack.pop(10)
        assertEquals(10, ret.size)
    }
}
```

- 단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법이지만, 한 경우만 테스트해서는 모든 상황에서 괜찮은지 알 수 없음
- 모든 `pop` 호출 위치에서 제대로 동작하는지 확인할 수 있다면 구현의 정확성을 확인하기 더욱 더 좋음

```kotlin
fun pop(num: Int = 1): List<T> {
    // ...
    assert(ret.size == num)
    return ret
}
```

- 이러한 조건은 현재 코틀린/JVM에서만 활성화되고, `-ea` JVM옵션을 활성화해야 확인할 수 있음
- 테스트할 때만 활성화되므로, 오류가 발생해도 사용자가 알아차릴 수는 없음 (표준 애플리케이션 실행에서는 `assert`가 예외를 throw하지 않음)
  - 이 코드가 심각한 오류고, 심각한 결과를 초래할 수 있는 경우라면 `check`를 사용하는 것이 좋음

## nullability와 스마트 캐스팅

- 코틀린에서 `require`와 `check` 블록으로 `true`가 나왔다면 해당 조건은 이후로도 `true`일 거라고 가정함
  - 이를 활용해서 **타입 비교를 했다면, 스마트 캐스트가 작동**함

```kotlin
fun changeDress(person: Person) {
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
    // ...
}
```

- 이러한 특징은 어떤 대상이 `null`인지 확인할 때 굉장히 유용함

```kotlin
class Person(val email: String?)

fun sendEmail(person: Person, message: String) {
    require(person.email != null)
    val email: String = person.email
    // ...
}
```

- `requireNotNull`, `checkNotNull` 함수들도 스마트 캐스트를 지원하므로, 변수를 '언팩(unpack)'하는 용도로 활용할 수 있음

```kotlin
class Person(val email: String?)
fun validateemail(email: String) { /*...*/ }

fun sendEmail(person: Person, text: String) {
    val email = requireNotNull(person.email)
    validateEmail(email)
    // ...
}

fun sendEmail(person: Person, text: String) {
    requireNotNull(person.email)
    validateEmail(person.email)
    // ...
}
```

- `return`과 `throw`를 활용한 Elvis 연산자는 nullable을 확인할 때 굉장히 많이 사용됨
  - 이러한 코드는 함수의 앞부분에 넣어서 잘 보이게 만드는 것이 좋음

```kotlin
fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: return
    // ...
}

fun sendEmail(person: Person, text: String) {
    val email: String = person.email ?: run {
        log("Email not sent, no email address")
        return
    }
    // ...
}
```