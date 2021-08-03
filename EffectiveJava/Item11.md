# equals를 재정의하려거든 hashCode도 재정의하라

- **`equals` 를 재정의한 클래스 모두에서 `hashCode` 도 재정의해야 함**
  - 그렇지 않으면 `hashCode` 일반 규약을 어기게 되어 `HashMap` 이나 `HashSet` 같은 컬렉션의 원소로 사용할 때 문제가 생김

> - equals 비교에 사용되는 정보가 변경되지 않았다면, 어플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야함. 단, 어플리케이션을 다시 실행한다면 이 값은 달라져도 상관없음.
> - equals(Object)가 두 객체를 같다고 판단햇다면, 두 객체의 hashCode는 똑같은 값을 반환해야함
> - equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없음. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아짐.

- **논리적으로 같은 객체는 같은 해시코드를 반환해야함**

  - ```java
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
    ```

    - 위 코드를 실행시키면 "제니"가 출력될 것 같지만 실제로는 `null` 을 반환함
    - 두 `PhoneNumber` 객체가 서로 다른 해시코드를 반환했기 때문

## hashCode 구현방법

### 피해야하는 구현 방법

```java
@Override
public int hashCode() {
    return 42;
}
```

- 이 코드는 모든 객체에서 똑같은 해시 코드를 반환해주므로, 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결리스트(linked list) 처럼 동작함
  - 평균 수행 시간이 O(1)인 해시테이블이 O(n)이 됨
- **좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환함**
  - 이상적인 해시 함수는 서로 다른 인스턴스들을 32비트 정수 범위에 균일하게 분배해야함

### hashCode를 작성하는 요령

1. `int` 변수 `result` 를 선언한 후 값 `c` 로 초기화한다. 이때 `c` 는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드다(여기서 핵심 필드란 `equals` 비교에 사용되는 필드를 의미. 아이템 10 참조)
2. 해당 객체의 나머지 핵심 필드 `f` 각각에 대해 다음 작업을 수행한다.
   1. 해당 필드의 해시코드 `c` 를 계산한다
      1. 기본 타입 필드라면, `Type.hashCode(f)` 를 수행한다. 여기서 `Type` 은 해당 기본 타입의 박싱 클래스
      2. 참조 타입 필드면서 이 클래스의 `equals` 메서드가 이 필드의 `equals` 를 재귀적으로 호출해 비교한다면, 이 필드의 `hashCode` 를 재귀적으로 호출한다. 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(canonical representation)을 만들어 그 표준형의 `hashCode` 를 호출한다. 필드의 값이 `null` 이면 0을 사용한다(다른 상수도 괜찮지만 전통적으로 0을 사용함)
      3. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 닥룬다. 이상의 규칙을 재귀적으로 적용해 각 핵심원소의 해시코드를 계산한 다음, 단계 2.2방식으로 갱신한다. 배열에 핵심 원소가 하나도 없다면 단순히 상수(0을 추천)를 사용한다. 모든 원소가 핵심 원소라면 `Arrays.hashCode` 를 사용한다.
   2. 단계 2.1에서 계산한 해시코드 `c` 로 `result` 를 갱신한다. `result = 31 * result + c;`
3. `result` 를 반환한다.



- 파생 필드는 해시코드 계산해서 제외해도 됨
  - 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 됨
- `equals` 비교에 사용되지 않는 필드는 반드시 제외해야함

- `31 * result` 는 필드를 곱하는 순서에 따라 `result` 값을 달라지게 함
  - 클래스에 비슷한 필드가 여러 개일 때 해시 효과를 크게 높여줌
  - 31은 홀수이면서 소수
    - 이 숫자가 짝수이고 오버플로가 발생한다면 정보를 잃게됨
    - `(result << 5) - result` 와 같으므로 시프트 연산과 뺄셈으로 최적화 가능

#### 적용

- ```java
  @Override
  public int hashCode() {
      int result = Short.hashCode(areaCode);
      result = 31 * result + Short.hashCode(prefix);
      result = 31 * result + Short.hashCode(lineNum);
      return result;
  }
  ```

  - `PhoneNumber` 클래스의 핵심 필드 3개만을 사용해 해시코드를 계산
    - 동치인 인스턴스들은 같은 해시코드를 갖게됨

### hash

- `Objects` 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 `hash` 를 제공함

  - 이 메서드를 활용하면 위에 소개한 방법과 비슷한 수준의 `hashCode` 함수를 단 한 줄로 작성할 수 있음

  - 위의 방식보다 속도는 더 느림

    - 입력 인수를 담기 위한 배열의 생성, 입력 중 기본 타입이 있다면 방식과 언방식을 거치기 때문
    - 성능에 민감하지 않는 상황에만 사용할 것

  - ```java
    @Override
    public int hashCode() {
        return Objects.hash(lineNUm, prefix, areaCode);
    }
    ```

  - 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 매번 새로 계산하기보다는 캐싱을 해야함

    - 해시의 키로 사용되지 않는 경우라면 지연 초기화 방법도 있음
      - 필드를 지연 초기화하려면 그 클래스를 스레드 안전하게 만들어야함 (아이템 83)

#### Thread Safe

```java
private int hashCode; // 자동으로 0으로 초기화된다.

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

- **성능을 높이기 위해 해시코드를 계산할 때 핵심 필드를 생략해서는 안됨**
  - 속도는 빨라지지만, 해시 품질이 나빠져 해시테이블의 성능을 떨어뜨릴 수 있음

- **`hashCode` 가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말 것**
  - **클라이언트가 이 값에 의지하지 않고, 추후에 계산 방식을 바꿀 수 있도록 해야함**

## 정리

> equals를 재정의할 때는 hashCode도 반드시 재정의해아함. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것임
>
> 재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야함
>
> AutoValue 프레임워크 혹은 IDE를 활용하면 equals와 hashCode를 자동으로 만들어줌



