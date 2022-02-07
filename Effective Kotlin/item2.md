# 아이템 2: 변수의 스코프를 최소화하라

 - 상태를 정의할 때는 변수와 프로퍼티의 스코프를 최소화하는 것이 좋음

> - 프로퍼티보다는 지역 변수를 사용하는 것이 좋음
> - 최대한 좁은 스코프를 갖게 변수를 사용함
>   - ex) 반복문 내부에서만 변수가 사용되면 변수를 반복문 내부에 작성

- 스코프를 좁게 만드는 것이 좋은 여러가지 이유 중에 가장 좋은 이유는 **프로그램을 추적하고 관리하기 쉽기 때문**

- 변수는 읽기 전용 또는 읽고 쓰기 전용 여부와 별개로 변수를 정의할 때 초기화되는 것이 좋음

```kotlin
// 나쁜 예
val user: User
if (hasValue) {
    user = getValue()
} else {
    user = User()
}

// 조금 더 좋은 예
val user: User = if (hasValue) {
    getValue()
} else {
    User()
}
```

- 여러 프로퍼티를 한꺼번에 설정해야 하는 경우, 구조분해 선언(destructuring declaration)을 활용하는 것이 좋음

```kotlin
// 나쁜 예
fun updateWeather(degrees: Int) {
    val description: String
    val color: Int
    if (degrees < 5) {
        description = "cold"
        color = Color.BLUE
    } else if (degrees < 23) {
        description = "mild"
        color = Color.YELLOW
    } else {
        description = "hot"
        color = Color.RED
    }
    // ...
} 

// 조금 더 좋은 예
fun updateWeather(degrees: Int) {
    val (description, color) = when {
        degrees < 5 -> "cold" to Color.BLUE
        degrees < 25 -> "mild" to Color.YELLOW
        else -> "hot" to Color.RED
    }
    // ...
}
```

## 캡처링

- 시퀀스를 활용해 에라토스테네스의 체를 구현하면 아래와 같이 구현할 수 있음

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    while (true) {
        val prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}

println(primes.take(10).toList())
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

- 위의 코드는 아래와 같이 최적화할 수 있다고 생각하기 쉽지만 실행 결과가 이상하게 됨

```kotlin
val primes: Sequence<Int> = sequence {
    var numbers = generateSequence(2) { it + 1 }

    var prime: Int
    while (true) {
        prime = numbers.first()
        yield(prime)
        numbers = numbers.drop(1).filter { it % prime != 0 }
    }
}

println(primes.take(10).toList())
// [2, 3, 5, 6, 7, 8, 9, 10, 11, 12]
```

- 시퀀스를 사용하므로 필터링이 지연되었고, 최종적인 `prime`값으로만 필터링되었기 때문에 결과가 이상하게 나옴