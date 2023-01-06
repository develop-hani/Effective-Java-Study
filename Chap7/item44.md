## 아이템 44. 표준 함수형 인터페이스를 사용하라


자바가 람다를 지원하면서 상위 클래스의 기본 메서드를 재정의해 원하는 동작을 구현하는 템플릿 메서드 패턴의 매력이 크게 줄었다. 이를 대체하는 현대적 해법은 같은 효과의 함수 객체를 받는 정적 팩터리나 생성자를 제공하는 것이다. 즉, 함수 객체를 매개변수로 받는 생성자와 메서드를 더 많이 만들어야 한다. 이때 함수형 매개변수 타입을 올바르게 선택해야 한다.
___
### 불필요한 함수형 인터페이스 - 대신 표준 함수형 인터페이스를 사용하라.

``` java
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
	return size() > 100;
}
```
`removeEldestEntry`를 위와 같이 재정의하면 맵에 원소가 100개가 될 때까지 커지다가, 그 이상이 되면 새로운 키가 더해질 때마다 가장 오래된 원소를 하나씩 제거한다. 즉, 가장 최근 원소 100개를 유지한다.

람다를 사용하면 훨씬 잘 해낼 수 있다. 

``` java
@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
```
이 함수형 인터페이스 역시 잘 작동하지만, 굳이 사용할 필요는 없다. 이미 같은 모양의 인터페이스가 준비되어 있기 때문이다. java.util.function 패키지를 보면 다양한 용도의 표준 함수형 인터페이스가 담겨있다. 
**필요한 용도에 맞는 게 있다면, 직접 구현하지 말고 표준 함수형 인터페이스를 활용하라.**
표준 함수형 인터페이스들은 유용한 디폴트 메서드를 많이 제공하므로 다른 코드와의 상호운용성도 크게 좋을 것이다. 
___
### 표준 함수형 인터페이스

### java.util.function

| 인터페이스 | 함수 시그니처 | 예 |
|---|:---:|---:|
|UnaryOperator<T> | `T apply(T t)` | `String::toLowerCase`|
|BinaryOperator<T> | `T apply(T t1, T t2)` | `BigInteger::add` |
|Predicate<T> | `boolean test(T t)` | `Collection::isEmpty` |
|Function<T>| `R apply(T t)` | `Arrays::asList` |
|Supplier<T>| `T get()` | `Instant::now` |
|Consumer<T>| `void accept(T t)` | `System.out::println` |
___
### Java 에서 기본적으로 제공하는 Functional Interfaces

<details>
<summary>1. Predicate</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```
`Predicate` 는 인자 하나를 받아서 `boolean` 타입을 리턴한다.
람다식으로는 `T -> boolean` 로 표현한다.
</div>
</details>
  
<details>
<summary>2. Consumer</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```
`Consumer` 는 인자 하나를 받고 아무것도 리턴하지 않는다.
람다식으로는 `T -> void` 로 표현한다.
소비자라는 이름에 걸맞게 무언가 (인자) 를 받아서 소비만 하고 끝낸다고 생각하면 된다.
</div>
</details>

<details>
<summary>3. Supplier</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
`Supplier` 는 아무런 인자를 받지 않고 T 타입의 객체를 리턴한다.
람다식으로는 `() -> T` 로 표현한다.
공급자라는 이름처럼 아무것도 받지 않고 특정 객체를 리턴한다.
</details>

<details>
<summary>4. Function</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```
`Function` 은 T 타입 인자를 받아서 R 타입을 리턴한다.
람다식으로는 `T -> R` 로 표현한다.
수학식에서의 함수처럼 특정 값을 받아서 다른 값으로 반환해준다.
T 와 R 은 같은 타입을 사용할 수도 있다.
</details>

<details>
<summary>5. Comparator</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```
`Comparator` 은 T 타입 인자 두개를 받아서 `int` 타입을 리턴한다.
람다식으로는 `(T, T) -> int` 로 표현한다.

</details>

<details>
<summary>6. Runnable</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
`Runnable` 은 아무런 객체를 받지 않고 리턴도 하지 않는다.
람다식으로는 `() -> void` 로 표현한다.
`Runnable` 이라는 이름에 맞게 "실행 가능한" 이라는 뜻을 나타내며 이름 그대로 실행만 할 수 있다고 생각하면 된다.
</details>
  
<details>
<summary>7. Callable</summary>
<div markdown="1">

``` java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}

```
`Callable` 은 아무런 인자를 받지 않고 T 타입 객체를 리턴한다.
람다식으로는 `() -> T` 로 표현한다.
`Runnable` 과 비슷하게 `Callable` 은 "호출 가능한" 이라고 생각하면 좀 더 와닿을 수 있다.
</details>

___
### Operator
기본 인터페이스는 기본 타입인 `int`, `long`, `double`용으로 3개씩 변형이 생긴다.
``` java
Int Predicate p = i -> ( i > 3);
LongBinaryOperator bo = Long::sum;
```
___
### SrcToResult, ToResult
Function 인터페이스는 기본 타입을 반환하는 변형이 총 9개가 더 있다.
  
입력 결과 타입이 기본 타입이면 접두어로 `SrcToResult`를 사용한다. 예를 들어 `long`을 받아 `int`로 반환하면 `LongToIntFunction`이 된다.  (총 6개)
  
나머지는 입력이 객체 참조이고 결과가 `int`, `long`, `doube`인 변형들이다. 예를 들어 `int[]` 인수를 받아 `long`을 반환하면 `ToLongFunction<int[]>` 이다. (총 3개)
___
  
### 기본 함수형 인터페이스 변형
  
특정 인자를 받는 `Predicate`, `Consumer`, `Function` 등은 두 개 이상의 타입을 받을 수 있는 인터페이스가 존재한다.
  
| 함수형 인터페이스 | Descripter | Method |
|---|:---:|---:|
| BiPredicate | `(T, U) -> boolean` | `boolean test(T t, U u)`|
| BiConsumer | `(T, U) -> void` | `void accept(T t, U u)`|
| BiFunction | `(T, U) -> R` | `R apply(T t, U u)`|
  
`BiFunction`에는 다시 기본 타입을 반환하는 세 변형 (ex : ToIntBiFuction<T, U> ...) 이 존재한다.
  
`Consumer`에도 객체 참조와 기본 타입 하나, 즉 인수를 2개 받는 변형인 `ObjDoubleConsumer<T>`, `ObjIntConsumer<T>`, `ObjLongConsumer<T>`가 존재한다. 

이렇게 해서 기본 인터페이스의 인수 2개짜리 변형은 총 9개이다.

표준 함수형 인터페이스는 대부분 기본 타입만 지원한다. 그렇다고 기본 함수형 인터페이스에 박싱된 기본 타입을 넣어 사용하지는 말자. 동작은 하지만 계산량이 많을 때는 성능이 현저히 느려진다.
  
___

### 언제 표준 함수형 인터페이스를 사용해야 할까?
  
  
표준 인터페이스 중 필요한 용도에 맞는 게 없다면 직접 작성해야 하며, 구조적으로 똑같은 표준 함수형 인터페이스가 있더라도 직접 작성해야만 할 때가 있다.
  
`Comparator<T>` 인터페이스의 경우, 구조적으로 `ToIntBiFunction<T, U>`와 동일하지만 독자적인 인터페이스로 존재해야 하는 이유가 몇 개 있다.

1. API에서 굉장히 자주 사용되는데, 이름이 그 용도를 아주 훌륭히 설명해준다.
2. 구현하는 쪽에서 반드시 지켜야 할 규약을 담고 있다.
3. 비교자들을 변환하고 조합해주는 유용한 디폴트 메서드들을 많이 담고 있다.
  
  
만약 전용 함수형 인터페이스를 작성하기로 했다면, '인터페이스'임을 명심해야 한다. 아주 주의해서 설계해야 한다.

### @FunctionalInterface 애너테이션
  
이 애너테이션을 사용하는 이유는 `@Override`를 사용하는 이유와 비슷하다. 프로그래머의 의도를 명시하는 것으로, 크게 세 가지 목적이 있다.

1. 해당 클래스의 코드나 설명 문서를 읽을 이에게 그 인터페이스가 람다용으로 설계된 것임을 알려준다.
2. 해당 인터페이스가 추상 메서드를 오직 하나만 가지고 있어야 컴파일되게 해준다.
3. 유지보수 과정에서 누군가 실수로 메서드를 추가하지 못하게 막아준다.
  
따라서, 직접 만든 함수형 인터페이스에는 항상 `@FunctionalInterface` 애너테이션을 사용하도록 한다.

### 함수형 인터페이스를 사용할 때의 주의점
서로 다른 함수형 인터페이스를 같은 위치의 인수로 받는 메서드들을 다중 정의해서는 안 된다.
  
클라이언트에게 불필요한 모호함만 줄 뿐이며, 이 때문에 실제로 문제가 일어나기도 한다.
