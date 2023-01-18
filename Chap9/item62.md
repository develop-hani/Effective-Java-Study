## 아이템 62. 다른 타입이 적절하다면 문자열 사용을 피하라

문자열(String)을 쓰지 않아야 할 상황들에 대해 알아보자.

___

## 문자열은 다른 값 타입을 대신하기에 적합하지 않다

키보드 입력으로부터 데이터를 받을때 실제 데이터가 수치형임에도 불구하고 문자열로 받는 경우가 흔하다. 숫자를 표현할때는 `int`, `float` 등등의 타입이 좋고 예/아니오는 `boolean` 타입이 좋다.

___
## 문자열은 열거 타입을 대신하기에 적합하지 않다

상수를 열거할때는 문자열보다 열거 타입이 월등히 낫다.
[아이템 34](https://github.com/pyh-dotcom/Effective-Java-Study/blob/main/Chap6/item34.md)

___
## 문자열은 혼합타입을 대신하기 적합하지 않다

문자열은 혼합 타입을 대신하기에 적합하지 않다. 즉, 여러 요소가 혼합된 데이터를 하나의 문자열로 표현하는 것은 대체로 좋지 않은 생각이다.

``` java
String compoundKey = className + "#" + i.next();
```
- 두 요소를 구분해주는 문자 `#`이 두 요소 중 하나에서 쓰였다면 혼란스러운 결과를 초래한다.
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 귀찮고, 오류 가능성도 커진다.
- 적절한 `equals`, `toString`, `compareTo` 메서드를 제공할 수 없으며, `String`이 제공하는 기능에만 의존해야 한다.

해결 방법은, 전용 클래스를 `private` 정적 멤버 클래스로 새로 만들면 된다.[아이템 24](https://github.com/pyh-dotcom/Effective-Java-Study/blob/main/Chap4/item24.md)

``` java
public class A {
	private class B {
    ...
    }
}
```

___
## 문자열은 권한을 표현하기에 적합하지 않다

### 문자열로 권한 부여

권한(capacity)을 문자열로 표현하는 경우가 종종 있다. 예를 들어 스레드 지역변수 기능을 설계한다고 해보자.

```java
public class ThreadLocal {

    //객체 생성 불가
    private ThreadLocal() {
    }

    // 현 스레드의 값을 키로 구분해 저장
    public static void set(String key, Object value) {
    }

    //키가 가르키는 현 스레드의 값을 반환한다.
    public static Object get(String key) {
        return null;
    }
}
```
위에 코드는 문자열 `key`를 통해 권한(capacity)을 구분하였다. 
- `key`가 전역공간에 저장되고 공유가 되는데, `key` 생성 로직에 문제가 있거나 다른 스레드에서 같은 키를 생성하면 같은 변수를 공유한다.

- 보안에도 취약한데 의도적으로 같은 키를 사용하여 다른 클라이언트의 값을 가지고 올수도 있다.

### key 클래스로 권한 부여

위 `ThreadLocal` 클래스는 문자열 대신 위조할 수 없는 키를 사용하면 해결된다.

``` java
public class ThreadLocal {
	private ThreadLocal() { }
    
    public static class Key {
    	Key() { }
    }
   
    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
   	   return new Key();
    }
   
    public static void set(Key key, Object value);
    public static Object get(Key key);
```
`set` 과 `get` 은 정적 메서드일 이유가 없어지니 인스턴스 메서드로 바꿔도 된다.
이렇게 되면 `Key` 는 더 이상 스레드 지역변수를 구분하기 위한 키가 아니라, 그 자체가 스레드 지역변수가 된다. 
결과적으로 지금의 톱레벨 클래스인 `ThreadLocal`은 별달리 하는 일이 없어지므로 치워버리고, 중첩 클래스 `Key`의 이름을 `ThreadLocal`로 바꿔버리자.

### Key를 ThreadLocal로 변경 후 매개변수화

```java
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```
`get`으로 얻은 `Object`를 실제 타입으로 형변환해 써야 해서 타입 안전하지 않다.

`ThreadLocal`을 매개변수화 타입으로 선언하면 간단하게 문제가 해결된다.[아이템 29](https://github.com/pyh-dotcom/Effective-Java-Study/blob/main/Chap5/item29.md)

``` java
public final class ThreadLocal<T> {
	public ThreadLocal();
    public void set(T value);
    public T get();
}
```
이 코드는 `java.lang.ThreadLocal`과 비슷하다.

___

## 정리
더 적합한 데이터 타입이 있거나 새로 작성할 수 있다면, 문자열은 쓰지 말자. 문자열은 잘못 사용하면 번거롭고, 덜 유연하며 느리고 오류 가능성도 크다.
