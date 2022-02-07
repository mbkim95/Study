# 아이템 4: inferred 타입으로 리턴하지 말라

- 타입 추론을 사용할 때 몇 가지 위험한 부분들이 있음
- 할당 때 inferred 타입은 **정확하게 오른쪽에 있는 피연산자에 맞게 설정된다는 것**을 기억해야함
  - 절대로 슈퍼클래스 또는 인터페이스로 설정되지 않음

```kotlin
open class Aniaml
class Zebra: Animal()

fun main() {
    var animal = Zebra()
    var animal2: Animal = Zebra()
    animal = Animal() // 오류: Type mismatch
    animal2 = Animal()
}
```

- 리턴 타입은 API를 잘 모르는 사람에게 전달해 줄 수 있는 중요한 정보이므로 외부에서 확인할 수 있게 명시적으로 지정해 주는 것이 좋음

