# 50. 적시에 방어적 복사본을 만들라

## ⭐ 선요약

클래스가 클라이언트로 받는 혹은 클라이언트로 반환하는 구성요소가 가변이라면 그 요소는 반드시 방어적으로 복사한다.

### 자바의 안정성

자바는 안정적으로 설계된 언어이지만, 클라이언트가 불변식을 깨뜨리려 혈안이 되어있다고 가정하고 방어적으로 프로그래밍해야 한다.

---

### 불변식을 지키지 못하는 클래스

```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param  start 시작 시각
     * @param  end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + "가 " + end + "보다 늦다.");
        this.start = start;
        this.end   = end;
    }

    public Date start() {
        return start;
    }
    public Date end() {
        return end;
    }

    public String toString() {
        return start + " - " + end;
    }
```

가변인 `Date`를 사용하였다. 자바 8 이후에 등장한 불변 `Instant`를 사용하여 해결할 수 있다.

---

### 방어적 복사본

1. **생성자에서 받은 기본 매개변수 각각을 방어적으로 복사**하여 외부 공격으로부터 내부를 보호할 수 있다.
    
    ```java
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        
        if (this.start.compareTo(this.end) > ) 
            throw new IllegalArgumentException(this.start + " after " + this.end);
    }
    ```
    
    매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용하면 안 된다
    
2. **접근자가 방어적 복사본을 반환**하도록 한다.
    
    ```java
    public Date start() {
        return new Date(start.getTime());
    }
    
    public Date end() {
        return new Date(end.getTime());
    }
    ```
