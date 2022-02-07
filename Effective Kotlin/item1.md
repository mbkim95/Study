# 아이템 1: 가변성을 제한하라

- 코틀린은 모듈로 프로그램을 설계함
  - 모듈은 클래스, 객체, 함수, 타입 별칭(type alias), 톱레벨(top-level) 프로퍼티 등의 요소로 구성
- 요소 중 일부는 상태(state)를 가질 수 있음
  - ex) 읽고 쓸 수 있는 프로퍼티(read-write property) var를 사용, mutable 객체 사용

## 코틀린에서 가변성 제한하기

- 코틀린은 가변성을 제한할 수 있도록 설계됨
  
> - 읽기 전용 프로퍼티(val)
> - 가변 컬렉션과 읽기 전용 컬렉션 구분하기
> - 데이터 클래스의 copy

### 읽기 전용 프로퍼티(val)

- 코틀린은 `val`을 사용해 읽기 전용 프로퍼티를 만들 수 있음
- 읽기 전용 프로퍼티가 **완전히 변경 불가능한 것은 아님**
  - 읽기 전용 프로퍼티가 mutable 객체를 담고 있다면, 내부적으로 변할 수 있음

```kotlin
val list = mutableListOf(1, 2, 3)
list.add(4)

println(list) // [1, 2, 3, 4]
```

- 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의할 수 있음
  - `var` 프로퍼티를 사용하는 `val` 프로퍼티는 **`var` 프로퍼티가 변할 때 변할 수 있음**
    - 값을 추출할 때마다 사용자 정의 게터가 호출되기 때문

```kotlin
var name: String = "Marcin"
var surname: String = "Moskala"
val fullName
    get() = "$name $surname"

fun main() {
    println(fullName) // Marcin Moskala
    name = "Maja"
    println(fullName) // Maja Moskala
}
```

> `val`은 읽기 전용 프로퍼티지만, **불변(immutable)을 의미하는 것은 아님**
> 완전히 변경할 필요가 없다면, `final` 프로퍼티를 사용하는 것이 좋음

### 가변 컬렉션과 읽기 전용 컬렉션 구분하기

- 코틀린은 읽고 쓸 수 있는 컬렉션과 읽기 전용 컬렉션으로 구분됨
  - 읽기 전용 컬렉션: Iterable, Collection, Set, List
  - 읽고 쓸 수 있는 컬렉션: MutableIterable, MutableCollection, MutableSet, MutableList

- 읽기 전용 컬렉션이 내부의 값을 변경할 수 있다는 의미는 아님
  - ex) `Iterable<T>.map`, `Iterable<T>.filter`는 변경할 수 있는 ArrayList를 리턴함

- 컬렉션을 진짜로 불변(immutable)하게 만들지 않고, **읽기 전용으로 설계한 것은 아주 중요함**
  - 내부적으로 인터페이스를 사용하므로 실제 컬렉션을 리턴할 수 있음
  - 코틀린이 내부적으로 immutable하지 않은 컬렉션을 외부적으로 immutable하게 보이게 만들어서 얻는 안정성

- 컬렉션 다운캐스팅은 안전하지 않고, 예측하지 못한 결과를 초래함

```kotlin
val list = listOf(1, 2, 3)

// 이렇게 하지 마세요!
if (list is MutableList) {
    list.add(4)
}
```

- 위 코드의 실행결과는 플랫폼에 따라 다른데, JVM에서 `listOf`는 자바의 `List` 인터페이스를 구현한 `Array.ArrayList` 인스턴스를 리턴함
- 자바의 `List`는 add, set 등의 메서드를 제공하므로 코틀린의 `MutableList`로 변경 가능
- 하지만 `Arrays.ArrayList`는 이러한 연산을 구현하지 않고 있으므로 `UnsupportedOperationExeption`이 발생함

- 코틀린에서 읽기 전용 컬렉션을 mutable 컬렉션으로 변경해야 한다면, 복제(copy)를 통해 새로운 mutable 컬렉션을 만드는 `list.toMutableList`를 활용해야함

### 데이터 클래스의 copy

- immutable 객체를 사용하면 다음과 같은 장점이 있음

1. 한 번 정의된 객체가 유지되므로, 코드를 이해하기 쉬움
2. 공유했을 때도 충돌이 따로 이루어지지 않으므로, 병렬 처리를 안전하게 할 수 있음
3. 객체에 대한 참조는 변경되지 않으므로, 쉽게 캐시할 수 있음
4. 방어적 복사본(defensive copy)을 만들 필요가 없음. 또한 객체를 복사할 때 깊은 복사를 따로 하지 않아도 됨
5. 다른 객체 (mutable 또는 immutable 객체)를 만들 때 활용하기 좋음. 또한 immutable 객체는 실행을 더 쉽게 예측할 수 있음
6. `set` 또는 `map`의 키로 사용할 수 있음. mutable 객체는 해시 테이블 내부에서 요소를 찾을 수 없게 되므로 사용할 수 없음

- immutable 객체는 변경할 수 없다는 단점이 있으므로 자신의 일부를 수정한 새로운 객체를 만들어내는 메서드를 가져야 함
  - data 한정자는 `copy` 메서드를 만들어줌

```kotlin
data class User(
    val name: String,
    val surname: String
)

var user = User("Maja", "Markiewicz")
user = user.copy(surname = "Moskala")
println(user) // User(name=Maja, surname=Moskala)
```

## 다른 종류의 변경 가능 지점

- 변경할 수 있는 리스트를 만들어야 한다고 가정하면 두 가지 선택지가 있음

1. mutable 컬렉션을 만드는 방법
2. `var`로 읽고 쓸 수 있는 프로퍼티를 만드는 방법

```kotlin
val list1: MutableList<Int> = mutableListOf()
var list2: List<Int> = listOf()

list1 += 1 // list1.plusAssign(1)로 변경됨
list2 += 1 // list2.plus(1)로 변경됨
```

- 두 방식의 변경 가능 지점(mutating point)가 서로 다름
- 첫 번째 코드는 구체적인 리스트 구현 내부에 변경 가능 지점이 있음
  - 멀티스레드 처리가 이뤄질 경우, 내부적으로 적절한 동기화가 되어 있는지 확실하게 알 수 없으므로 위험
- 두 번째 코드는 프로퍼티 자체가 변경 가능 지점
  - 멀티스레드 처리 안정성이 더 좋음

- mutable 프로퍼티를 사용하는 형태는 사용자 정의 세터 또는 이를 사용하는 델리게이트를 활용해 변경을 추적할 수 있음

```kotlin
var names by Delegates.observable(listOf<String>()) { _, old, new -> 
    println("Names changed from $old to $new")
}

names += "Fabio" // Names changed from [] to [Fabio]
names += "Bill" // Names changed from [Fabio] to [Fabio, Bill]

## 변경 가능 지점 노출하지 말기

- 상태를 나타내는 mutable 객체를 외부에 노출하는 것은 굉장히 위험함

```kotlin
data class User(val name: String)

class UserRepository {
    private val storedUsers: MutableMap<Int, String> = mutableMapOf()

    fun loadAll(): MutableMap<Int, String> {
        return storeUsers
    }

    // ...
}
```

- loadAll을 사용해서 private 상태인 UserR