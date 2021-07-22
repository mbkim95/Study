# 아이템 2: 생성자에 매개변수가 많다면 빌더를 고려하라

- 정적 팩터리와 생성자는 선택적 매개변수가 많을 때 적절히 대응하기 어렵다는 단점이 있다

## 대안 1: 점층적 생성자 패턴(telescoping constructor pattern)

- 점층적 생성자 패턴(telescoping constructor pattern)을 사용해서 해결할 수 있지만, 매개변수 개수가 더 늘어나면 클라이언트 코드를 작성하거나 가독성이 떨어진다는 한계가 있음

  - ex) 원하는 매개변수를 모두 포함한 생성자 중에서 가장 짧은 것을 선택해야
    - `Nutritionfacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);`

  ```java
  public class NutritionFacts {
      private final int servingSize;  // (ml, 1회 제공량) 필수
      private final int servings;     // (회, 총 n회 제공량) 필수
      private final int calories;     // (1회 제공량당) 선택
      private final int fat;          // (g/1회 제공량) 선택
      private final int sodium;       // (mg/1회 제공량) 선택
      private final int carbohydrate; // (g/1회 제공량) 선택
  
      public NutritionFacts(int servingSize, int servings) {
          this(servingSize, servings, 0);
      }
  
      public NutritionFacts(int servingSize, int servings, int calories) {
          this(servingSize, servings, calories, 0);
      }
  
      public NutritionFacts(int servingSize, int servings, int calories, int fat) {
          this(servingSize, servings, calories, fat, 0);
      }
  
      public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
          this(servingSize, servings, calories, fat, sodium, 0);
      }
  
      public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
          this.servingSize = servingSize;
          this.servings = servings;
          this.calories = calories;
          this.fat = fat;
          this.sodium = sodium;
          this.carbohydrate = carbohydrate;
      }
  }
  ```

## 대안 2: 자바빈즈 패턴(JavaBeans pattern)

- 또다른 대안으로 자바빈즈 패턴(JavaBeans pattern)이 있음
  - 매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식

```java
public class NutritionFacts {
    private int servingSize = -1;  // 필수, 기본값 없음
    private int servings = -1;     // 필수, 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {
    }

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```

- 점증적 생성자 패턴보다 코드가 길어졌지만 인스턴스 만들기가 더 쉽고, 가독성이 좋음

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

### 단점

- 객체 하나를 만드려면 메서드를 여러 개 호출해야함
- 객체가 완전히 생성되기 전까지 일관성(consistency)이 무너진 상태에 놓임
  - 클래스를 불변으로 만들 수 없어서 스레드 안정성을 얻기 위해 추가 작업이 필요함
  - 단점을 완화하고자 생성이 끝난 객체를 수동으로 얼리고(freezing), 얼리기 전에는 사용할 수 없도록 처리하기도 함
    - 다루기 어려워 실전에서는 거의 사용되지 않음
    - 객체 사용 전에 `freeze` 메서드가 확실히 호출됐는지 컴파일러가 보증할 수 없음 -> 런타임 오류에 취약

## 대안 3: 빌더 패턴(Builder pattrern)

- 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비한 패턴

- 사용법
  1. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻음
  2. 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정함
  3. 마지막으로 매개변수가 없는 `build` 메서드를 호출해 객체를 얻음 (보통은 불변 객체)

- 일반적으로 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어둠
  - ex) `NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();`

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int calories) {
            this.calories = calories;
            return this;
        }

        public Builder fat(int fat) {
            this.fat = fat;
            return this;
        }

        public Builder sodium(int sodium) {
            this.sodium = sodium;
            return this;
        }

        public Builder carbohydrate(int carbohydrate) {
            this.carbohydrate = carbohydrate;
            return this;
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

- 빌더 패턴은 명명된 선택적 매개변수(named optional parameters)를 흉내낸 패턴

- 유효성 검사
  - 잘못된 매개변수를 최대한 일찍 발견하려면 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, `build` 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식(invariant)를 검사하라
  - 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야함
  - 검사해서 잘못된 점이 발견되면 `IllegalArgumentException` 을 던지면 됨
    - 불변식: 프로그램이 실행되는 동안, 혹은 정해진 기간 동안 반드시 만족해야 하는 조건
      - ex) `List` 의 크기는 음수가 될 수 없다, `Period` 클래스에서 `start` 필드의 값이 `end` 필드의 값보다 앞서야 함
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기 좋음
  - 각 계층의 클래스에 관련 빌더를 멤버로 정의하라
    - 추상 클래스는 추상 빌더, 구체 클래스(concrete class)는 구체 빌더를 갖게함

```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTppings(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의(overriding) 해서 "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

- 추상 메서드 `self` 는 하위 클래스에서 형변환 하지 않고도 메서드 연쇄를 할 수 있도록 해주는데, 이를 시뮬레이트한 셀프 타입(simulated self-type) 관용구라고 함

```java
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```



```java
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

- 각 하위 클래스 빌더가 정의한 `build` 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언함
- 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 공변반환 타이핑(covariant return typing)이라함
  - 공변반환 타이핑을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있음

### 단점

- 객체를 만들려면 그에 앞서 빌더부터 만들어야함
  - 성능에 민감한 상황에서 문제가 될 수 있음
- 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 함

## 정리

> 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다.
>
> 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 낫다.
>
> 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
