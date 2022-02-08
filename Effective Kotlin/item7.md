# 아이템 7: 결과 부족이 발생할 경우 null과 Failure를 사용하라

- 함수가 원하는 결과를 만들어 낼 수 없을 때가 있음
  - 인터넷 문제로 서버로부터 데이터를 읽어 들이지 못한 경우
  - 조건에 맞는 요소가 없어서 찾지 못한 경우
  - 텍스트의 형식이 맞지 않아서 파싱하지 못한 경우
  
- 이러한 상황을 처리하는 방법은 크게 두 가지 있음
  - `null` 또는 '실패를 나타내는 `sealed` 클래스 (일반적으로 Failure라는 이름을 붙임)를 리턴
  - 예외를 `throw`

- 예외는 정보를 전달하는 방법으로 사용해서는 안됨
  - 예외는 예외적인 상황이 발생했을 때 사용하는 것이 좋음
  - 예외는 놓칠 수 있으며, 전체 애플리케이션을 중지시킬 수 있음

> 예외를 정보 전달의 방법으로 사용하면 안되는 이유
>
> - 많은 개발자는 예외가 전파되는 과정을 제대로 추적하지 못함
> - 코틀린의 모든 예외는 unchecked 예외이므로 사용자가 예외를 처리하지 않을 수도 있으며, 이와 관련된 내용은 문서에도 제대로 드러나지 않음. 실제로 API를 사용할 때 예외와 관련된 사항을 단순하게 메서드 등을 사용하면서 파악하기 어려움
> - 예외는 예외적인 상황을 처리하기 위해서 만들어졌으므로 명시적인 테스트(explicit test)만큼 빠르게 동작하지 않음
> try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한됨

- `null`과 Failure은 예상되는 오류를 표현하기에 좋음
  - 명시적으로 처리해야 하며, 애플리케이션의 흐름을 중지하지 않음
- **충분히 예측할 수 있는 범위의 오류는 `null`과 Failure을 사용하고, 예측하기 어려운 예외적인 범위의 오류는 예외를 `throw`해서 처리하는 것이 좋음**

```kotlin
inline fun <reified T> String.readObjectOrNull(): T? {
    // ...
    if (incorrectSign) {
        return null
    }
    // ...
    return result
}

inline fun <reified T> String.readObject(): Result<T> {
    // ...
    if (incorrectSign) {
        return Failure(JsonParsingException())
    }
    // ...
    return Success(result)
}

sealed class Result<out T>
class Success<out T>(val result: T): Result<T>()
class Failure(val throwable: Throwable): Result<Nothing>()

class JsonParsingException: Exception()
```

```kotlin
// null을 활용한 처리
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

```kotlin
// sealed 클래스를 활용한 처리
val person = userText.readObjectOrNull<Person>()
val age = when (person) {
    is Success -> person.age
    is Failure -> -1
}
```
