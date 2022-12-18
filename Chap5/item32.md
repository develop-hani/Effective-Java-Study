# 제네릭과 가변인수를 함께 쓸 때는 신중하라

가변인수를 구현하면 가변인수를 담기 위한 배열이 생기는데 이 배열을 노출하는 문제가 발생하였다.<br>
&rarr; varargs 매개변수에 제네릭이나 매개변수화 타입이 포함되면 알기 어려운 컴파일 경고가 발생한다.

메서드를 선언할 때 실체화 불가 타입으로 varargs 매개변수를 선언하면 컴파일러가 경고를 보낸다.<br>
메서드를 varargs 매개변수가 실체화 불가 타입이면 호출에도 경고가 발생한다.<br>
&rarr; 힙 오염 발생

```java
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList;		//힙 오염 발생
  String s = stringLists[0].get(0);	//ClassCastException
}
```

varargs 매개변수를 받는 메서드는 경고로 끝나는 이유는 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 메서드가 실무에서 유용하기 때문이다.

## @SafeVarargs

자바 7 이전에는 경고에 대해 해줄 수 있는 일이 없었다.
@SuppressWarning 에너테이션을 달아 경고를 숨겨야 했다.

자바 7에서는 @SafeVarargs 에너테이션으로 경고를 숨길수 있게 되었다.
@SafeVarargs는 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치다.

- 주의할 점
  - 메서드가 안전한 것이 확실하지 않으면 @SafeVarargs를 달아서는 안된다.
  - 제네릭이나 메개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달려 있어야 한다.
  - 재정의할 수 없는 메서드에만 @SafeVarargs를 달아야 한다.

- @SafeVarargs가 확신할 수 있게 만드는 방법
  - varargs 매개변수를 담는 배열에 아무것도 저장하지 않는다.
  - 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다.
  
&rarr; varargs 매개변수 배열이 호출자로부터 그 메서드로 순수하게 인수들을 전달하는 일만 한다면 메서드는 안전한다.


## 메서드가 안전하지 않은 예

메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일타임에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보를 주어지지 않아 타입을 잘못 판단할 수 있다.<br>
&rarr; 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 호출한 쪽의 콜스택으로까지 전이할 수도 있다.

- 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안되는 예
```java
static <T> T[] pickTwo(T a, T b, T c) { //varargs 메서드를 안전하게 작성하는 게 불가능한 상황
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return toArray(a,b);
    case 1: return toArray(a,c);
    case 2: return toArray(b,c);
  }
    throw new AssertionError();	//도달할 수 없다.
}

public static void main(String[] args) {
  String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```
컴파일러는 toArray에 넘길 T 인스턴스 2개를 담을 varargs 매개변수 배열을 만드는 코드를 생성한다.<br>
이 코드가 만드는 배열은 Object[]를 반환한다.<br>
attributes는 Object[] 하위 타입이 아니므로 형변환을 실패한다.

- varargs 매개변수 배열에 다른 메서드가 접근하도록 하용할 때에도 안전한 경우 
1. @SafeVarargs 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 경우
2. 이 배열 내용의 일부 함수를 호출만 하는 일반 메서드에 넘기는 

## varargs 매개변수 대신 List 매개변수 사용

@SafeVarargs를 사용하지 않고 List 매개변수를 사용할 수도 있다.<br>
List 매개변수를 사용하면 실수로 안전하다고 판단할 걱정을 할 필요 없이 타입 안정성을 검증할 수 있다.<br>
대신 속도가 조금 느려지고 코드가 지저분해질 수 있다.

List 매개변수는 varargs 메서드를 안전하게 작성하는 게 불가능한 상황에서도 쓸 수 있다.
```java
static <T> List<T> pickTwo(T a, T b, T c) {
  switch(ThreadLocalRandom.current().nextInt(3)) {
    case 0: return List.of(a,b);
    case 1: return List.of(a,c);
    case 2: return List.of(b,c);
  }
  throw new AssertionError();
}

public static void main(String[] args) {
  List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
}
```

- toArray 대신 List.of를 사용하면 안전하다.<br>
&rarr; List.of 메서드에는 @SafeVarargs 에너테이션이 달려있기 때문이다.<br>
- 배열 없이 제네릭만 사용하므로 타입 안전하다.

