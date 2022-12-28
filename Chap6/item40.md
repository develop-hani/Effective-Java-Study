# 40. @Override 애너테이션을 일관되게 사용하라

### 선요약

- 재정의한 모든 메서드에 `@Override`를 다는것이 좋다.
- annotation을 달지 않아도 되는 예외 상황은 있지만 달아서 해로울 것은 없다.
    
    구체클래스인데 아직 구현하지 않은 추상 메서드가 있다면 컴파일러가 알려준다.
    

---

## @Override란,

- 상위 타입의 메서드를 재정의할 때 사용
- 악명 높은 버그들 예방에 도움

---

## 아래 코드의 문제점

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first  = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

- Set는 중복을 허용하지 않아 26이 출력될 것 같지만 실제론 260이 출력된다.
    
    `equals`를 새로 정의하였기 때문이다.
    
    객체 식별성만을 확인하여 10개의 바이그램을 다른 객체로 인식한다.
    
- `@Overide`를 명시하여 해결할 수 있다.
    
    ```java
    @Override public boolean equals(Object o) {
            if (!(o instanceof Bigram2))
                return false;
            Bigram2 b = (Bigram2) o;
            return b.first == first && b.second == second;
        }
    ```
