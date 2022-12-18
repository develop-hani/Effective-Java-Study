## 아이템 29. 이왕이면 제네릭 타입으로 만들라

JDK가 제공하는 제네릭 타입과 메서드를 사용하는 일은 일반적으로 쉬운 편이지만, 제네릭 타입을 새로 만드는 일은 조금 더 어렵다.

### Ojbect 기반 스택 - 제네릭이 절실한 강력 후보
``` java
public class StackBasedObject {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public StackBasedObject() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

이 클래스는 원래 제네릭 타입이어야 마땅하다. 이 클래스를 제네릭으로 바꾼다고 해도 현재 버전을 사용하는 클라이언트에는 아무런 해가 없다. 오히려 지금 상태에서의 클라이언트는 스택에서 꺼낸 객체를 형변환해야 하는데, 이때 런타임 오류가 날 위험이 있다. 

### 제네릭 스택으로 가는 첫 단계 - 컴파일되지 않는다.

 일반 클래스를 제네릭 클래스로 만드는 첫 단계는 클래스 선언에 타입 매개변수를 추가하는 일이다. 
 스택이 담을 원소의 타입 하나만 추가하면 되고, 이때 타입 이름으로는 보통 `E`를 사용한다.
 그런 다음 코드에 쓰인 `Object`를 적절한 타입 매개변수로 바꾸고 컴파일한다.

``` java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러 발생
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

> Stack.java:8: generic array creation
elements = new E[DEFAULT_INITIAL_CAPACITY];

이 단계에서 대체로 하나 이상의 경고나 오류가 뜨는데,아이템 28에서 설명한 것처럼 E와 같은 실체화 불가 타입으로는 배열을 만들 수 없다. 배열을 사용하는 코드를 제네릭으로 만들려 할 때는 이 문제가 항상 발목을 잡는다. 해결책으로 두 가지가 있다.

### 해결책 1. 제네릭 배열 생성을 금지하는 제약을 대놓고 우회

`Object` 배열을 생성한 다음 제네릭 배열로 형변환해보자. 그러면 컴파일러는 오류 대신 경고를 내보낸다. 컴파일러가 이 프로그램의 타입 안전성을 보장할 방법이 없다. 따라서 직접 타입 안전성을 해치지 않음을 우리 스스로 확인해야 한다.

문제의 배열 `elements`는 `private` 필드에 저장되고, 클라이언트로 반환되거나 다른 메서드에 전달되지 않고, `push` 메서드를 통해 배열에 저장되는 원소의 타입은 항상 `E`이다. 따라서 이 비검사 형변환은 확실히 안전하며, `@Suppress Warnings` 애너테이션으로 해당 경고를 숨긴다.(아이템27)

``` java
//배열 elements는 push(E)로 넘어온 E 인스턴스만 담는다.
//따라서 타입 안전성을 보장하지만, 이 배열의 런타임 타입은 E[]가 아닌 Object[]이다.
@SuppressWarnings("unchecked")
public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY]; 
}
```
### 해결책 2. elements필드의 타입을 E[]에서  Object[]로 바꿈

이 경우 다른 에러가 발생한다.
> Stack.java:19: incompatible types
found: Object, required: E
        E result = elements[--size];
        
배열이 반환한 원소를 `E`로 형변환하면 오류 대신 경고가 뜬다.
>Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
        E result = (E) elements[--size];
    
`E`는 실체화 불가 타입이므로 컴파일러는 런타임에 이루어지는 형변환이지만 안전한지 증명이 불가능하다. 이번에도 마찬가지로 직접 증명한 다음 경고를 숨길 수 있다.

``` java
// 비검사 경고를 적절히 숨긴다
public E pop() {
    if (size == 0)
        throw new EmptyStackException();

    // push에서 E 타입만 허용하므로 이 형변환은 안전하다. 
    @SuppressWarnings("unchecked") E result = (E) elements[--size];

    elements[size] = null; // 다 쓴 참조 해제 
    return result;
}
```
### 차이점

- 첫 번째 방법은 가독성이 더 좋다. 배열의 타입을 `E[]`로 선언하여 오직 `E` 타입 인스턴스만 받음을 확실히 어필한다. 그리고 코드도 더 짧다.
- 첫 번째 방식에서는 형변환을 배열 생성 시 단 한 번만 해주면 된다.
하지만 `E`가 `Object`가 아닌 한 배열의 런타임 타입이 컴파일타임 타입과 달라 힙 오염(heap pollution)을 일으키니 주의해야 한다.
- 두 번째 방식에서는 배열에서 원소를 읽을 때마다 해줘야 한다.

### 다음은 명령줄 인수들을 역순으로 바꿔 대문자로 출력하는 프로그램

아래의 코드는 방금 만든 제네릭 `Stack` 클래스를 사용하는 모습을 보여준다. `Stack`에서 꺼낸 원소에서 `String`의 `toUpperCase` 메서드를 호출할 때 명시적 형변환을 수행하지 않으면, 이 형변환이 항상 성공함을 보장한다. 

``` java
public static void main(String[] args) {
    Stack<String> stack = new Stack<>();
    for (String arg : args)
        stack.push(arg);
    while (!stack.isEmpty())
        System.out.println(stack.pop().toUpperCase());
}
```
### 대부분의 제네릭 타입은 타입 매개변수에 제약을 두지 않는다.

- `Stack<Object>`, `Stack<int[]>`, `Stack<List<String>>` 등등.. 어떤 참조 타입으로도 `Stack`을 만들 수 있다.
  
- 단, 기본 타입은 사용할 수 없다. 하지만 이는 박싱된 기본 타입을 사용해 우회할 수 있다.
  
- 타입 매개변수에 제약을 두는 제네릭 타입도 있다.
   ex. `java.util.concurrent.DelayQueue`

``` java
class DelayQueue<E extends Delayed> implements BlockingQueue<E> {
    // ...
}
```
### 정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편하다.

- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 한다. 이를 위해서는 제네릭 타입으로 만들어야 할 경우가 많다.

- 기존 타입 중 제네릭이었어야 하는 것이 있다면 제네릭 타입으로 변경하도록 한다.

- 제네릭 타입을 사용하면 기존 클라이언트에는 아무 영향을 주지 않으면서, 새로운 사용자에게 타입 안전성이라는 편의를 제공해줄 수 있다.
