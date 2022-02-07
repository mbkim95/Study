# 아이템 3: 최대한 플랫폼 타입을 사용하지 말라

- 코틀린은 자바 등 다른 프로그래밍 언어에서 넘어온 타입을 특수하게 다루는데, 이러한 타입을 **플랫폼 타입(platform type)**이라고 부름
    - 플랫폼 타입은 `String!`처럼 타입 이름 뒤에 '!'를 붙여서 표기

```kotlin
// Java
public class UserRepo {
    public User getUser() {
        // ...
    }
}

// Kotlin
val repo = UserRepo()
val user1 = repo.user           // User!
val user2: User = repo.user     // User
val user3: User? = repo.user    // User?
```

- 플랫폼 타입을 사용할 때는 `null`이 아니라고 생각되는 것이 `null`일 수 있으므로 주의해야함
- 자바와 코틀린을 함께 사용할 때, 가능하면 `@Nullable`과 `@NotNull` 어노테이션을 붙여서 사용하기를 권장

- 플랫폼 타입은 안전하지 않으므로, 최대한 빨리 제거하는 것이 좋음

```kotlin
// Java
public class JavaClass {
    public String getValue() {
        return null;
    }
}

// Kotlin
fun statedType() {
    val value: String = JavaClass().value // NPE
    // ...
    println(value.length)
}

fun platformType() {
    val value = JavaClass().value
    // ...
    println(value.length) // NPE
}
```

- 플랫폼 타입으로 지정된 변수는 타입 검사기가 검출해줄 수 없으므로 nullable일 수 도 있음

```kotlin
interface UserRepo {
    fun getUserName() = JavaClass().value
}
```

- 메서드의 inferred 타입이 플랫폼 타입이므로 사용하는 사람에 따라 nullable이 아닐 거라고 받아들일 수 있는데, 이는 큰 문제가 될 수 있음
  - 안전한 코드를 원한다면 이런 부분을 제거하는 것이 좋음