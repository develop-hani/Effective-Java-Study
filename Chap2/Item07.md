# 1. 메모리 누수의 주범

### 1) 스택(Stack)과 같이 자기 자신의 자원을 관리하는 클래스

- 스택을 사용하는 프로그램을 오래 실행하다 보면 점차 GC의 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다.

- 아래 코드에서 성능이 저하되는 위치는 어디일까?
    
    ```java
    public class Stack {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
        public Stack() {
            elements = new Object[DEFAULT_INITIAL_CAPACITY];
        }
    
        **public Object pop()** { // 스택이 줄어들 때 꺼내진 객체를 GC가 회수하지 않는다.
            if (size == 0)
                throw new EmptyStackException();
            return elements[--size];
        }
    		...
    }
    ```
    
    스택이 객체들의 다 쓴 참조(obsolete reference)를 가지고 있어 **GC가 스택에서 꺼내진 객체들을 회수하지 않는다**.
    
- 해당 참조를 다 썼을 때 **null 처리(참조 해제)** 를 통해 해결할 수 있다.
    
    ```java
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        **elements[size] = null;** // 다 쓴 참조 해제
        return result;
    }
    ```
    
    `null` 처리된 참조를 사용하려 한다면 `NullPointerException`을 던지며 종료한다.
    
    ⇒ 프로그램의 오류를 조기에 발견할 수 있다.
    

#### **객체 참조를 `null` 처리하는 일은 예외적이어야 한다.**

- 모든 객체를 일일이 null처리하는 것은 코드를 필요 이상으로 지저분해지게 한다.

- **참조를 담은 변수를 유효 범위 밖으로 밀어내는 방법**을 사용하는 것이 가장 효과적이다.

#### Stack이 메모리 누수에 취약한 이유

- 스택이 **자기 메모리를 직접 관리**하기 때문이다.
  
    객체 자체가 아니라 객체 참조를 담는 elements 배열로 저장소 풀을 만들어 원소를 관리한다.
    
    ⇒ 활성 영역에 속한 원소 사용하고 비활성 영역의 원소는 사용하지 않지만 가비지 컬렉터는 이를 모른다.
    
    ⇒ **프로그래머가 직접 비활성 영역이 되는 순간 직접 null 처리**를 하여 GC에 해당 객체를 더 이상 사용하지 않는다는 것을 알려야 한다.
</br>    

### 2) 캐시(Cache)

- 캐시에 담아두고 사용하지 않는 경우 캐시는 캐시의 역할을 하지 못하게 된다.

    (캐시: 데이터 값을 미리 복사해 두는 임시장소로 자주 사용하거나 접근하는 시간이 오래 걸리는 경우 시간을 절약하기 위해 사용한다.)
    
- **WeakHashMap을 통해 해결**할 수 있다.
    
    WeakHashpMap은 특정 key가 더이상 사용되지 않는다고 판단되면 해다 Key-Value 쌍을 삭제한다.
</br>    

### 3) 리스너(listener) 혹은 콜백(callback)

- 리스너와 콜백을 등록만 하고 해지하지 않는 상황에서 메모리 낭비가 발생한다.

- 콜백을 **약한 참조(weak reference)로 저장하여 가비지 컬렉터가 즉시 수거해 갈 수 있도록** 한다.

> ※ 콜백(callback)
> 
> - Calle Side에서 Caller Side 위에 작동하는 메서드를 호출하는 것을 말한다.
> - Calle가 콜백 인터페이스 및 메서드 선언
>     
>     → Caller가 콜백 인터페이스 구현 및 Calle 객체에 등록
>     
>     → Calle가 호출
>     
