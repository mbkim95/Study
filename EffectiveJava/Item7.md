# 아이템 7: 다 쓴 객체 참조를 해제하라

- Java처럼 가비지 컬렉터를 갖춘 언어를 사용하다보면 자칫 메모리 관리에 신경 쓰지 않아도 된다고 오해할 수 있음

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

- 특별한 문제가 없어 보이나 사실 **메모리 누수** 문제가 있다
  - 이 스택을 사용하는 프로그램을 오래 실행하다보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 성능이 저하될 것
  - 심한 경우에는 디스크 페이징이나 `OutOfMemoryError` 를 일으켜 프로그램이 종료될 수도 있음
- 이 코드에서는 스택이 커졌다가 줄어들었을 대 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않음
  - 스택이 그 객체들의 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문
    - 다 쓴 참조(obsolete reference): 앞으로 다시 쓰지 않을 참조

## 메모리 누수

- 가비지 컬렉션 언어에서는 메모리 누수를 찾기가 아주 까다로움
- 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체(그리고 또 그 객체들이 참조하는 모든 객체∙∙∙)를 회수하지 못함
  - 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할 수 있고 잠재적으로 성능에 악영향을 줄 수 있음

### 원인

- 스택이 자기 메모리를 직접 관리하기 때문에 메모리 누수가 발생함
  - 객체 참조를 담는 elements 배열로 저장소 풀을 만들어 원소들을 관리하고 있음
  - 배열의 활성 영역(0 ~ size - 1)에 속한 원소들이 사용되고 비활성 영역(size ~ elements.size - 1)은 쓰이지 않음
  - 가비지 컬렉터는 어디부터 어디까지가 활성영역인지 알 수 없음

## 해법

- 해당 참조를 다 썼을 때 `null` 처리(참조 해제)하면 메모리 누수를 방지할 수 있음

```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

- 만약 `null` 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 `NullPointerException` 을 던지며 종료됨
  - 프로그램 오류는 가능한 한 조기에 발견되는게 좋음

### 주의

- 하지만 모든 객체를 다 쓰자마자 일일이 `null` 처리하는 것은 바람직하지 않음
  - **객체 참조를 `null` 처리하는 일은 예외적인 경우여야 함**
    - 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위(scope) 밖으로 밀어내는 것

- 프로그래머는 비활성 영역이 되는 순간 `null`  처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에게 알려야함

## 메모리 누수 발생 케이스

### 자기 메모리를 직접 관리하는 클래스

- 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 `null` 처리해줘야함

### 캐시

- 객체 참조를 캐시에 넣고 나서, 이 사실을 잊은 채 , 그 객체를 한참을 그냥 나두는 일이 빈번함

#### 해결책

- 캐시 외부에서 키(key)를 참조하는 동안만(값이 아님) 엔트리가 살아 있는 캐시가 필요한 상황
  - `WeakHashMap` 을 사용해 캐시를 만들자
    - 다 쓴 엔트리는 즉시 자동으로 제거됨
    - `WeakHashMap` 은 이런 상황에서만 유용함을 기억할 것
- 캐시를 만들 때 캐시 엔트리의 유효 기간을 정확히 정의하기 어려움
  - 시간이 지날수록 엔트리의 가치를 떨어뜨리는 방식을 흔히 사용함
  - 쓰지 않는 엔트리를 이따금 청소해줘야하는데, (`ScheduledThreadPoolExecutor` 같은) 백그라운드 스레드를 활용하거나 캐시에 새 엔트리를 추가할 때 부수 작업으로 수행하는 방법이 있음
    - ex) `LinkedHashMap` 은 `removeEldestEntry` 메서드를 사용해 후자의 방식으로 처리함
    - 더 복잡한 캐시를 만들고 싶다면 `java.lang.ref` 패키지를 직접 활용해야함

### 리스너(listener) 혹은 콜백(callback)

- 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 콜백은 계속 쌓여감
  - 콜백을 약한 참조(weak reference)로 저장하면 가비지 컬렉터가 즉시 수거해감
    - ex) `WeakHashMap` 에 키로 저장하는 방법

## 정리

> 메모리 누수는 겉으로 잘 드러나지 않아 시스템에 수년간 잠복하는 사례도 있음
>
> 이런 누수는 철저한 코드 리뷰나 힙 프로파일러 같은 디버깅 도구를 동원해야만 발견되기도 함
>
> 이런 종류의 문제는 예방법을 익혀두는 것이 매우 중요
