옵셔널 반환은 신중히 하라
=
### 메서드가 특정 조건에서 값을 반환할 수 없을 때 취할수 있는 선택지
1. 예외를 던지기\
-> 진짜 예외 적인 상황에만 사용해야 하며 예외를 생성할 때 스택 추적 전체를 캡처하기 때문에 비용이 크다.\
2. 반환 타입이 객체 참조라면 null을 반환하기\
-> null을 반환 할 수 있는 메서드를 호출할때는 null 처리 코드를 별도로 추가해야한다.\
3. 자바 8에서 추가된 Optinal< T > 반환
## Optinal< T >
- null이 아닌 T 타입 참조를 하나 담거나, 혹은 아무것도 담지 않을 수 있다.
- 옵셔널을 반환하는 메서드는 예외를 던지는 메서드보다 유연하고 사용하기 쉬우며, null을 반환하는 메서드보다 오류 가능성이 적다.
아래는 아이템 30의 max 코드를 옵셔널을 반환하도록 수정한 메서드이다.

<details>
<summary>아이템 30의 max 코드</summary>
<div markdown="1">
  
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("빈 컬렉션");

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return result;
}
```
 
</div>
</details>

```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    if (c.isEmpty())
        return Optional.empty();

    E result = null;
    for (E e : c)
        if (result == null || e.compareTo(result) > 0)
            result = Objects.requireNonNull(e);

    return Optional.of(result);
}
```
빈 옵셔널은 Optional.empty()로 만들고, 값이 든 옵셔널은 Optional.of(value)로 생성했다.\
이때 Optional.of(value)에 null을 넣으면 NullPointerException을 던진다. \
옵셔널을 반환하는 메서드에서 null을 반환하는것은 옵셔널을 도입한 취지를 무시하는 일이므로 하지말자
## 옵셔널과 스트림
스트림의 종단 연산중 상당수가 옵셔널을 반환한다.\
아래는 위 max 코드를 스트림버전으로 다시 작성한 것이다.
```java
public static <E extends Comparable<E>>
Optional<E> max(Collection<E> c) {
    return c.stream().max(Comparator.naturalOrder()); //max 연산이 우리에게 필요한 옵셔널을 생성
}
```
### null반환, 예외 대신 옵셔널을 반환해야할 때
결과가 없을 수 있으며, 클라이언트가 이 상황을 특별하게 처리해야 할때 
### 메서드가 옵셔널을 반활할때 값을 받지 못했을때 클라이언트가 취할 수 있는 행동
1. 기본값 설정(생성 비용이 부담이 될 경우 `Supplier<T>`를 이용해 필요할 때 생성하도록 할 수 있다.)
2. 상황에 맞는 예외 던지기
### 스트림에서 특별한 메서드들
- filter : 스트림 요소를 순회하면서 특정 조건을 만족하는 요소로 구성된 새로운 스트림을 반환
- map : 일 스트림의 원소를 매핑시킨 후 매핑시킨 값을 다시 스트림으로 변환
- flatMap : map은 입력한 원소를 그대로 스트림으로 반환하지만 flatMap은 입력한 원소를 가장 작은 단위의 단일 스트림으로 반환합
- isPresent : 옵셔널이 채워져 있으면 true, 비어있으면 false 반환
- stream : 옵셔널을 스트림으로 변환
### 옵셔널 주의사항
- 컬렉션, 스트림, 배열, 옵셔널 같은 컨테이너 타입은 옵셔널로 감싸면 안된다.
- 새로 객체를 할당하고 초기화 해야하고, 메서드를 통해 값을 꺼내는 단계가 추가로 필요하므로 성능이 중요한 상황에는 맞지 않을 수 있다.
- 박싱된 기본 타입은 기본 타입 자체보다 무거워 이를 담아 반환하는 대신 전용 옵셔널 클래스를 사용하는것이 좋다.
- 옵셔널을 컬렉션의 키, 값, 원소나 배열의 원소로 사용하는게 적적한 경우는 거의 없으므로 반환 이외의 용도로 사용하는 것은 지양하는것이 좋다.
