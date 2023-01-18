# 전통적인 for문보다는 for-each 문을 사용하라

## 컬렉션과 배열을 순회할 때 for문의 단점

컬렉션과 배열을 순회하는 방법은 while문 보다는 for문이 낫지만 여기에도 단점이 있다.

- for 문의 단점
	- 반복자와 인덱스 변수들은 지저분할 뿐 우리가 필요한 건 원소이다.
	- 이러한 코드는 오류가 날 가능성이 높아진다.
	- 컬렉션이나 배열이냐에 따라 코드가 달라진다.
    
&rarr; for-each문으로 해결 가능


## for-each문

- 반복자와 인덱스 변수를 사용하지 않는다.
- 어떤 컨테이너를 다루는지 신경 쓰지 않아도 된다.
- 컬렉션을 중첩해야 하는 경우에 이점이 더욱 커진다.

```java

enum Suit {CLUB, DIAMOND, HEART, SPADE}
enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, QUEEN, KING }

static Collection<Suit> suits = Arrays.asList(Suit.values());
static Collection<Rank> ranks = Arrays.asList(Rank.values());

for (Suit suit : suits)
  for (Rank rank : ranks)
    deck.add(new Card(suit, rank);
```

## for-each문을 사용할 수 없는 경우

- **파괴적인 필터링**: 컬렉션을 순회하면서 선택된 원소를 제거해야 해서 반복자의 remove 메서드를 호출해야 하는 경우<br>
&rarr; removeIf 메서드를 사용해 켈렉션을 명시적으로 순회하는 일을 피할 수 있다.

- **변형**: 리스트나 배열을 순회하면서 그 원소의 값 혹은 전체를 교체해야 하는 경우에는 반복자나 배열의 인덱스를 사용해야 한다.

- **병렬 반복**: 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수를 사용해 엄격하고 명시적으로 제어해야 한다.

## Iterable

Iterable 인터페이스를 구현한 객체라면 무엇이든 순회할 수 있다.

```java
public interface Iterable<E> {
  // 이 객체의 원소들을 순회하는 반복자를 반환한다.
  Iterator<E> iterator();
}
```

Iterable 인터페이스를 구현하기는 어렵지만 원소들의 묶음을 표현하는 타입을 작성해야 한다면, Iterable을 구현하는 쪽을 고민해 봐라.<br>
그 타입을 사용하는 프로그래머들이 편할 것이다.
