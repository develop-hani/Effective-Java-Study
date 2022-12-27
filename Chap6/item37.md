# ordinal 인덱싱 대신 EnumMap을 사용하라

```java
class Plant {
  enum LifeCycle {ANNUAL, PERENNIAL, BIENNIAL}
    
  final String name;
  final LifeCycle lifeCycle;
    
  Plant(String name, LifeCycle lifecycle) {
  this.name = name;
  this.lifeCycle = lifeCycle;
  }
     
  @Override
  public String toString() {
    return name;
  }
}
```

식물들을 생애주기 별로 묶어 배열 하나로 관리한다고 하자.

## ordinal 값을 그 배열의 인덱스로 사용하는 경우

```java
Set<Plant>[] plantsByLifeCycle = 
(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];
for (int i = 0; i < plantsByLifeCycle.length; i++)
  plantsByLifeCycle[i] = new HashSet<>();

for (Plant p : garden)
  plantsByLifeCycle[p.lifeCycle.ordinal()].add(p);
    
// 결과 출력
for (int i = 0; i < plantsByLifeCycle.length; i++) {
  System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
}
```

### ordinal을 배열의 인덱스로 사용하는 경우 문제점
- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 해야한다.
	&rarr; 깔끔히 컴파일 되지 않는다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 함. 
- 정확한 정숫값을 사용한다는 것을 보증해야 함.
&rarr; 열거 타입의 정수값은 안전하지 않음.
---

## EnumMap으로 문제 해결

```java
Map<Plant.LifeCycle, Set<Plant>> plantsByLifeCycle = new EnumMap<>(Plant.LifeCycle.class);
for (Plant.LifeCycle lc : Plant.LifeCycle.values())
  plantsByLifeCycle.put(lc, new HashSet<>());
for (Plnat p : garden)
  plantsByLifeCycle.get(p.lifeCycle).add(p);
System.out.println(plantsByLifeCycle);
```
### EnumMap 장점

- EnumMap은 열거 타입을 키로 사용한다.
- 위의 경우보다 휠씬 안전하고 성능도 원래 버전과 비슷
- 키가 열거 타입 자체라서 출력 결과에 직접 레이블을 달 필요 없다.
- 인덱스 오류가 날 가능성도 없어진다.

EnumMap 내부에서 배열 사용을 사용한다.<br>
&rarr; 내부 구현 방식을 안으로 숨겨 Map의 타입 안정성과 배열의 성능 이점<br>
EnumMap 생성자가 받는 키 타입의 Class 객체는 한정적 타입 토큰이다.

---

## 두 열거 타입 값을 매핑하는 경우

### 배열들의 배열의 인덱스에 ordinal()을 사용 - 따라 하지 말 것!
```java
public enum Phase {
  SOLID, LIQUID, GAS;
}

public enum Transition {
  MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;
    
  // 행은 from의 ordinal을, 열은 to의 ordinal을 인덱스로 쓴다.
  private static final Transition[][] TRANSITIONS = {
    {null, MELT, SUBLIME},
    {FREEZE, null, BOIL},
    {DEPOSIT, CONDENSE, null}
  };  
}
```
### 문제점 
- 배열 인덱스 관계를 알지 못하므로 잘못 수정하거나 두 열거타입을 함께 수정하지 않으면 오류가 발생할 가능성이 있다.
- 가짓수가 늘어나면 null로 채워지는 칸도 늘어나 공간이 낭비된다.

### EnumMap을 사용하는 경우

```java
public enum Phase {
  SOLID, LIQUID, GAS;
    
  public enum Transition {
    MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
    BOIL(LIQUID, GAS), CONDENSE(GAS, LIQUID),
    SUBLIME(SOLID, GAS) DEPOSIT(GAS, SOLID);
 		
    private final Phase from;
    private final Phase to;
        
    Transition(Phase from, Phase to) {
      this.from = from;
      this.to = to;
    }
        
    // 상전이 맵을 초기화한다.
    private static final Map<Phase, Map<Phase, Transition>>
    m = Stream.of(values()).collect(groupingBy(t -> t.from,
    () -> new EnumMap<>(Phase.class),
    toMap(t -> t.to, t -> t, (x, y) -> y, () -> new EnumMap<>(Phase.class))));
        
    public static Transition from(Phase from, Phase  to) {
      return m.get(from).get(to);
    }
  }
}
	
```

EnumMap을 사용하면 추가할 때도 간단하고 낭비되는 공간도 적다.<br>
맵의 내부의 배열을 사용해 내부 로직으로 처리하므로 안전하고 유지보수하기 좋다.
