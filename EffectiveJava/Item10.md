# 아이템 10: equals는 일반 규약을 지켜 재정의하라

- `equals` 메서드는 재정의하기 쉬워 보이지만 잘못하면 자칫하면 끔찍한 결과를 초래하므로 특정 상황에 해당한다면 재정의하지 않는 것이 최선

## 재정의하면 안되는 상황

### 각 인스턴스가 본질적으로 고유할 때

- 값을 표현하는게 아니라 동작하는 개체를 표현하는 클래스일 때
  - ex) Thread

### 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없을 때

- `java.util.regex.Pattern` 은 `equals` 를 재정의해서 두 `Pattern` 의 인스턴스가 같은 정규표현식을 나타내는지(논리적 동치성)를 검사 할 수 있음
  - 설계자가 클라이언트에서 이 방식을 원하지 않거나 필요하지 않다고 판단하면 재정의하지 않는 것이 좋음

### 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞을 때

- 대부분의 `Set` 구현체는 `AbstractSet` ,  `List` 구현체는 `AbstractList` , `Map` 구현체들은 `AbstractMap` 에 구현된 `equals` 를 상속받아 사용함

### 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없을 때

- `equals` 가 실수로라도 호출되는 걸 막고 싶다면 다음과 같이 구현할 것

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // 호출 금지!
}
```

## 재정의 해야할 때

- 객체 식별성(object identity)이 아니라 논리적 동치성을 확인해야 하는데 상위 클래스의 `equals` 가 논리적 동치성을 비교하도록 재정의되지 않았을 경우에는 재정의해야함

  - 객체 식별성: 두 객체가 물리적으로 같은가를 의미함

  - 일반적으로 값 클래스(Integer, String ...)가 해당됨
    - 값 클래스라 해도, 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하는 인스턴스 통제 클래스(아이템 1)라면 재정의하지 않아도 됨
    - `Enum` (아이템 34)은 논리적으로 같은 인스턴스가 2개 이상 생기지 않으므로 `equals` 가 논리적 동치성까지 확인해준다고 볼 수 있음

- `equals` 가 논리적 동치성을 확인하도록 재정의해두면 `Map` 의 키와 `Set` 의 원소로 사용할 수 있게됨

### 일반 규약

- `equals` 를 재정의할 때는 반드시 일반 규약을 따라야함
- `Object` 명세에 적힌 규약은 아래와 같음

> equals 메서드는 동치관계(equivalence relation)를 구현하며, 다음을 만족한다.
>
> - 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true
> - 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true이면 y.equals(x)도 true
> - 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고 y.equals(z)가 true면 x.equals(z)도 true
> - 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해도 항상 true 또는 false
> - null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

#### 반사성

- 객체는 자기 자신과 같아야 함

#### 대칭성

- 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 함

```java
public final class CaseInsensitiveString {
    private final String s;
    
    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }
    
    // 대칭성 위배!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(
                    ((CaseInsensitiveString) o).s);
        if (o instanceof String) // 한 방향으로만 작동한다!
            return s.equalsIgnoreCase((String) o);
        return false;
    }
    ... // 나머지 코드는 생략
}
```

- ```java
  CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
  String s = "polish";
  
  cis.equals(s); // true
  s.equals(cis); // false
  ```

  - `String` 은 `CaseInsensitiveString` 의 존재를 모르기 때문
  - 이 문제를 해결하기 위해서는 아래와 같이 바뀌어야함

```java
@Override
public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

#### 추이성

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
    
    ... // 나머지 코드는 생략
}
```

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    
    ... // 나머지 코드는 생략
}
```

- `ColorPoint` 에 `equals` 를 구현하지 않으면 색상 정보는 무시하게 되므로 `equals` 를 구현해야함

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- 위의 메서드는 대칭성에 위배될 수 있음

  - ```java
    Point p = new Point(1, 2);
    ColorPoint cp = new ColorPoint(1, 2, Color.RED);
    
    p.equals(cp); // true
    cp.equals(p); // false
    ```

  - `ColorPoint.equals` 가 `Point` 와 비교할 때는 색상을 무시하도록 수정해보자

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if (!(o instanceof ColorPoint))
        return o.equals(this);
    
    // o가 ColorPoint면 색상까지 비교한다.
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

- 이 방식은 대칭성은 지켜주지만 추이성을 위배함

  - ```java
    ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
    Point p2 = new Point(1, 2);
    ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
    
    p1.equals(p2); // true
    p2.equals(p3); // true
    p1.equals(p3); // false
    ```

  - 이 방식은 무한 재귀에 빠질 수도 있음

    - `Point` 의 또 다른 하위 클래스 `SmellPoint` 만들고 같은 방식으로 `equals` 를 구현했을 때, `myColorPoint.equals(mySmellPoint);` 를 호출하면 `StackOverflowError` 가 발생

- **객체 지향적 추상화의 이점을 포기하지 않는다면, 구체 클래스를 확장해 새로운 값을 추가하면서 `equals` 구약을 만족시킬 방법은 없음**

  - `equals` 안의 `instanceof` 검사를 `getClass` 검사로 바꾸면 가능할 것 같음

  - ```java
    @Override
    public boolean equals(Object o) {
        if (o == null || o.getClass() != getClass())
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
    ```

  - 위 코드는 같은 구현 클래스의 객체일 때만 `true` 를 반환하지만 리스코프 치환 원칙을 위배함

    - 리스코프 치환 원칙(Liskov substitution principle): 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하므로 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야함

    - ```java
      // 단위 원 안의 모든 점을 포함하도록 unitCircle을 초기화한다.
      private static final Set<Point> unitCircle = Set.of(
      		new Point(1, 0), new Point(0, -1),
      		new Point(-1, 0), new POint(0, -1));
      
      public static boolean onUnitCircle(Point p) {
          return unitCircle.contains(p);
      }
      ```

    - 주어진 점이 (반지름이 1인) 단위 원 안에 있는지 판별하는 메서드가 필요하다고 가정

    - ```java
      public class CounterPoint extends Point {
          private static final AtomicInteger counter = new AtomicInteger();
          
          public CounterPoint(int x, int y) {
              super(x, y);
              counter.increaseAndGet();
          }
          public static int numberCreated() { return counter.get(); }
      }
      ```

    - 이 때 `CounterPoint` 의 인스턴스를 `onUnitCircle` 메서드에 넘기면 `x`, ` y` 값과 무관하게 `false` 를 반환함

      - `onUnitCircle` 에서 사용하는 `Set` 을 포함한 대부분의 컬렉션은 `equals` 를 사용하는데, `CounterPoint` 의 인스턴스는 어떤 `Point` 와도 같을 수 없기 때문
      - `Point` 의 `equals` 를 `instanceof` 기반으로 구현했다면 정상적으로 동작함

- **구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만, "상속 대신 컴포지을 사용하라"(아이템 18)의 조언을 따르면 우회할 수 있음**

  - `Point` 를 상속하는 대신 `ColorPoint` 의 `private` 필드로 두고, `ColorPoint` 와 같은 위치의 일반 `Point` 를 반환하는 뷰(view) 메서드(아이템 6)를 `public` 으로 추가하는 방식

  - ```java
    public class ColorPoint {
        private final Point point;
        private final Color color;
    
        public ColorPoint(int x, int y, Color color) {
            point = new Point(x, y);
            this.color = Objects.requireNonNull(color);
        }
        
        /*
         * 이 ColorPoint의 Point 뷰를 반환한다.
         */
        public Point asPoint() {
            return point;
        }
        
        @Override
        public boolean equals(Object o) {
            if(!(o instanceof ColorPoint))
                return false;
            ColorPoint cp = (ColorPoint) ;
            return cp.point.equals(point) && cp.color.equals(color);
        }
        ... // 나머지 코드는 생략
    }
    ```

  - Java 라이브러리에서도 구체 클래스를 확장해 값을 추가한 클래스가 종종 있음

    - ex) `java.sql.Timestamp` 는 `java.util.Date` 를 확장한 후 `nanoseconds` 필드를 추가함
      - `Timestamp` 의 `equals` 는 대칭성을 위배하고, `Date` 객체와 한 컬렉션에 넣거나 서로 섞어 사용하면 이상하게 동작할 수 있음

  > 추상 클래스의 하위 클래스에서라면 `equals` 규약을 지키면서도 값을 추가할 수 있음
  >
  > "태그 달린 클래스보다는 클래스 계층구조를 활용하라"는 아이템 23의 조언을 따르는 클래스 계층구조에서는 아주 중요함.
  >
  > 아무런 값을 갖지 않는 추상 클래스 `Shape` 를 위에 두고, 이를 확장하여 `radius` 필드를 추가한 `Circle` 클래스와 `length` 와 `width` 필드를 추가한 `Rectangle` 클래스를 만들 수 있음
  >
  > 상위 클래스를 직접 인스턴스로 만드는 게 불가능하다면 지금까지 이야기한 문제들은 일어나지 않음

#### 일관성

- 두 객체가 같다면 (어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 함을 의미
- **클래스가 불변이든 가변이든 `equals` 판단에 신뢰할 수 없는 자원이 끼어들게 하면 안됨**
  - ex) `java.net.URL` 의 `equals` 는 주어진 URL과 매핑된 호스트의 IP 주소를 이용해 비교함
    - 호스트 이름을 IP 주소로 바꾸려면 네트워크를 통해야 하는데, 그 결과가 항상 같다고 보장할 수 없음
  - **이런 문제를 피하려면 `equals` 는 항시 메모리에 존재하는 객체만을 사용한 결정적(deterministic) 계산만 수행해야함**

#### null-아님

- 모든 객체가 `null` 과 같지 않아야 함

- ```java
  // 명시적 null 검사 - 필요 없다!
  @Override
  public boolean equals(Object o) {
      if (o == null)
          return false;
      ...
  }
  ```

- ````
  // 묵시적 null 검사 - 이쪽이 낫다.
  @Override
  public boolean equals(Object o) {
      if(!(o instanceof MyType))
          return false;
      MyType mt = (MyType) o;
      ...
  }
  ```

- `instanceof` 는 입력이 피연산자 또는 입력이 `null` 이면 `false` 를 반환함

## 양질의 equals 메서드 구현 방법

1. `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인
2. `instanceof` 연산자로 입력이 올바른 타입인지 확인
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사

- `equals` 를 다 구현했다면 '대칭성', '추이성', '일관성' 을 확인해볼 것

## 정리

> 꼭 필요한 경우가 아니면 `equals` 를 재정의하지 말 것
>
> 많은 경우에 `Object` 의 `equals` 가 원하는 비교를 수행해줌
>
> 재정의해야 할 때는 그 클래스의 핵심 필드 모두를 빠짐없이, 5가지 규약을 확실히 지켜가며 비교해야함
