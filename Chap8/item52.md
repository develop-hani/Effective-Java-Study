## 아이템52. 다중정의(overloading)는 신중히 사용하라

## 다중정의 문제

``` java
// 코드 52-1 컬렉션 분류기 - 오류! 이 프로그램은 무엇을 출력할까? 
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "집합";
    }

    public static String classify(List<?> lst) {
        return "리스트";
    }

    public static String classify(Collection<?> c) {
        return "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
___

```
그 외
그 외
그 외

Process finished with exit code 0
```
"집합", "리스트", "그 외"를 차례로 출력하기를 예상했지만, 실제로 수행해보면 "그 외"만 세 번 출력한다.

다중정의(overloading)된 메소드(classify)중 어떤것을 호출할지는 런타임시에 정해진다.
컴파일 타임에는 for 문 안에 c는 항상 `Collection<?>` 타입이다. 런타임에는 타입이 달라지지만, 영향을 주지 못한다.

위같은 이유는 재정의된 메소드(Override)는 동적으로 선택 되지만 다중정의된 메소드(overloading)는 정적으로 선택이 된다.

## 재정의

다형성을 이용해 메서드를 재정의한 것은 해당 객체의 런타임 타입이 어떤 메서드를 호출할지 기준이 된다. 복습 차원에서 적자면 메서드 재정의란 상위 클래스가 정의한 것과 똑같은 시그니처의 메서드를 하위 클래스에서 다시 정의한 것을 말한다. 메서드를 재정의한 다음 '하위 클래스의 인스턴스'에서 그 메서드를 호출하면 재정의한 메서드가 실행된다.

오버라이딩은 상위 클래스 메서드 재정의이고 오버로딩은 메서드 시그니처 차이에 따라 같은 이름의 메서드를 정의하는 것이다. 근본적으로 어느 런타임에 정해지는지의 차이가 있다.

```java
class Wine {
    String name() { return "포도주"; }
}
```
``` java
class SparklingWine extends Wine {
    @Override
    String name() {
        return "발포성 포도주";
    }
}
```
``` java
class Champagne extends SparklingWine {
    @Override
    String name() {
        return "샴페인";
    }
}
```
``` java
public class Overriding {
    public static void main(String[] args) {
        List<Wine> wineList = List.of(
                new Wine(), new SparklingWine(), new Champagne());

        for (Wine wine : wineList)
            System.out.println(wine.name());
    }
}
```
___
```
포도주
발포성 포도주
샴페인
```
컴파일타임 타입이 모두 `Wine`인 것에 무관하게 가장 하위에서 정의한 재정의 메서드가 실행되었다.

### 다중정의가 혼란을 주는 상황을 피하는 방법

### 1. 매개변수 수가 같은 다중정의는 만들지 말고, 메서드 이름을 다르게 지어주자

안전하고 보수적으로 가려면 매개변수 수가 같은 다중정의는 만들지 말자.
가변인수를 사용하는 메서드라면 다중정의는 아예 하지 말아야 한다.(아이템53)
이런 상황에서는 다중정의를 만들지 말고, 메서드 이름을 다르게 지어주자.

```java
public class FixedCollectionClassifier {
    public static String classify(Collection<?> c) {
        return c instanceof Set ? "집합" :
                c instanceof List ? "리스트" : "그 외";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };

        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
위처럼 인자를 받아 instanceof로 검사를 하면 된다.

다중정의를 피하기 위해 이름을 다르게 지어주는 예제로는 ObjectInputStream의 read 메서드를 보면 된다.

### 2. 생성자의 경우, 정적 팩터리를 사용하거나 근본적으로 다른 타입을 사용하자

생성자의 경우, 이름을 다르게 지을 수 없기 때문에 무조건 다중정의가 된다. 이때는 정적 팩터리가 대안이 될 수 있는 경우가 많다.

``` java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove(i);
        }
        System.out.println(set + " " + list);
    }
}
```
___
`[-3, -2, -1] [-2, 0, 2]`

위에 실행결과를 보면 우리가 예상했던 결과와 틀리다.

위에서는 `ArrayList`에서 `remove`를 다중정의 했기 때문이다.
그래서 같은 메소드를 호출해주기 위해 아래처럼 코드를 수정했다.

``` java
public class SetList {
    public static void main(String[] args) {
        Set<Integer> set = new TreeSet<>();
        List<Integer> list = new ArrayList<>();

        for (int i = -3; i < 3; i++) {
            set.add(i);
            list.add(i);
        }
        for (int i = 0; i < 3; i++) {
            set.remove(i);
            list.remove((Integer) i);
        }
        System.out.println(set + " " + list);
    }
}
```
___
`[-3, -2, -1] [-3, -2, -1]`

람다와 메소드 참조도 다중정의에 혼란을 이르켰다. 아래 코드를 보자.

``` java
public class Lamda {
    public static void main(String[] args) {

        new Thread(System.out::println).start();

        ExecutorService es = Executors.newCachedThreadPool();

        es.submit(System.out::println);

    }
}
```
___
`D:\repo\codeTestJDK8\src\main\java\com\github\sejoung\codetest\methods\overloading\Lamda.java
Error:(13, 11) java: reference to submit is ambiguous
  both method <T>submit(java.util.concurrent.Callable<T>) in java.util.concurrent.ExecutorService and method submit(java.lang.Runnable) in java.util.concurrent.ExecutorService match
Error:(13, 18) java: incompatible types: cannot infer type-variable(s) T
    (argument mismatch; bad return type in method reference
      void cannot be converted to T)`
      
그래도 여러 생성자가 같은 수의 매개변수를 받아야 하는 경우를 완전히 피해갈 수 없다면, 매개변수 중 하나 이상이 근본적으로 다르다(radically different) 면 헷갈일 일이 없다. 근본적으로 다르다는 건 null이 아닌 두 타입의 값을 서로 어느 쪽으로든 형변환할 수 없다는 뜻이다. 이 조건을 만족하면 매개변수들의 런타임 타입만으로 결정이 되어 혼란을 주는 주된 원인이 사라진다.
      
### 3. 메서드를 다중정의할 때, 서로 다른 함수형 인터페이스라도 같은 위치의 인수로 받아서는 안된다

서로 다른 함수형 인터페이스라도 서로 근본적으로 다르지 않기 때문이다.

`String` 클래스를 보면 `contentEquals` 메소드는 `forward` 시켜 버리는 방법을 선택했다. 같은 객체를 입력하면 동일한 기능을 수행한다.

``` java
public boolean contentEquals(StringBuffer sb) {
    return contentEquals((CharSequence)sb);
}

```
하지만 아래는 같은 객체를 넘겨도 다른 작업을 하게 된다.
``` java

public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}

  
public static String valueOf(char data[]) {
    return new String(data);
}

```

## 정리

프로그래밍 언어가 다중정의를 허용한다고 해서 다중정의를 꼭 활용하란 뜻은 아니다.
일반적으로 매개변수 수가 같을 때는 다중정의를 피하는 게 좋다.
상황에 따라, 특히 생성자라면 이 조언을 따르기가 불가능할 수 있다. 그럴 때는 헷갈릴 만한 매개변수는 형변환하여 정확한 다중정의 메서드가 선택되도록 해야 한다.
이것이 불가능하면, 예컨대 기존 클래스를 수정해 새로운 인터페이스를 구현해야 할 때는 같은 객체를 입력받는 다중정의 메서드들이 모두 동일하게 동작하도록 만들어야 한다. 그렇지 않으면 프로그래머들은 다중정의된 메서드나 생성자를 효과적으로 사용하지 못할 것이고, 의도대로 동작하지 않는 이유를 이해하지도 못할 것이다.
