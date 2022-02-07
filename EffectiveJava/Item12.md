#  아이템 12: toString을 항상 재정의하라

- `toString` 의 일반 규약
  - ''간결하면서 사람이 읽기 쉬운 형태의 유익한 정보'를 반환해야함
  - 모든 하위 클래스에서 이 메서드를 재정의할 것
- `toString` 을 잘 구현한 클래스를 사용한 시스템은 디버깅하기 쉬움
- `toString` 메서드는 객체를 `println`, `printf`, 문자열 연결 연산자(+), `assert` 구문에 넘길 때, 또는 디버거가 객체를 출력할 때 자동으로 호출됨
- `toString` 을 제대로 정의하지 않으면 쓸모없는 메시지만 로그에 남음

- **`toString` 은 그 객체가 가진 주요 정보를 모두 반환하는게 좋음**

## toString을 구현하는 방법

- `toString` 을 구현할 때면 반환값의 포맷을 문서화할지 정해야함

  - 값 클래스라면 문서화하기를 권장

    - ex) 전화번호 클래스, 행렬 클래스 등

  - 포맷을 명시하면 그 객체는 표준적이고, 명확하고, 사람이 읽을 수 있게 됨

    - 그 값 그대로 입출력에 사용하거나 CSV 파일처럼 사람이 읽을 수 있는 데이터 객체로 저장할 수 있음
    - 명시한 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩터리나 생성자를 함께 제공하는 것이 좋음
      - ex) `BigInteger`, `BigDecimal`, 대부분의 기본 타입 클래스
    - 포맷을 한번 명시하면 (그 클래스가 많이 쓰인다면) 평생 그 포맷에 얽매이게 됨
    - 포맷을 명시하지 않으면 향후 릴리스에서 정보를 더 넣거나 포맷을 개선할 수 있는 유연성을 얻게 됨

  - **포맷을 명시하던 명시하지 않던 의도를 명확히 밝혀야 함**

    - 포맷을 명시하는 경우

      - ```java
        /**
         * 이 전화번호의 문자열 표현을 반환한다.
         * 이 문자열은 "XXX-YYY-ZZZZ" 형태의 12글자로 구성된다.
         * XXX는 지역 코드, YYY는 프리픽스, ZZZZ는 가입자 번호다.
         * 각각의 대문자는 10진수 숫자 하나를 나타낸다.
         * 
         * 전화번호의 각 부분의 값이 너무 작아서 자릿수를 채울 수 없다면,
         * 앞에서부터 0으로 채워나간다. 예컨대 가입자 번호가 123이라면
         * 전화번호의 마지막 네 문자는 "0123"이 된다.
         */
        @Override
        public String toString() {
            return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
        }
        ```

    - 포맷을 명시하지 않는 경우

      - ```java
        /**
         * 이 약물에 관한 대략적인 설명을 반환한다.
         * 다음은 이 설명의 일반적인 형태이나,
         * 상세 형식은 정해지지 않았으며 향후 변경될 수 있다.
         * 
         * "[약물 #9: 유형=사랑, 냄세=테레빈유, 겉모습=먹물]"
         */
        @Override
        public String toString() { ... }
        ```

  - **`toString` 이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자**

    - 그렇지 않으면 `toString` 의 반환 값을 파싱할 수밖에 없음
      - 성능 저하
      - 포맷이 바뀌면 시스템이 망가질 수 있음

- 정적 유틸리티 클래스(아이템 4)는 `toString` 을 제공할 이유가 없음

- 대부분의 열거 타입(아이템 34)은 따로 재정의할 필요 없음

- 하위 클래스들이 공유해야 할 문자열 표현이 있는 추상 클래스는 `toString` 을 재정의해야함

## 정리

> 모든 구체 클래스에서 Object의 toString을 재정의할 것 (상위 클래스에서 이미 알맞게 재정의한 경우는 예외)
>
> toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해줌
>
> toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야함