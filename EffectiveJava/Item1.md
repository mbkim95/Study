# 생성자 대신 정적 팩토리 메소드를 고려하라
클래스는 생성자와 별도로 정적팩토리 메소드(static factory method)를 제공할 수 있다. 이 방식에는 장점과 단점이 모두 존재한다. 우선 장점을 먼저 알아보자.

## 1. 이름을 가질 수 있다.
생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못하지만, 정적 팩토리 메소드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다. 예제를 보며 이해해보자.
```java
public class Person {
    String name;
    String address;

    public static Person withName(String name) {
        Person person = new Person();
        person.name = name;
        return person;
    }

    public static Person withAddress(String address) {
        Person person = new Person();
        person.address = address;
        return person;
    }

    public static void main(String[] args) {
        Person person1 = Person.withName("김철수");
        Person person2 = Person.withAddress("서울시 OO구 OO동");        
    }
}
```
위의 예제 코드에서 `withAge()`, `withHeight()`와 같은 정적 팩토리 메소드를 사용함으로써 반환될 객체의 특성을 쉽게 파악할 수 있게 된다.

그리고 하나의 시그니처로는 생성자를 하나만 만들 수 있기 때문에 한 클래으세 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩토리 메소드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주는 방식으로 활용할 수도 있다.

## 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
이 덕분에 불변 클래스(immutable class)는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
```java
public class Person {
    String name;
    String address;
    private static final Person HONG_KILDONG = Person.withName("홍길동");

    public static Person withName(String name) {
        Person person = new Person();
        person.name = name;
        return person;
    }

    public static Person withAddress(String address) {
        Person person = new Person();
        person.address = address;
        return person;
    }

    public static Person getHongKildong() {
        return HONG_KILDONG;
    }

    public static void main(String[] args) {
        Person person1 = Person.withName("김철수");
        Person person2 = Person.withAddress("서울시 OO구 OO동");
        Person person3 = Person.getHongKildong();
    }
}
```

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
정적 팩토리 메소드는 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 제공한다. API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어서 API를 작게 유지할 수 있다.

자바 컬렉션 프레임워크는 총 45개의 유틸리티 구현체를 제공하는데, 이 구현체 대부분을 단 하나의 인스턴스화 불가 클래스인 `java.util.Colletions`에서 정적 팩토리 메소드를 통해 얻도록 했다. 컬렉션 프레임워크는 이 45개의 클래스를 공개하지 않기 때문에 API 외견을 훨씬 작게 만들 수 있었고, 이는 API를 사용하기 위해 익혀야 하는 개념의 수와 난이도를 낮췄다는 것을 의미한다.

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
예를 들어 `EnumSet` 클래스는 public 생성자 없이 오직 정적 팩토리만 제공하는데, Open JDK에서는 원소의 수에 따라서 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다. 원소가 64개 이하면 `RegularEnumSet`의 인스턴스, 65개 이상이면 `JumboEnumSet`의 인스턴스를 반환한다. 하지만 클라이언트에서는 팩토리가 건네주는 객체가 어떤 인스턴스인지 알 수 없고, 알 필요도 없다. 그저 `EnumSet`의 하위 클래스이기만 하면 된다.

## 5. 정적 팩토리 메소드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
이러한 유연성을 제공하는 static 팩토리 메소드는 서비스 프로바이더 프레임워크의 근본이다. JDBC를 예로 들고 있다.

서비스 프로바이더 프레임워크는 서비스의 구현체를 대표하는 서비스 인터페이스와 구현체를 등록하는데 사용하는 프로바이더 등록 API 그리고 클라이언트가 해당 서비스의 인스턴스를 가져갈 때 사용하는 서비스 엑세스 API가 필수로 필요하다. 부가적으로, 서비스 인터페이스의 인스턴스를 제공하는 서비스 프로바이더 인터페이스를 만들 수도 있는데, 그게 없는 경우에는 리플랙션을 사용해서 구현체를 만들어 준다.

JDBC의 경우, `DriverManager.registerDriver()`가 프로바이더 등록 API, `DriverManager.getConnection()`이 서비스 엑세스 API, 그리고 Driver가 서비스 프로바이더 인터페이스 역할을 한다.

자바 6부터는 `java.util.ServiceLoader`라는 일반적인 용도의 서비스 프로바이더를 제공하지만, JDBC가 그 보다 이전에 만들어졌기 때문에 JDBC는 ServiceLoader를 사용하진 않는다.

이번엔 단점을 알아보자.
## 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩토리 메소드만 제공하면 하위 클래스를 만들 수 없다.
컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없는 것을 말한다. 하지만 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야한다는 점에서 장점으로 받아들여질 수도 있다.

## 2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다.
생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩토리 메소드 방식 클래스를 인스턴스화할 방법을 알아내야한다. 때문에 API 문서를 잘 작성하고 메소드 이름도 널리 알려진 규약을 따라 짓는 방식으로 문제를 완화해야 한다.

### 핵심 정리
> #### 정적 팩토리 메소드와 public  생성자는 각자의 쓰임새가 있으니 상대적인 장단점을 이해하고 사용하는 것이 좋다. 그렇다고 하더라도 정적 팩토리를 사용하는 게 유리한 경우가 더 많으므로 무작정 public 생성자를 제공하던 습관이 있다면 고치자.