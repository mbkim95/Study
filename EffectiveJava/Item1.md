# 아이템 1: 생성자 대신 정적 팩터리 메서드를 고려하라

- 클래스는 생성자와 별도로 정적 팩터리 메서드 (static factory method)를 제공할 수 있다.

```java
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;
}
```

## 장점

1. **이름을 가질 수 있다**

   - 생성자는 반환될 객체의 특성을 제대로 설명하지 못함

   - 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있음

     - ex) `BigInteger(int, int Random)` vs `BigInteger.probablePrime()`
       - 후자가 값이 소수인 `BigInteger` 를 반환한다는 의미를 더 잘 설명함

   - 한 클래스에 시그니처가 같은 생성자가 여러 개 필요하다면 정적 팩터리 메서드를 사용하자

     - 시그니처: 메서드명, 파라미터 순서, 파라미터 타입, 개수를 의미 (리턴 타입, 예외는 시그니처에서 제외)

       

2. **호출할 때마다 인스턴스를 새로 생성하지 않아도 된다**

   - 불변 클래스는 객체를 미리 만들어두거나 객체를 캐싱해서 재활용해서 불필요한 객체 생성을 피할 수 있음

     - ex) `Boolean.valueOf(boolean)` 의 경우 객체를 생성하지 않음

   - 어느 인스턴스를 살아있게 할지 통제할 수 있는 정적 팩터리 방식의 클래스를 인스턴스 통제(instance-controlled) 클래스라고 함.

     - 인스턴스 통제를 통해 클래스를 싱글턴으로 만들 수 있음

     - 인스턴스 통제를 통해 클래스를 인스턴스화 불가(noninstantiable)로 만들 수 있음

     - 불변 값 클래스에서 동치인 인스턴스가 단 하나 뿐임을 보장할 수 있음

       - ex) `a == b` 일 때만 `a.equals(b)` 가 성립

     - 인스턴스 통제는 플라이웨이트 패턴의 근간이 됨

     - 열거 타입은 인스턴스가 하나만 만들어짐을 보장함

       

3. **반환 타입의 하위 타입 객체를 반환할 수 있다**

   - 자바 컬렉션 프레임워크는 총 45개의 유틸리티 구현체를 인스턴스화 불가 클래스인 `java.util.Collections` 에서 정적 팩터리 메서드를 통해 얻도록 함.

     - 45개의 클래스를 공개하지 않기 때문에 API 외견을 작게 만들 수 있음
     - API를 사용하는 개발자가 익혀야 하는 개념의 수와 난이도도 낮아짐
       - 명시한 인터페이스대로 동작할 것을 알고 있으므로 굳이 별도 문서를 찾아가며 실제 구현 클래스를 볼 필요가 없음

   - 정적 팩터리 메서드를 사용하는 클라이언트는 메서드를 통해 생성한 객체를 인터페이스만으로 다루게 됨

     

4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다**

   - 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하던 상관없음

     - ex) OpenJDK에서의 `EnumSet` 클래스의 정적 팩터리 메서드는 원소가 64개 이하면 `RegularEnumSet` 의 인스턴스를, 65개 이상이면 `JumboEnumSet` 의 인스턴스를 반환함

       

5. **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다**

   - 서비스 제공자 프레임워크(service provider framework)의 근간이 됨 ex) JDBC
     - 서비스 제공자 프레임워크는 3개의 핵심 컴포넌트로 이루어짐 (종종 4 번째 컴포넌트도 쓰임)
       - 서비스 인터페이스(service interface): 구현체의 동작을 정의
       - 제공자 등록 API(provider registration API): 제공자가 구현체를 등록할 때 사용
       - 서비스 접근 API(service access API): 클라이언트가 서비스의 인스턴스를 얻을 때 사용
       - 서비스 제공자 인터페이스(service provider interface): 서비스 인터페이스의 인스턴스를 생성하는 팩터리 객체를 설명
         - 서비스 제공자 인터페이스가 없으면 각 구현체를 인스턴스화할 때 리플렉션을 사용해야함
     - 서비스 접근 API가 유연한 정적 팩터리의 실체라고 할 수 있음
       - 클라이언트가 별도의 조건을 명시하지 않으면 기본 구현체를 반환하고, 조건을 명시하면 지원하는 구현체들을 반환함
     - ex) JDBC
       - `Connection`: 서비스 인터페이스 역할
       - `DriverManager.registerDriver`: 제공자 등록 API 역할
       - `DriverManager.getConnection`: 서비스 접근 API 역할
       - `Driver`: 서비스 제공자 인터페이스 역할

## 단점

1. **상속을 하려면 `public`이나 `protected` 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다**

   - 상속보다 컴포지션을 사용하도록 유도하고, 불변 타입으로 만드려면 이 제약을 지켜야하므로 장점으로 받아들일 수 있음

     

2. **정적 팩터리 메서드는 프로그래머가 찾기 어렵다**

   - 생성자처럼 API 설명에 명확히 드러나지 않으므로 사용자가 정적 패터리 메서드를 찾아야한다
     - 이를 완화하기 위해 API 문서를 잘 쓰고, 정적 팩터리 메서드 이름을 잘 지어야함

### 주로 사용하는 명명법

- from: 매개 변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  - ex) `Date d = Date.from(instant);`
- of: 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  - ex) `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- valueOf: from과 of의 더 자세한 버전
  - ex) `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- instance / getInstance: (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않음
  - ex) `StackWalker luke = StackWalker.getInstance(options);`
- create / newInstance: instance / getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장
  - ex) `Object newArray = Array.newInstance(classObject, arrayLen);`
- getType: getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
  - ex) `FileStore fs = Files.getFileStore(path);`
- newType: newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용
  - ex) `Bufferedreader br = Files.newBufferedReader(path);`
- type: getType과 newType의 간결한 버전
  - ex) `List<Complaint> litany = Collections.list(legacyLitany);`

## 정리

> 정적 팩터리 메서드와 `public` 생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다.
>
> 일반적으로 정적 팩터리 메서드를 사용하는 것이 유리한 경우가 많으므로 무작정 `public` 생성자를 제공하던 습관이 있다면 고칠 것. 
