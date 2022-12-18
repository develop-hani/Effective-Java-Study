# 한정적 와일드카드를 사용해 API 유연성을 높이라

매개변수화 타입은 불공변이다.<br> 예를 들어 List\<String>은 List\<Object>가 하는 일을 제대로 수행할 수 없다.(리스코프 치환 원칙)

## 와일드카드 타입을 사용하지 않을 때의 문제점
```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
}

//Stack<E>에 메서드 추가
public void pushAll(Iterable<E> src) {
	for (E e : src)
    	push(e);
}
//Stack<E>에 메서드 추가
public void popAll(Collection<E> dst) {
	while(!isEmpty())
    	dst.add(pop());
}
```

### pushAll() 추가할 때

Iterable과 Stack의 원소 타입이 일치하면 잘 작동한다.<br>

다음 코드와 같을 때 문제가 발생한다.

```java
Stack<Number> numberStack = new Stack<>();
Iterable<Integer> integers = ...;
numberStack.pushAll(integers);

```

매개변수타입이 불공변이기 때문에 오류 메세지가 뜬다. 
&rarr; Iterable\<Number>는 Iterable\<Integer>를 담을 수 없다.

- 해결책: 한정적 와일드 카드를 사용한다.
매개변수로 Iterable\<? extend E>를 사용하면 E와 E의 하위 타입을 담을 수 있다.

### popAll() 추가할 때

컬렉션과 스택의 타입이 일치하면 컴파일되고 문제없이 동작한다.

```java
Stack<Number> numberStack = new Stack<>();
Collection<Object> objects = ...;
numberStack.popAll(objects);
```

매개변수타입이 불공변이기 때문에 오류 메세지가 뜬다. 
&rarr; Collection\<Object>는 Collection\<Number>의 하위 타입이 아니다.

- 해결책: 한정적 와일드 카드를 사용한다.
매개변수로 Collection<? super E>를 사용하면 E의 상위 타입의 Collection을 담을 수 있다.

## PECS 원칙

**유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하라**

> 팩스(PECS): producer-extends, consumer-super

- 생산자는 extends 와일드카드 타입, 소비자는 super 와일드카드 타입을 사용하라.
- 입력 매개변수가 생산자와 소비자 역할을 동시에 한다면 와일드카드 타입을 써도 좋을 게 없다.
- 반환 타입에는 한정적 와일드카드 타입을 사용하면 안 된다.

**클래스 사용자가 와일드카드 타입을 신경 써야 한다면 그 API에 무슨 문제가 있을 가능성이 크다.**

## 명시적 타입 인수

자바 7까지는 타입 추론 능력이 부족해서 명시적 타입 인수를 사용해서 타입을 알려주어야 했다.<br>
자바 8부터는 목표 타이핑을 지원해서 명시적 타입 인수를 사용하지 않아도 알아서 컴파일 된다.

## PECS 두 번 사용하는 경우

```java
//다듬은 max 메서드
public static <E extends Comparable<? super E>> E max(
List<? extends E> list)
```

입력 매개변수에서는 E 인스턴스를 생산하므로 extends를 사용했다.<br>
Comparable은 E 인스턴스를 소비하므로 Comparable<E> 보다 Comparable<? super E>를 사용하는 편이 낫다.

### max 메서드를 다듬은 이유

ScheduledFuture 클래스가 Comparable\<ScheduledFuture>를 구현하지 않았기 때문이다.<br>
&rarr; ScheduledFuture의 상위 인터페이스인 Delayed는 Comparable\<Delayed>를 정의하였기 때문에 확장한 타입을 지원하기 위해서는 와일드카드 타입이 필요하다.

## 타입 매개변수와 와일드 카드

```java
// 비한정적 타입 매개변수를 사용한 경우
public static <E> void swap(List<E> list, int i, int j);
// 비한정적 와일드 카드를 사용한 경우
public static void swap(List<?> list, int i, int j);
```
어느 것을 사용하는 것이 나을까?
- **기본 윈칙: 메서드 선언에 타입 매개변수가 한 번만 나오면 와일드카드로 대체하라**<br>
public API를 간단히 하려면 와일드 카드를 사용한 경우가 낫다.

```java
// 와일드카드로 선언하면 안되는 경우
public static void swap(List<?> list, int i, int j) {
	list.set(i, list.set(j, list.get(i));
}

```

위의 코드가 와일드카드로 선언하면 안되는 이유는 리스트의 타입이 List<?>이므로 타입이 정해지지 않아 null 이외에는 어떤 값도 넣을 수 없다.

해결책: 와일드카드 타입의 실제 타입을 알려주는 private 도우미 메서드를 따로 작성해서 활용한다.
