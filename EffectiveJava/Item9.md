# 아이템 9: try-finally 보다는 try-with-resources를 사용하라

- Java 라이브러리에는 `close` 메서드를 호출해 직접 닫아줘야 하는 자원이 많음
- ex) `InputStream` , `OutputStream` , `java.sql.Connection`
  - 이런 자원 중 상당수가 안전망으로 `finalize` 를 활용하고 있지만 그리 믿을만하지 않음(아이템 8)

## 전통적인 방법

- 전통적으로 자원을 제대로 닫힘을 보장하기 위해 try-finally가 쓰임

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

- 나쁘지 않은 방법이지만 자원을 하나 더 사용한 경우 처리가 어려워짐

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        } 
    }finally {
        in.close();
    }
}
```

- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있으므로 위의 코드들은 결점이 있음
  - 기기에 물리적인 문제가 생긴다면 `firstLineOfFile` 메서드 안의 `readLine` 메서드가 예외를 던지고, 같은 이유로 `close` 메서드도 실패함
  - 이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜서 스택 추적 내역에 첫 번째 예외 관련 정보는 남지 않아 디버깅이 어려움

## 개선된 방법

- Java 7에 try-with-resources 가 이 문제를 해결해줌
  - **이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야함**
    - `void`를 반환하는 `close` 메서드 하나만 정의된 인터페이스
    - **닫아야 하는 자원을 뜻하는 클래스를 작성한다면 `AutoCloseable` 을 반드시 구현할 것**

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
        new FileReader(path))) {
        return br.readLine();
    }
}
```

- 첫 번째 코드에 적용한 예시

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

- 두 번째 코드에 적용한 예시

- try-with-resources 버전이 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋음
  - `firstLineOfFile` 메서드의 `readLine` 과 (코드에는 나타나지 않는) `close` 호출 양쪽에서 예외가 발생하면, `close` 에서 발생한 예외는 숨겨지고 `readLine` 에서 발생한 예외가 기록됨
    - 숨겨진 예외들은 스택 추적 내역에 '숨겨졌다(suppressed)'로 출력됨

### catch 절

- try-finally에서처럼 try-with-resources에서도 catch 절을 사용할 수 있음

```java
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

## 정리

> 꼭 회수해야 하는 자원을 다룰 때는 예외 없이 try-finally 말고, try-with-resources를 사용할 것
>
> 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용함
>
> try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있음
