# 26. 로 타입은 사용하지 말라

## 용어 정리

- 제네릭(generic)
    
    타입 매개변수를 사용하여 선언한 클래스와 인터페이스
    
    e.g.) List<E>
    
- 매개변수화 타입(parameterized type)
    
    e.g.) List<String>
    
- 실제(actual) 타입 매개변수
    
    e.g.) List<String>의 String
    
- 정규(formal) 타입 매개변수
    
    e.g.) List<E>의 E
    
- 로 타입(raw type)
    
    제네릭 타입에서 타입 매개 변수를 사용하지 않은 것
    
    e.g.) List
    

---

## Generic을 사용했을 때 이점

1. **안정성**
    
    타입 오류를 컴파일 시점에 발견할 수 있다.
    
    ```java
    private final Collection<Stamp> stamps = ...;
    stamps.add(new Coin(...)); // 컴파일 에러 발생
    ```
    
2. **표현력**
    
    타입 체크와 형변환을 생략하여 코드가 간결해진다.
    

---

## Raw Type

### 존재 이유

**호환성**

자바가 제네릭을 받아들이기 이전의 코드를 수용하기 위한 방법이다.

### 문제점

**제네릭의 장점인 안정성과 표현성을 모두 잃는다.**

오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 떄 발견하는 것이 좋다.

그러나 로 타입을 사용할 경우, 오류가 발생하고 한참 뒤인 런타임에야 알아챌 수 있다.

```java
// 안정성 측명
private final Collection stamps = ...;
stamps.add(new Coin(...)); // 아무 오류 없이 컴파일 되고 실행된다.

// 표현성 측면 - 제네릭을 사용할 경우 필요 없다.
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
	Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
	stamp.cancel();
}
```

---

## Raw 타입 개선 방법

1. **임의 객체를 허용하는 매개변수화 타입**
    
    List<Object>는 모든 타입을 허용한다는 의미를 컴파일러에 명확히 전달한다.
    
    타입 에러가 있을 때, 컴파일 타임에 오류가 발생했음을 알 수 있다.
    
    ```java
    // 1) Raw 타입 List를 사용한 경우
    public class Raw {
        public static void main(String[] args) {
            List<String> strings = new ArrayList<>();
            unsafeAdd(strings, Integer.valueOf(42));
            String s = strings.get(0); // 컴파일러가 자동으로 형변환 코드를 넣어준다.
        }
    	
        private static void unsafeAdd(List list, Object o) {
            list.add(o); // ClassCastException 발생
        }
    
    		// 2) 
    		private static void unsafeAdd(List<Object> list, Object o) {
            list.add(o);
        }
    }
    ```
    
    ```java
    // 2) 매개변수화 타입을 사용한 경우
    public class Raw {
        public static void main(String[] args) {
            List<String> strings = new ArrayList<>();
            unsafeAdd(strings, Integer.valueOf(42)); // 컴파일 에러 발생
            String s = strings.get(0);
        }
    
    		private static void unsafeAdd(List<Object> list, Object o) {
            list.add(o);
        }
    }
    ```
    
2. **비한정적 와일드카드 타입(unbounded wildcard type)**
    
    제네릭 타입을 사용하고 싶지만 실제 매개변수를 신경쓰고 싶지 않을 때, 물음표(?)를 사용하는 방법이다.
    
    e.g.)  List<?>
    
    ```java
    // 1) 모르는 타입도 받는 로 타입을 사용한 경우
    public int numElementsInCommon(Set s1, Set s2) {
        int result = 0;
        for (Object o1 : s1)
            if (s2.contains(o1))
                result++;
        return result;
    }
    
    // 이 경우, 아무 원소나 넣을 수 있으므로 타입 불변식을 훼손하기 쉽고 안전하지 않다.
    ```
    
    ```java
    // 2) 비한정적 와일드카드 타입을 사용한 경우
    public int numElementsInCommon(Set<?> s1, Set<?> s2) {
        int result = 0;
        for (Object o1 : s1)
            if (s2.contains(o1))
                result++;
        return result;
    }
    
    // Collection<?>에는 null 외엔 어떤 원소도 넣을 수 없으므로 컬렉션의 타입 불변식을 훼손하지 못한다
    ```
    

---

## Raw 타입을 사용하는 예외 상황

1. **class 리터럴에서는 로 타입을 써야 한다**
    
    자바 명세에서 class 리터럴에 매개변수화 타입을 사용하지 못하게 했다(배열과 기본 타입은 허용)
    
    List.class, String[].class, int.class는 허용하지만, List<String>.class, List<?>.class는 허용하지 않는다.
    
2. **instanceof 연산자**
    
    런타임에는 제네릭 타입의 정보가 지워진다.
    
    ⇒ 로 타입과 비한정적 와일드 카드의 `instanceof`는 차이가 없다.
    
    차라리 로 타입을 쓰는 것이 깔끔하다.
    
    ```java
    if (o instanceof Set) {         // 로타입
        Set<?> s = (Set<?>) o;      // 와일드카드 타입
        ...
    }
    ```
