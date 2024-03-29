# 아이템 6: 불필요한 객체 생성을 피하라

- 똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많음
  - 재사용은 빠르고 세련됨
  - 특히 불변 객체는 언제든 재사용할 수 있음

## 극단적인 예시

```java
String s = new String("bikini"); // 따라 하지 말 것!
```

- 실행될 때마다 `String` 인스턴스를 새로 만듬
- 반복문이나 빈번히 호출되는 메서드 안에 있다면 쓸데없는 `String` 인스턴스가 수백만 개 만들어질 수 있음

### 개선

```java
String s = "bikini";
```

- 새로운 인스턴스를 매번 만드는 대신 하나의 `String` 인스턴스를 사용함
- 같은 가상 머신 안에서 이와 똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 재사용함이 보장됨

### 정적 팩터리 메서드

- 생성자 대신 정적 팩터리 메서드(아이템 1)를 제공하는 불변 클래스에서는 정적 팩터리 메서드를 사용해 불필요한 객체 생성을 피할 수 있음
  - ex) `Boolean(String)` 생성자 대신 `Boolean.valueOf(String)` 팩터리 메서드를 사용하는 것이 좋음
  - 생성자는 호출될 때마다 새로운 객체를 만들지만, 팩터리 메서드는 그렇지 않음
  - 불변 객체만이 아니라 가변 객체라 해도 사용 중에 변경되지 않을 것임을 안다면 재사용 가능

### 객체 생성 비용

- 생성 비용이 아주 비싼 객체가 반복해서 필요하다면 캐싱해서 재사용하는 것이 좋음
  - 자신이 만드는 객체가 비싼 객체인지를 매번 명확히 알기는 어려움

```java
static boolean isRomanNumeral(String s) {
        return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    }
```

- `String.matches` 는 정규표현식으로 문자열 형태를 확인하는 가장 쉬운 방법이지만, 성능이 중요한 상황에서 반복해 사용하기엔 적합하지 않음
  - 메서드 내부에서 만드는 정규표현식용 `Pattern` 인스턴스는 한번 쓰고 버려져서 곧바로 GC의 대상이 됨
  - `Pattern` 은 입력받은 정규표현식에 해당하는 유한 상태 머신(finite state machine)을 만드므로 인스턴스 생성 비용이 높음

####  개선

- 필요한 정규표현식을 표현하는 (불변인) `Pattern` 인스턴스를 클래스 초기화(정적 초기화) 과정에서 직접 생성해 캐싱해두고, 나중에 `isRomanNumeral` 메서드가 호출될 때마다 이 인스턴스를 재사용하는 식으로 개선 가능

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

- 길이가 8인 문자열 기준으로 1.1μs에서 0.17μs로 약 6.5배 빨라짐
- `Pattern` 인스턴스를 `static final` 필드로 끄집어내고 이름을 지어줌으로써 코드의 의미가 명확해짐
- `isRomanNumeral` 메서드를 한 번도 호출하지 않는다면 `ROMAN` 필드는 쓸데없이 초기화 된 것이므로 메서드가 처음 호출될 때 필드를 초기화하는 지연 초기화(lazy initialization, 아이템 83) 불필요한 초기화를 없앨 수 있음

## 단점

- 일반적으로 객체가 불변이라면 재사용해도 안전함이 명백하지만 훨씬 덜 명확하거나 직관에 반대되는 상황도 있음

### 예시

- 어댑터는 실제 작업은 뒷단 객체에 위임하고, 자신은 제 2의 인터페이스 역할을 해주는 객체
  - 어뎁터는 뒷단 객체만 관리하면 됨
  - 뒷단 객체 외에는 관리할 상태가 없으므로 뒷단 객체 하나당 어댑터 객체 하나씩만 만들어지면 충분함
  - ex) `Map` 인터페이스의 `keySet` 메서드는 `Map` 객체 안의 키 전부를 담은 `Set` 뷰를 반환
    - `keySet` 을 호출할 때마다 새로운 `Set` 인스턴스가 만들어질거라 생각할 수 있지만, 매번 같은 `Set` 을 반환할지도 모름
    - 반환된 인스턴스들은 모두가 똑같은 `Map` 인스턴스를 대변하므로, 반환한 객체 중 하나를 수정하면 다른 모든 객체가 따라서 바뀜
      - `keySet` 이 뷰 객체를 여러 개 만들어도 상관없지만 그럴 필요도 없고 이득도 없음

- 오토박싱은 기본 타입과 그에 대응하는 박싱된 기본 타입의 구분을 흐려주지만, 완전히 없애주는 것은 아님

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++)
        sum += i;
    return sum;
}
```

- `sum` 변수를 `long` 이 아닌 `Long` 으로 선언해서 불필요한 `Long` 인스턴스가 2^31개나 생성됨
- **박싱된 기본 타입보다는 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의하자**

## 정리

- 아이템 6을 **"객체 생성은 비싸니 피해야 한다"로 오해하면 안됨**
- 요즘의 JVM에서 별다른 일을 하지 않는 작은 객체를 생성하고 회수하는 일이 크게 부담되지는 않으므로, 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이라면 괜찮음
- 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 객체 풀(pool)을 만들지는 말자
  - 데이터베이스 연결 같은 경우 생성 비용이 워낙 비싸니 재사용하는 편이 낫지만, 일반적으로 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨림
- 방어적 복사(defensive copy)를 다루는 아이템 50과 내용이 대조적임
  - **방어적 복사가 필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실**을 기억할 것
  - 방어적 복사에 실패하면 언제 발생할지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 줌
