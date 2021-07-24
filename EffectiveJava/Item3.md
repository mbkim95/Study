# 아이템 3: private 생성자나 열거 타입으로 싱글턴임을 보증하라

- 싱글턴(singleton)이란 **인스턴스를 오직 하나만 생성할 수 있는 클래스**를 말함
  - ex) 함수와 같은 무상태 객체, 설계상 유일해야 하는 시스템 컴포넌트
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려움
  - 타입을 인터페이스로 정의해서 만든 싱글턴이 아니면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없음

## 생성 방법 1

- public static 멤버를 final 필드로 두는 방법

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```

- private 생성자는 `Elvis.INSTANCE` 를 초기화할 때 딱 한 번만 호출됨
  - public, protected 생성자가 없으므로 인스턴스가 전체 시스템에 하나뿐임이 보장됨

### 장점

- 해당 클래스가 싱글턴임이 API에 명백히 드러남
- 간결함

### 예외

- 권한이 있는 클라이언트에서 `AccessibleObject.setAccessible` 을 사용해 private 생성자를 호출할 수 있음
  - 이러한 공격을 방어하려면 생성자를 수정해 두 번 째 객체가 생성되려고 하면 예외를 던지게 하면 됨

## 생성 방법 2

- 정적 팩터리 메서드를 public static 멤버로 제공하는 방법

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    
    public void leaveTheBuilding() { ... }
}
```

### 장점

- API를 바꾸지 않고도 싱글턴이 아니게 할 수 있음
  - 팩터리 메서드에서 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수도 있음
- 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있음

### 예외

- 권한이 있는 클라이언트에서 `AccessibleObject.setAccessible` 을 사용해 private 생성자를 호출할 수 있음
  - 이러한 공격을 방어하려면 생성자를 수정해 두 번 째 객체가 생성되려고 하면 예외를 던지게 하면 됨

### 직렬화

- 클래스를 직렬화 하려면 모든 인스턴스 필드를 일시적(transient)이라 선언하고, `readResolve()` 를 제공해야 함

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
    // '진짜' Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```

## 생성방법 3

- 원소가 하나인 열거 타입을 선언하는 방법

```java
public enum Elvis {
    INSTANCE;
    
    public void leaveTheBuilding{ ... }
}
```

### 장점

- public 필드 방식보다 더 간결함
- 복잡한 직렬화 상황이나 리플렉션 공격에서도 2번째 인스턴스가 생긱는 일을 막아줌

### 단점

- 만들려는 싱글턴이 `Enum` 외의 클래스를 상속해야 한다면 사용할 수 없음
  - 열거 타입이 다른 인터페이스를 구현하도록 선언 가능
