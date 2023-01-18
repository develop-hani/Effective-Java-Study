## 아이템 61. 박싱된 기본 타입보다는 기본 타입을 사용하라

자바의 데이터 타입은 크게 두 가지로 나눠진다.

1. 기본 타입 : `int`, `double`, `boolean` ...
2. 참조 타입 : `Integer`, `Double`, `Boolean`

오토박싱과 오토언박싱 덕분에 두 타입이 자동으로 형변환되어 크게 구분하지 않고 사용할 수 있다. 하지만 두 개의 차이는 명확하게 구분된다.

___
## 기본 타입과 박싱된 기본 타입의 차이

### 1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 대해 식별성이라는 속성을 갖는다.
다시 말하면, 값이 같은 박싱된 기본 타입의 인스턴스가 두 개 존재할 경우, 이 두 개는 서로 다르게 식별될 수 있다.

``` java
public static void main(String [] args) {
  Comparator<Integer> naturalOrder = (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
  int num = naturalOrder.compare(new Integer(42), new Integer(42));
  System.out.println(num);
}
//실행 결과 : 1
```

두 `Integer` 객체를 비교했을때, 숫자의 값이 같아서 0을 반환해야 할텐데 `1` 을 반환하는 이유는 무엇일까?

첫 번째 `i < j` 에서 오토박싱된 `Integer` 인스턴스는 기본 타입을 변환되고 값이 작은지 평가한다. 하지만 두번째 `i = j` 에서는 '객체 참조'의 식별성을 검사하게 된다. 즉 내용 기준이 아닌 객체의 주소값을 기준으로 비교하게 되고, 서로 다른 `Integer` 인스턴스라면 `1`을 반환하게 되는 것이다.

> 박싱된 기본 타입에 `==`연산자를 사용하면 오류가 일어난다.

### 해결

오토박싱으로 검사 전에 바꾸어서 식별성 검사가 이루어지지 않게 하는 방법이 있다.

```java 
public static void main(String [] args) {
  Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, j = jBoxed; //오토언박싱
    return i < j ? -1 : (i == j ? 0 : 1);
  };
  int result = naturalOrder.compare(new Integer(43), new Integer(43));
  System.out.println(result);
}
```
지역 변수 2개를 두고 각각의 `Integer` 매개변수의 값을 기본 타입의 정수로 오토언박싱하여 저장해서 모든 비교가 저장한 기본타입으로 진행된다.

___
### 2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 `null` 값을 가질 수 있다. 

```java
public class BoxingNull {
  static Integer i;

  public static void main(String [] args) {
    if(i == 42) {
      System.out.println("믿을 수 없군!");
    }
  }
}

//실행 결과 : NullPointerException
```
`i == 42` 를 검사할때 `NullPointerException` 을 던지는 기이한 결과를 보여준다.

원인은, `Integer`가 다른 참조 타입 필드와 마찬가지로 초기값이 `null` 이기 때문이다.

거의 예외 없이 기본 타입과 박싱된 기본 타입을 혼용한 연산에서는 박싱된 기본 타입의 박싱이 자동으로 풀린다. 따라서 `null` 참조를 언박싱하게 되면 `NullPointerException`이 발생하는 것이다.

### 해결

`i`를 `int`로 선언해주면 된다.

```java
static int i;

public static void main(String [] args) {
  if(i == 42) {
    System.out.println("믿을 수 없군!");
  }
}
```
___
### 3. 기본타입이 박싱된 기본 타입보다 시간과 메모리 사용에 있어 더 효율적이다.

``` java
public static void main(String[] args) {
	Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
    	sum += i;
    }
    System.out.println(sum);
}
```
이 프로그램은 지역변수 sum 을 박싱된 기본 타입으로 선언하여 매우 느린 성능을 보여준다. 오류나 경고 없이 컴파일되지만, 박싱과 언박싱이 반복해서 일어나고 있기 때문이다.

___
## 박싱된 기본 타입의 용도

1. 컬렉션의 원소, 키, 값으로 쓴다.
- 컬렉션은 기본 타입으로 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 사용해야 한다.
- 일반화해서 말하면 매개변수화 타입이나 매개변수화 메서드의 타입 매개 변수로는 박싱된 기본 타입을 사용해야 한다.
- `ThreadLocal<int> -> ThreadLocal<Integer>`

2. 리플렉션을 통해 메서드를 호출할 때도 박싱된 기본 타입을 사용해야 한다.

___
## 정리
기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 되도록이면 기본 타입을 사용해라.

1. 두 박싱된 기본 타입을 `==` 연산자로 비교한다면 식별성 비교가 이뤄지므로 우리가 원하는 결과가 나타나지 않을 확률이 높다.

2. 같은 연산에서 언박싱과 박싱된 기본타입을 혼용해서 사용하면 박싱된 기본 타입이 언박싱이 이뤄지면 언박싱 과정에서 `NullPointerException` 던질 수가 있다.

3. 기본 타입을 박싱하는 작업은 필요 없는 객체를 생성하는 부작용을 나을 수 있다.
