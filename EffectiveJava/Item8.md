# 아이템 8: finalizer와 cleaner 사용을 피하라

- Java는 두 가지 객체 소멸자를 제공
  - 그 중 `finalizer` 는 예측할 수 없고, 상황에 따라 위험할 수 있어서 일반적으로 불필요함
    - 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 함
    - Java 9에서는 `finalizer` 를 deprecated하고 `cleaner` 를 대안으로 소개함
  - `cleaner` 는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요함

## 주의

- Java의 `finalizer` 와 `cleaner` 는 C++의 파괴자(destructor)와는 다른 개념
  - C++에서 파괴자는 특정 객체와 관련된 자원을 회수하는 보편적인 방법
  - Java에서는 접근할 수 없게 된 객체를 가비지 컬렉터가 알아서 회수함
  - C++의 파괴자는 비메모리 자원을 회수하는 용도로 사용
    - Java에서는 try-with-resources와 try-finally를 사용 (아이템 9)

- `finalizer` 와 `cleaner` 는 즉시 수행된다는 보장이 없음
  - **`finalizer` 와 `cleaner` 로는 제때 실행되어야 하는 작업은 절대 할 수 없음**
  - ex) 파일 닫기를 `finalizer` 와 `cleaner` 에게 맡기면 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문에 중대한 오류를 일으킬 수 있음
    - 시스템이 `finalizer` 나 `cleaner` 실행을 게을리해 파일을 계속 열어 둔다면 새로운 파일을 열지 못해 프로그램이 실패할 수 있음

- `finalizer` 나 `cleaner` 를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달려있음
  - 가비지 컬렉터마다 천차만별 (JVM에 따라 달라질 수 있음)

- Java 언어 명세는 `finalizer` 나 `cleaner` 의 수행 시점 뿐 아니라 수행 여부조차 보장하지 않음
  - 프로그램 생애주기와 상관없는, **상태를 영구적으로 수정하는 작업에서는 절대 `finalizer` 나 `cleaner` 에 의존하면 안됨**
    - ex) 데이터베이스 같은 공유 자원의 영구 락(lock) 해제를 `finalizer` 나 `cleaner` 에게 맡기면 안됨
- `System.gc` 나 `System.runFinalization` 메서드에 현혹되면 안됨
  - `finalizer` 와 `cleaner` 가 실행될 가능성을 높여주나 보장해주진 않음
    - `System.runFinalizersOnExit` 와 `Runtime.runFinalizersOnExit` 는 보장해주지만 ThreadStop 이라는 심각한 결함이 있음
- `finalizer` 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료됨
  - 잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있음
  - 다른 스레드가 훼손된 객체를 사용하려하면 어떻게 동작할지 예측할 수 없음
  - `finalizer` 동작 중 예외가 발생해서 종료되면 경고조차도 출력되지 않음

- **`finalizer` 와 `cleaner` 는 심각한 성능 문제를 동반함**
  - 가비지 컬렉터의 효율을 떨어뜨리기 때문
- `finalizer` 를 사용한 클래스는 `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수 있음
  - 생성자나 직렬화 과정(readObject와 readResolve 메서드)에서 예외가 발생하면 생성되다 만 객체에서 악의적인 하위 클래스의 `finalizer` 가 수행될 수 있음
    - 이 `finalizer` 는 정적 필드에 자신의 참조를 할당해 가비지 컬렉터가 수집하지 못하게 막을 수 있음
  - 객체 생성을 막으려면 생성자에서 예외를 던지면 되지만, `finalizer` 가 있다면 그렇지 않음
    - `final` 클래스는 하위 클래스를 만들 수 없으므로 해당 공격에서 안전함
    - `final` 이 아닌 클래스는 빈 `finalize` 메서드를 만들고 `final `로 선언해서 방지할 수 있음

## 대안

- `AutoCloseable` 을 구현하고, 클라이언트에서 인스턴스를 다 쓰고 나면 `close` 메서드를 호출하는 것으로 파일이나 스레드 등 종료해야 할 자원을 담고 있는 객체의 클래스에서 `finalizer` 나 `cleaner` 를 대신해줄 수 있음
  - 일반적으로 예외가 발생해도 제대로 종료되도록 try-with-resources를 사용해야함(아이템 9)
  - 각 클래스는 자신이 닫혔는지 추적하는 것이 좋음
    - `close` 메서드에서 이 객체는 더 이상 유효하지 않음을 피르에 기록하고 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후에 불렸다면 `IllegalStateException` 을 던지는 방법

## 쓰임새

- `cleaner` 와 `finalizer` 는 2가지 쓰임새가 있음
  - 자원의 소유자가 `close` 메서드를 호출하지 않는 것에 대비한 안전망 역할
    - `cleaner` 나 `finalizer` 가 즉시 호출되리란 보장은 없지만, 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것이 더 좋음
    - 안전망 역할의 `finalizer` 를 작성할 때는 값어치가 있는지 심사숙고할 것
    - ex) `FileInputStream`, `FileOutputStream`, `ThreadPoolExecutor`
  - 네이티브 피어(native peer)와 연결된 객체
    - 네이티브 피어: 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 의미
    - 네이티브 피어는 Java 객체가 아니므로 GC는 존재를 알지 못함
      - Java 피어를 회수할 때 네이티브 객체까지 회수하지 못함
    - 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때는 `cleaner` 나 `finalizer` 를 사용할만함
    - 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 `close` 메서드를 이용할 것

### `cleaner` 사용법

- `cleaner` 는 사용하기에 조금 까다로움

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
    private static class State implements Runnable {
        int numJunkPiles; // 방(Room) 안의 쓰레기 수

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        // close 메서드나 cleaner가 호출된다.
        @Override
        public void run() {
            System.out.println("방 청소");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```

- `State` 는 `cleaner` 가 방을 청소할 때 수거할 자원을 담고 있음
- `numJunkPiles` 필드는 수거할 자원

- `State` 는 `Runnable` 을 구현하고, 그 안의 `run` 메서드는 `cleanable` 에 의해 딱 한번만 호출됨
- `cleanable` 객체는 `Room` 생성자에서 `cleaner` 에 `Room` 과 `State` 를 등록할 때 얻음

- `run` 메서드가 호출되는 상황은 둘 중 하나
  - `Room` 의 `close` 메서드를 호출할 때
    - `close` 메서드에서 `Cleanable` 의 `clean` 을 호출하면 이 메서드 안에서 `run` 을 호출함
    - 가비지 컬렉터가 `Room` 을 회수할 때까지 클라이언트가 `close` 를 호출하지 않는다면, `cleaner` 가 `State` 의 `run` 메서드를 호출함
- **`State` 인스턴스는 절대로 `Room` 인스턴스를 참조하면 안됨**
  - `Room` 인스턴스를 참조할 경우 순환참조가 생겨 가비지 컬렉터가 `Room` 인스턴스를 회수하지 않음
    - **정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 됨**(아이템 24)
    - 람다 역시 바깥 객체의 참조를 갖기 쉬우니 사용하지 않는 것이 좋음

### 개선

- 위 코드에서 `Room` 의 `cleaner` 는 단지 안전망으로만 쓰임
  - 클라이언트가 모든 `Room` 생성을 try-with-resources 블록으로 감쌌다면 자동 청소는 전혀 필요 없음

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room(7)) {
            System.out.println("안녕~");
        }
    }
}
```

- `Adult` 프로그램은 "안녕~" 을 출력한 후, 이어서 "방 청소"를 출력함

```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴");
    }
}
```

- "방 청소"는 한 번도 출력되지 않음
- `cleaner` 의 명세는 다음과 같음

> `System.exit` 을 호출할 때의 `cleaner` 동작은 구현하기 나름
>
> 청소가 이뤄질지는 보장하지 않음

- 명세에선 명시하지 않았지만 일반적인 프로그램 종료에서도 마찬가지
- 내 컴퓨터에서는 `Teenager` 의 `main` 메서드의 `System.gc()` 를 추가하는 것으로 종료 전에 "방 청소"를 출력할 수 있지만, 다른 컴퓨터에서도 그러리라는 보장은 없음

## 정리

> `cleaner`(Java 8까지는 `finalizer`)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용할 것
>
> 물론 이런 경우라도 불확실성과 성능 저하에 주의해야함
