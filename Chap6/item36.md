# 비트 필드 대신 EnumSet을 사용하라

## 비트 필드를 사용하면 좋지 않은 이유
열거한 값들이 주로 집합으로 사용될 경우, 예전에는 각 상수에 서로 다른 2의 거듭제곱 값을 할당한 정수 열거 패턴을 사용해 왔다.

```java
public class Text {
  public static final int STYLE_BOLD          = 1 << 0;	// 1
  public static final int STYLE_ITALIC        = 1 << 1;	// 2
  public static final int STYLE_UNDERLINE     = 1 << 2;	// 4
  public static final int STYLE_STRIKETHROUGH = 1 << 3;	// 8
    
  // 매개변수 styles는 0개 이상의 STYLE_ 상수를 비트별 OR한 값이다.
  public void appyStyles(int styles) {...}
}

// 비트별 연산을 사용해 열거값들을 집합으로 표현한 경우
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

비트 필드를 사용하면 비트별 연산을 사용해 집합 연산을 효율적으로 수행할 수 있지만, 다음 문제들이 발생한다.

### 비트 필드 사용시 문제점
1. 비트 필드 값이 그대로 출력되면 단순한 정수 열거 상수를 출력할 때보다 해석하기가 훨씬 어렵다.
2. 비트 필드 하나에 녹아 있는 모든 원소를 순회하기 까다롭다
3. 최대 몇 비트가 필요한지를 API 작성 시 미리 예측하여 적절한 타입을 선택해야 한다.

---
## 더 나은 대안 EnumSet

### EnumSet의 장점

- Set 인터페이스를 완벽히 구현
- 타입 안전
- 어떤 Set 구현체와도 함께 사용 가능

EnumSet 내부는 비트 벡터로 구현 되어 있다. **비트를 직접 다루지 않고 내부 구현을 사용하므로 난해한 문제나 흔히 겪는 오류에서 해결해 준다.**<br>
원소의 개수가 64개 이하라면, 비트 필드에 비견되는 성능을 보여준다.

```java
public class Text { 
  public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
    
  // 어떤 Set을 넘겨도 되나, EnumSet이 가장 좋다.
  public void applyStyles(Set<Style> styles) {...}
}

// EnumSet을 사용한 경우
text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));
```

EnumSet만을 받을 것이라고 예상되는 메서드라도 인터페이스로 받는 것이 좋은 습관이다.
