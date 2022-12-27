확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라
=
타입 안전 열거 패턴
```java
public class TypesafeOperation {
    private final String type;
    private TypesafeOperation(String type) {
        this.type = type;
    }

    public String toString() {
        return type;
    }
    
    public static final TypesafeOperation PLUS = new TypesafeOperation("+");
    public static final TypesafeOperation MINUS = new TypesafeOperation("-");
    public static final TypesafeOperation TIMES = new TypesafeOperation("*");
    public static final TypesafeOperation DIVIDE = new TypesafeOperation("/");
}
```
>클래스를 이용하고, 생성자를 private로 만들어 최초 정의된 객체만 참조할 수 있게 했다.
열거 타입은 이러한 타입 안전 열거 패턴보다 거의 모든 상황에서 우수하다.\
단, 타입 안전 열거 패턴은 확장할 수 있으나 열거 타입은 그럴 수 없다는 단점아닌 단점(대부분의 상황에서 열거 타입을 확장하는건 좋지 않은 선택이기 때문)이 있다.

확장할 수 있는 열거 타입의 쓰임
-
Operation 타입과 같은 연산 코드에서 각 원소는 기계가 수행하는 연산을 뜻한다. 
이러한 연산 코드에서 가끔 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 열어줘야 할 때가 있다.\
-> 열거 타입에서 임의의 인터페이스를 구현할 수 있다는 사실을 이용하여 이러한 효과를 낼 수 있다.
아래는 아이템 34의 Operation 타입을 확장할 수 있게 만든 코드다.
<details>
<summary>아이템 34의 Operation 타입</summary>
<div markdown="1">
    
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
}
```
</div>
</details>

```java
public interface Operation{
  double apply(double x, double y);
}

public enum BasicOperation implements Operation {// 인터페이스를 이용해 확장 가능 열거 타입을 흉내 냈다.
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

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override public String toString() {
        return symbol;
    }
}
```
열거 타입인 BasicOperation은 확장할 수 없지만 인터페이스인 Operation은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용하면 된다. 
apply가 인터페이스에 선언되어 잇으니 열거 타입에 추상 메서드로 선언하지 않아도 된다.\
이를 활용해 여러가지 원하는 연산을 새로 작성할 수 있는데 이러한 새로 작성한 연산들은 기존 연산을 쓰던 곳(Operation 인터페이스를 사용하도록 작성돼있는 곳)이면 어디든 쓸 수 있다.\

## 타입 수준에서의 확장
개별 인스턴스 수준에서뿐 아니라 타입 수준에서도, 기본 열거 타입 대신 확장된 열거 타입을 넘겨 확장된 열거 타입의 원소 모두를 사용하게 할 수도 있다.
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(ExtendedOperation.class, x, y);//test 메서드에 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다
}
private static <T extends Enum<T> & Operation> void test(//Class 객체가 열거 타입인 동시에 Operation의 하위 타입이어야 한다는 뜻
        Class<T> opEnumType, double x, double y) {
    for (Operation op : opEnumType.getEnumConstants())
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```
위 코드에서 Class 객체 대신 한정적 와일드 카드 타입인 Collection<? extends Operation>을 넘길수도 있다.
```java
public static void main(String[] args) {
    double x = Double.parseDouble(args[0]);
    double y = Double.parseDouble(args[1]);
    test(Arrays.asList(ExtendedOperation.values()), x, y);
}
private static void test(Collection<? extends Operation> opSet,
                         double x, double y) {
    for (Operation op : opSet)
        System.out.printf("%f %s %f = %f%n",
                x, op, y, op.apply(x, y));
}
```
이 코드는 이전 코드보다 덜 복잡하고 test 메서드가 더 유연한 장점이 있지만 특정 연산에서는 EnumSet과 EnumMap을 사용하지 못하는 단점이 있다.

### 인터페이스를 이용해 확장 가능한 열거타입을 흉내 내는 방식의 문제점
열거 타입까리 상속을 구현할 수 없다.
- 아무 상태에도 의존하지 않는 경우에는 디폴트 구현을 이용해 인터페이스에 추가하는 방법이 있다.\
(Operation 같은 경우 연산기호를 저장하고 찾는 로직이 BasicOperation과  ExtendedOperation에 모두 들어가야 한다)\
- 공유하는 기능이 많다면 그 부분을 별도의 도우미 클래스나 정적 도우미 메서드로 분리하여 중복을 줄일 수 있다.
