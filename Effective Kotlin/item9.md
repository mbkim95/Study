# 아이템 9: use를 사용하여 리소스를 닫아라

- 코틀린/JVM에는 더 이상 필요하지 않을 때, `close` 메서드를 사용해서 명시적으로 닫아야 하는 리소스들이 많음
  - InputStream, OutputStream
  - java.sql.Connection
  - java.io.Reader(FileReader, BufferedReader, CSSParser)
  - java.new.Socket, java.util.Scanner

- 이러한 리소스들은 `AutoCloseable`을 상속받는 `Closeable` 인터페이스를 구현(implement)하고 있음
- 최종적으로 리소스에 대한 레퍼런스가 없어질 때, 가비지 컬렉터가 처리하지만 굉장히 느리고, 그동안 리소스를 유지하는 비용이 많이 들기 때문에 더 이상 필요하지 않을 때 명시적으로 `close` 메서드를 호출해 주는 것이 좋음

```kotlin
fun countCharactersInFile(path: String): Int {
    val reader = BufferedReader(FileReader(path))
    try {
        return reader.lineSequence().sumBy { it.length }
    } finally {
        reader.close()
    }
}
```

- 일반적으로 위와 같이 처리했지만 좋은 방법이 아님
  - 리소스를 닫을 때 발생하는 예외를 따로 처리하지 않음
  - `try` 블록과 `finally` 블록 내부에서 오류가 발생하면 둘 중 하나만 전파됨
- 표준 라이브러리에 `use` 함수로 대체할 수 있음

```kotlin
fun countCharactersInFile(path: String): Int {
    BufferedReader(FileReader(path)).use {
        return reader.lineSequence().sumBy { it.length }
    }
}
```

- 코틀린 라이브러리는 파일을 한 줄씩 처리할 때 활요할 수 있는 `useLines` 함수도 제공함

```kotlin
fun countCharactersInFile(path: String): Int {
    File(path).useLines { lines ->
        return lines.sumBy { it.length }
    }
}
```
