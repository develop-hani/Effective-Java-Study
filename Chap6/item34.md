int 상수 대신 열거 타입을 사용하라
=
열거 타입 : 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입
-
## 자바에서 열거 타입을 지원하기 전
```java
//정수 상수를 한 묶음 선언해서 사용
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
### 단점
- 타입 안전을 보장할 방법이 없으며 표현력도 좋지 않다.
- 오렌지를 건네야 할 메서드에 사과를 보내고 동등 연산자(==)로 비교하더라도 컴파일러는 아무런 경로를 보내지 않는다.
- 자바가 정수 열거 패턴을 위한 별도 이름공간을 지원하지 않기 때문에 각 과일용 상수는 그 과일 이름을 앞에 붙여야한다.
- 상수의 값이 바뀌면 클라이언트도 반드시 다시 실행해야한다.
- 문자열로 출력하기 까다롭다.
>정수 대신 문자열 상수를 사용하는 변형 패턴도 있지만 상수의 의미를 출력할 수 있다는 점 외에는 더 안좋다.

## 열거 타입
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
### 열거 타입의 특징
- 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 밖에서 접근할 수 있는 생성자를 제공하지 않으므로 사실상 final이다.
- 컴파일타임 타입 안전성을 제공한다.
- 각자의 이름공간이 있어서 이름이 같은 상수도 평화롭게 공존한다.
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일하지 않아도 된다.
- 열거 타입의 toString 메서드는 출력하기에 적합한 문자열을 내어준다.
- 열거 타입에는 임의의 메서드나 필드를 추가할 수 있고 임의의 인터페이스를 구현하게 할 수도 있다.

### 열거 타입에서 상수를 하나 제거하면?
-> 제거한 상수를 참조하지 않는 클라이언테에는 아무 영향이 없다.
-> 제거한 상수를 참조하는 클라이언트에서는 정수 열거 패턴과 다르게 유용한 컴파일오류/예외가 발생한다.

### 상수가 더 다양한 기능을 제공해줬으면 할 때
**상수마다 동작이 달라져야하는 상황**\
-> 열거 타입에 apply라는 추상 메서드를 선언하고, 각 상수별 클래스 몸체, 즉 각 상수에서 자신에 맞게 재정의하여 사용
```java
public enum Operation {//상수별 메서드 구현을 상수별 데이터와 결합, Operation의 toString을 재정의해 해당 연산을 뜻하는 기호를 반환받도록 함
    PLUS("+") {
        public double apply(double x, double y) { return x + y; }
    },
    MINUS("-") {
        public double apply(double x, double y) { return x - y; }
    },
    TIMES("*") {
        public double apply(double x, double y) { return x * y; }
    },
    DIVIDE("/") {
        public double apply(double x, double y) { return x / y; }
    };

    private final String symbol;

    Operation(String symbol) { this.symbol = symbol; }

    @Override public String toString() { return symbol; }

    public abstract double apply(double x, double y);
```
### 열거 타입의 메서드
- 열거 타입에는 상수 이름을 입력받아 그 이름에 해당하는 상수를 반환해주는 valuesOf(String) 메서드가 자동 생성된다.
- 열거 타입의 toString 메서드를 재정의 할 때는 toString이 반환하는 문자열을 해당 열거 타입 상수로 변환해주는 fromString 메서드도 함께 제공하는걸 고려하자
```java
//모든 열거 타입에서 사용할 수 있도록 구현한 fromString(단, 타입 이름을 적절히 바꾸고 모든 상수의 문자열 표현이 고유해야 함)
private static final Map<String, Operation> stringToEnum =
        Stream.of(values()).collect(
                toMap(Object::toString, e -> e));

// 지정한 문자열에 해당하는 Operation을 (존재한다면) 반환한다.
public static Optional<Operation> fromString(String symbol) {
    return Optional.ofNullable(stringToEnum.get(symbol));//주어진 문자열이 가리키는 연산이 존재하지 않을 수 있음을 클라이언트에 알리고 대처하도록 한 반환형태
}
```
### 이외의 열거 타입 특징들
- 상수별 메서드 구현에는 열거 타입 상수끼리 코드를 공유하기 어렵다는 단점이 있다.
-> 전략 열거 타입을 이용하여 새로운 상수를 추가할 때 적절한 메서드를 선택하도록 한다
- switch 문은 열거 타입의 상수별 동작을 구현하는데 적합하지 않지만 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 고려해볼만 하다.


# 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자
