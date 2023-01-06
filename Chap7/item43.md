# 람다보다는 메서드 참조를 사용하라

## 메서드 참조(method reference)

- 람다보다 함수 객체를 더 간결하게 만들 수 있다.
- e.g.) 임의의 키와 Integer 값의 매핑을 관리하는 프로그램
    
    ```java
    Map<String, Integer> frequencyTable = new TreeMap<>();
    ```
    
    1. 람다
        
        ```java
        for (String s : args)
                    frequencyTable.merge(s, 1, (count, incr) -> count + incr);
        ```
        
        - 매개변수 `count`와 `incr`가 불필요하게 많은 공간을 차지한다.
    2. 메서드 
        
        ```java
        for (String s : args)
                    frequencyTable.merge(s, 1, Integer::sum);
        ```
        
        - 훨씬 간결한 코드를 확인할 수 있다.

### 5가지 유형

1. **정적 메서드를 가리키는** 메서드 참조
    - 람다로 구현했을 때 더 간결한 경우가 있다.
        
        다음 코드는 `GoshThisClassNameIsHumongous` 내부에 구현되어 있다.
        
        메서드 참조) `service.excute(GoshThisClassNameIsHumongous::action);`
        
        람다) `service.execute(() -> action());`
        
    - 같은 맥락에서 java.util.function 패키지가 제공하는 제네릭 정적 팩터리 메서드인 Function.identity()를 사용하기보다는 똑같은 기능의 람다(x -> x)를 사용하는 편이 짧고 명확하다.
    - 메서드 참조) `Integer::parseInt`
        
        람다) `str -> Integer.parseInt(str)`
        
2. **수신 객체(receiving object; 참조 대상 인스턴스)를 특정하는** 한정적(bound) 인스턴스 메서드 참조
    - 함수 객체가 받는 인수 == 참조되는 메서드가 받는 인수
    - 정적 참조와 유사하다.
    - 메서드 참조) `Instant.now()::isAfter`
        
        람다) `str->Integer.parseInt(str)`
        
3. **수신 객체를 특정하지 않는 비한정적(unbound)** 인스턴스 메서드 참조
    - 함수 객체를 적용하는 시점에 수신 객체를 알려준다.
    - 사용) 스트림 파이프라인에서의 매핑과 필터 함수
    - 메서드 참조) `Integer::parseInt`
        
        람다) `Instant then = Instant.now();` `t->then.isAfter(t)`
        
4. **클래스의 생성자를 가리키는** 메서드 참조 / 5. **배열 생성자를 가리키는** 메서드 참조
    - 생성자 참조는 팩터리 객체로 사용된다.
    - 4번 예시
        
        메서드 참조) `TreeMap<K,V>::new`
        
        람다) `() -> new TreeMap<K,V>()`
        
    - 5번 예시
        
        메서드 참조) `int[]::new`
        
        람다) `len -> new int[len]`
        

---

## 메서드 참조보다 람다가 더 유용한 경우

- 람다로 할 수 없는 일이라면 메서드 참조로도 불가하다. (제너릭 함수 타입에는 예외가 있다.)
- 람다의 **매개변수 이름** 자체가 **프로그래머에게 좋은 가이드**가 되기도 한다.
    
    매개변수 수가 늘어날수록 매서드 참조로 제거할 수 있는 코드양이 늘어난다.
    
- 람다가 메서드 참조보다 **간결**할 때가 있다.
    
    주로 메서드와 람다가 같은 클래스에 있을 때 그렇다.
    

---

## 결론

- 메서드 참조는 람다의 간단명료한 대안이 될 수 있다.
- 메서드 참조 쪽이 짧고 명확하다면 메서드 참조를 쓰고, 그렇지 않을 때만 람다를 사용하라.
