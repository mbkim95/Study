# 아이템 5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스가 하나 이상의 자원에 의존함
  - ex) 맞춤법 검사기는 사전(dictionary)에 의존함
  - 이런 클래스를 정적 유틸리티 클래스로 구현하는 모습을 드물지 않게 볼 수 있음



- 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    
    private SpellChecker() {} // 객체 생성 방지
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

- 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;
    
    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

- 두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 좋은 방식이 아님
  - 실전에서는 사전이 언어별로 따로 있고 특수 어휘용 사전을 별도로 두기도 함
  - 테스트용 사전이 필요할 수도 있음



## 개선

- `SpellChecker` 가 여러 사전을 사용할 수 있도록 만들어보자

### 간단한 방법

- 필드에서 final 한정자를 제거하고 다른 사전으로 교체하는 메서드를 추가할 수 있음
  - 이 방식은 어색하고 오류를 내기 쉬우며 멀티스레드 환경에서는 쓸 수 없음

> 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않음

- 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원(dictionary)을 사용해야함
  - **인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식**을 사용하면 됨
    - 의존 객체 주입의 한 형태

```java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

- 예에서는 딱 하나의 자원만 사용하지만, 자원이 몇 개든 의존 관계가 어떻든 상관없이 잘 동작함
- 불변을 보장하여 (같은 자원을 사용하려는) 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있음
- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있음

### 변형

- 생성자에 자원 팩터리를 넘겨주는 방식으로 변형할 수 있음 (팩터리 메서드 패턴을 구현)
  - 팩터리: 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
  - ex) Java 8의 `Supplier<T>`
    - `Supplier<T>` 를 입력으로 받는 메서드는 일반적으로 한정적 와일드카드 타입(bounded wildcard type)을 사용해 팩터리 타입 매개변수를 제한해야함
      - 이 방식으로 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 넘길 수 있음
    - ex) `Mosaic create(Supplier<? extends Tile> tileFactory) { ... }`
      - 클라이언트가 제공한 팩터리가 생성한 타일(Tile)들로 구성된 모자이크(Mosaic)를 만드는 메서드

## 의존 객체 주입의 단점

- 의존 객체 주입이 유연성과 테스트 용이성을 개선해주지만, 의존성이 수 천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들 수 있음
  - 대거(Dagger), 주스(Guice), 스프링(Spring)과 같은 의존 객체 주입 프레임워크를 사용해  어질러짐을 해소할 수 있음

## 정리

> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 도작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋음.
>
> 이 자원들을 클래스가 직접 만들게 해서도 안됨.
>
> 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리나 빌더에) 넘겨주자.
>
> 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 기막히게 개선해줌.
