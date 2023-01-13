# null이 아닌, 빈 컬렉션이나 배열을 반환하라

null을 반환하는 메서드를 호출하는 경우는 항시 방어 코드를 넣어줘야 한다.<br>
null을 반환하려면 반환하는 쪽에서도 이 상황을 특별히 취급해줘야 해서 코드가 더 복잡해진다.

## null보다는 빈 컨테이너를 반환하자.


```java
// 빈 컬렉션을 반환하는 예
public List<Cheese> getCheeses() {
  return new ArrayList<>(cheesesInStock);
}
```

빈 컨테이너를 할당하는데 비용이 들어도 null 대신에 빈 컨테이너를 사용하자.

- null 보다는 빈 컨테이너를 사용해야하는 이유
1. 성능 분석 결과 빈 컨테이너가 성능 저하의 주범이라고 확인하지 않는 한, 이 정도의 성능 차이는 신경 쓸 수준이 못 된다.
2. 빈 컬렉션과 배열은 굳이 새로 할당하지 않고도 반환할 수 있다.

### 빈 컬렉션을 매번 반환하지 않게 하는 경우

매번 똑같은 빈 '불변' 컬렉션을 반환한다. 불변 객체는 자유롭게 공유해도 안전하다.

```java
// 최적화 - 빈 컬렉션을 매번 새로 할당하지 않는 경우
public List<Cheese> getCheeses () {
  return cheesesInStock.isEmpty() ? 
    Collections.emptyList() : new ArrayList<>(cheesesInStock);
}

```

Collections.emptyList 메서드를 사용해서 똑같은 빈 '불변' 컬렉션을 반환할 수 있다.<br>
필요하면 Collections.emptySet, Collections.emptyMap을 사용할 수도 있다.<br>
단 실제로 성능을 확인해 보고 성능이 개선되는 경우에만 사용하자.

### 배열의 경우

배열에서도 null 대신에 길이가 0인 배열을 반환하자.

```java
public Cheese[] getCheeses() {
  return cheesesInStock.toArray(new Cheese[0]);
}
```
toArray 메서드는 인자의 배열이 충분히 크면 인자에 원소를 담아 리턴하고, 아니면 새로운 배열을 만들어 반환한다.<br>
&rarr; cheesesInStock의 길이가 0인 경우에는 0짜리 배열을 반환하고, 아닌 경우에는 chesesInStock 원소만큼의 배열로 반환한다.

이 경우도 빈 배열을 매번 할당하므로 미리 선언해두고 그 배열을 반환할 수도 있다.

```java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

public Cheese[] getCheeses() {
  return CheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
}
```
- 이 경우도 cheesesInStock이 비었을 때면 언제나 EMPTY_CHEESE_ARRAY를 반환하게 된다.<br>
- 단순히 성능을 개선할 목적이라면 toArray에 넘기는 배열을 미리 할당하는 것은 추천하지 않는다.<br>
&rarr; 오히려 성능이 떨어진다는 연구 결과도 있다.
