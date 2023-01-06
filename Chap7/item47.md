# 반환 타입으로는 스트림보다 컬렉션이 낫다.

- 자바 7 이전에는 원소 시퀀스를 반환하는 메서드에 반환 타입으로 Collection, Set, List, Iterable, 배열 등을 썼다.<br>
하지만 자바 8부터 스트림이라는 개념이 나오면서 반환 타입 선택이 복잡해졌다.

- 스트림은 반복을 지원하지 않아 스트림과 반복을 알맞게 조합해야 좋은 코드가 나온다.

- Stream 인터페이스는 Iterable 인터페이스가 정의한 추상 메서드를 전부 포함할 뿐 아니라, Iterable 인터페이스가 정의한 방식대로 동작한다.<br> 
**그러나 Stream이 Iterable을 확장하지 않았다.**

이런 문제를 해결하기 위해서는 어댑터 메서드가 필요하다.

```java
//Stream<E>를 Iterable<E>로 중계해주는 어댑터
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
  return stream::iterator;
}

for (ProcessHandle p : iterableOf(ProcessHandle.allProcesses())) {
  //프로세스를 처리한다.
}
```

반대로 API가 Iterable만 반환하는데 파이프라인에서 사용해야 한다면 Stream으로 바꿔주는 어댑터를 사용한다.

```java
//Iterable<E>를 Stream<E>로 중계해주는 어댑터
public static <E> Stream<E> streamOf(Iterable<E> iterable) {
  return StreamSupport.streamn(iterable.spliterator(), false);
}
```
--- 
## 반환타입 Collection의 장점

- 공개 API를 작성 할 때는 스트림 파이프라인을 사용하는 사람과 반복문에서 쓰려는 사람 모두를 배려해야 한다.
- Collection 인터페이스는 Iterable의 하위 타입이고 stream 메서드도 제공하니 반복과 스트림을 동시에 지원한다.

따라서 **원소 시퀀스를 반환하는 공개 API의 반환 타입에는 Collection이나 그 하위 타입을 쓰는게 일반적으로 최선이다.**

--- 
## 덩치 큰 시퀀스를 반환하는 경우

반환하는 시퀀스의 크기가 메모리에 올려도 안전할 만큼 작다면 표준 컬렉션 구현체를 반환하는게 최선일 수 있다.

하지만 **단지 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.**<br>
&rarr; 전용 컬렉션을 구현하는 방안을 검토하자.

### 전용 컬렉션을 구현하는 예

AbstartList를 이용해서 전용 컬렉션을 구현하여 멱집합을 반환하는 예이다.

```java
public class PowerSet {
  public static final <E> Colelction<Set<E>> of(Set<E> s) {
    List<E> src = new ArrayList<>(s);
    if (src.size() > 30)
      throw new IllegalArgumentException(
        "집합에 원소가 너무 많습니다(최대 30개).:" + s);
        
    return new AbstractList<Set<E>>() {
      @Override
      public int size() {
        //멱집합의 크기는 2를 원래 집합의 원소 수만큼 거듭제곱 한 것과 같다.
        return 1 << src.size();
      }
            
      @Override
      public boolean contains(Object o) {
        return o instanceof Set && src.containsAll((Set)o);
      }
        
      @Override
      public Set<E> get(int index) {
        Set<E> result = new HashSet<>();
        for (int i = 0; index != 0; i++, index >>= 1)
          if ((index & 1) == 1)
            result.add(src.get(i));
          return result;
      }
    };
  }
}
```

- AbstracionCollection을 활용해서 Collection 구현체를 작성할 때는 Iterable용 메서드 외에 contains 와 size만 구현하면 된다.
- contains와 size를 구현하는게 불가능 할 때는 컬렉션 보다는 스트림이나 iterable을 반환하는 편이 낫다.
원한다면 별도의 메서드를 두어 두 방식을 모두 제공해도 된다.

---

## 쉬운 쪽을 선택하는 경우

연속하는 부분 리스트를 모두 반환하는 메서드를 작성해야 하는 경우, 전용 컬렉션을 만드는 것은 번거로운 일이다.<br>
그러나 약간의 통찰만 있으면 스트림으로 쉽게 작성할 수 있다.

```java 
public class SubLists {
  public static <E> Stream<List<E>> of(List<E> list) {
    return Stream.concat(Stream.of(Collections.emptyList()),
      prefixes(list).flatMap(SubLists::suffixes));
  }
    
  public static <E> Stream<List<E>> prefixes(List<E> list) {
    return IntStream.rangedClosed(1, list.size())
      .mapToObj(end -> list.subList(0, end));
  }

  public static <E> Stream<List<E>> suffixes(List<E> list) {
    return IntStream.range(0m list.size())
      .mapToObj(start -> list.subList(start, list.size()));
  }
}
```

- Stream.concat 메서드는 빈 리스트를 추가하고, flatMap 메서드는 모든 프리픽스의 모든 서픽스로 구성된 하나의 스트림을 만든다. 프릭스들과 서픽스들은 range와 rangeClosed로 매핑하였다.

- for 반복문을 중첩한 코드를 스트림으로 바꿀 수도 있다.

---
## 어댑터와 전용 Collection 속도

- Stream-Iterable 어댑터를 사용하면 코드를 어수선하게 하고 느려진다.
- 전용 Collection을 사용하면 코드는 지저분해지지만 속도는 빨라진다.
