ordinal 메서드 대신 인스턴스 필드를 사용하라
=
## ordinal 메서드 : 열거 타입에서 해당 상수가 그 열거 타입에서 몇 번째 위치를 반환하는 메서드
### 열거 타입 상수와 연결된 정숫값이 필요할때 ordinal 메서드를 이용하면 좋을까?
-> 아니다.
```java
public enum Ensemble {//합주단의 종류를 연주자 수(1~10)에 따라 정의한 열거 타입
  SOLO, DUET, TRIO, QUARTET, QUINTET,
  SEXTET, SEPTET, OCTET, NONET, DECTET;
  
  public int numberOfMusicians() { return ordinal() + 1; }
}
```
위 코드의 문제점
- 상수 선언 순서를 바꾸는 순간 numberOfMusicians가 오동작한다.
- 이미 사용중인 정수와 값이 같은 상수는 추가할 방법이 없다.(복4중주는 8중주와 값이 같음)
- 값을 중간에 비워둘수 없다.(12명이 연주하는 3중 4중주를 추가하기위해서는 11명짜리 더미 상수(존재하지 않는 합주단)를 추가해야함

### 해결책
열거 타입 상수에 연결된 값을 ordinal 메서드로 얻지말고, 인스턴스 필드에 저장하도록 하자.
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;
    Ensemble(int size) { this.numberOfMusicians = size; }
    public int numberOfMusicians() { return numberOfMusicians; }
}
```

### ordinal 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓰는 경우가 아니라면 절대 사용하지 말자!
