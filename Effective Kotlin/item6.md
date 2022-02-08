# 아이템 6: 사용자 정의 오류보다는 표준 오류를 사용하라

- 예측하지 못한 상황을 나타내야 하는 경우가 있는데, 가능하다면 직접 오류를 정의하는 것보다는 최대한 표준 라이브러리의 오류를 사용하는 것이 좋음
  - 표준 라이브러리의 오류는 많은 개발자가 알고 있으므로, 이를 재사용하는 것이 좋음

## 일반적으로 사용하는 예외

- `IllegalArgumentException`: `require`를 사용할 때 `throw` 할 수 있는 예외
- `IllegalStateException`: `check`를 사용할 때 `throw` 할 수 있는 예외
- `IndexOutOfBoundsException`: 인덱스 파라미터의 값이 범위를 벗어났음을 의미하는 예외. 일반적으로 컬렉션 또는 배열과 함께 사용함
- `ConcurrentModificationException`: 동시 수정(concurrent modification)을 금지했는데 발생해 버렸음을 의미
- `UnsupportedOperationException`: 사용자가 사용하려고 했던 메서드가 현재 객체에서는 사용할 수 없음을 나타냄
- `NoSuchElementException`: 사용자가 사용하려고 했던 요소가 존재하지 않을 때를 나타냄.
  - ex) 내부에 요소가 없는 `Iterable`에 대해 `next`를 호출할 때
