## 아이템45. 스트림은 주의해서 사용하라

### 스트림이란?

스트림은 데이터 컬렉션 반복을 처리하는 기능으로, 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.

### 수정 전
``` java
List<Dish> lowCaloricDishes = new ArrayList<>();
for (Dish dish : menu) { // (1) 400 칼로리 이하의 요리 선택
  if(dish.getCalories() < 400) {
    lowCaloricDishes.add(dish);
  }
}

Collections.sort(lowCaloricDishes, new Comparator<Dish>() {
  pubic int compare(Dish dish1, Dish dish2) { // (2) 정렬
    return Integer.compare(dish1.getCalories(), dish2.getCalories());
  }
});

List<String> lowCaloricDishesName = new ArrayList<>();
for(Dish dish : lowCaloricDishes) { // (3) getName 메소드를 참조하여 요리명 추출, (4) 리스트에 저장
  lowCaloricDishesName.add(dish.getName());
}
```
### 1차 수정 : stream
``` java
List<String> lowCaloricDishesName = 
            menu.stream()
              .filter(d -> d.getCalories() < 400) // (1) 400 칼로리 이하의 요리 선택
              .sorted(comparing(Dish::getCalories)) // (2) 정렬
              .map(Dish::getName) // (3) getName 메소드를 참조하여 요리명 추출
              .collect(toList()); // (4) 리스트에 저장
```
### 2차 수정 : stream() -> parallelStream()으로 바꾸면 이 코드를 멀티코어 아키텍처에서 병렬로 실행
``` java
List<String> lowCaloricDishesName = 
            menu.parallelStream()
              .filter(d -> d.getCalories() < 400) // (1) 400 칼로리 이하의 요리 선택
              .sorted(comparing(Dish::getCalories)) // (2) 정렬
              .map(Dish::getName) // (3) getName 메소드를 참조하여 요리명 추출
              .collect(toList()); // (4) 리스트에 저장
```
`filter`, `sorted`, `map`, `collect` 같은 여러 빌딩 블록 연산을 연결해서 복잡한 데이터 처리 파이프 라인을 만들 수 있다. 여러 연산을 파이프라인으로 연결해도 여전히 가독성과 명확성이 유지된다.

`filter(람다) -> sorted(람다) -> map(람다) -> collect`

`filter` 메서드의 결과는 `sorted` 메서드로, 다시 `sorted` 결과는 `map` 메서드로, `map` 메서드의 결과는 `collect` 연결된다.

___

### 스트림

#### 스트림 API 핵심

스트림 API는 다량의 데이터 처리 작업을 위해 추가되었다. 이 API가 제공하는 추상 개념 중 핵심은 두가지다.

1. 스트림은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
2. 스트림 파이프라인(stream pipeline)은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

스트림 원소들은 어디로부터든 올 수 있다. 대표적으로는 컬렉션, 배열, 파일, 정규표현식 패턴 매치, 난수 생성기, 혹은 다른 스트림이 있다.

#### 특징

스트림 파이프라인은 소스스트림 -> (중간연산) -> 종단연산 으로 이루어진다.

#### 중간 연산

1. sorted
```Stream<Integer> sorted = operands.stream().sorted();```

2. filter
```Stream<Integer> integerStream = operands.stream().filter((value) -> value > 2);```

3. map:요소들을 변경하여 새로운 컨텐츠를 생성
```list.map(s -> s.toUpperCase()); //소문자를 대문자로 변경 ```

#### 종단 연산

마지막 중간 연산의 스트림에 최후의 연산을 가한다. 1개 이상의 중간 연산들은 계속 합쳐진 후 종단 연산시 수행된다. 원소를 정렬해 컬렉션에 담거나, 특정 원소를 하나 선택하거나, 모든 원소를 출력하는 식이다.

- 종단 연산이 없는 파이프라인은 여떤 연산도 수행되지 않는다.
- 지연평가는 무한 스트림을 다룰 수 있게 해주는 열쇠다.
<details>
<summary>종단 연산</summary>
<div markdown="1">

1. forEach() : 요소의 출력
``` java
intList.stream().forEach(System.out::println); // 1,2,3
intList.stream().forEach(x -> System.out.printf("%d : %d\n",x,x*x)); // 1,4,9
```

2. reduce() : 요소의 소모
- 두 개의 인자(n, n+1)을 가지며 연산과는 n이 되고 다시 다음 요소와 연산을 한다.

``` java
int sum = intList.stream().reduce((a,b) -> a+b).get();
System.out.println("sum: "+sum);  // 6
```

3. findFirst(), findAny() : 요소의 검색
- 스트림에서 지정한 첫 번째 요소를 찾는 메서드이다.

``` java
strList.stream().filter(s -> s.startsWith("H")).findFirst().ifPresent(System.out::println);  //Hwang
strList.parallelStream().filter(s -> s.startsWith("H")).findAny().ifPresent(System.out::println);  //Hwang or Hong
```

4. anyMatch(), allMatch(), noneMatch() : 요소의 검사
- 스트림의 요소중 특정 조건을 만족하는 요소를 검사하는 메서드. 원소중 일부, 전체 혹은 일치하는 것이 없는 경우를 검사하고 boolean 값을 리턴한다. noneMatch()의 경우 일치하는 것이 하나도 없을때 true.
```java
boolean result1 = strList.stream().anyMatch(s -> s.startsWith("H"));  //true
boolean result2 = strList.stream().allMatch(s -> s.startsWith("H"));  //false
boolean result3 = strList.stream().noneMatch(s -> s.startsWith("T")); //true
System.out.printf("%b, %b, %b",result1,result2, result3);
```

5. count(), min(), max() : 요소의 통계
- 스트림의 원소들로부터 전체 갯수, 최소값, 최대값을 구하기 위한 메서드
```java
intList.stream().count();	// 3
intList.stream().filter(n -> n !=2 ).count(); 	// 2
intList.stream().min(Integer::compare).ifPresent(System.out::println);; 		// 1
intList.stream().max(Integer::compareUnsigned).ifPresent(System.out::println);; // 3

strList.stream().count();	// 3
strList.stream().min(String::compareToIgnoreCase).ifPresent(System.out::println);	// Hong
strList.stream().max(String::compareTo).ifPresent(System.out::println);	// Kang
```
6. sum(), average() : 요소의 연산
- 스트림의 원소들의 합계를 구하거나 평균을 구하는 메서드이다.
```java
intList.stream().mapToInt(Integer::intValue).sum();	// 6
intList.stream().reduce((a,b) -> a+b).ifPresent(System.out::println); // 6

intList.stream().mapToInt(Integer::intValue).average();	// 2
intList.stream().reduce((a,b) -> a+b).map(n -> n/intList.size()).ifPresent(System.out::println); // 2
```
7. collect() : 요소의 수집
스트림의 결과를 모으기 위한 메서드로 Collectors 객체에 구현된 방법에 따라 처리하는 메서드이다. 최종 처리 후 데이터를 변환하는 경우가 많기 때문에 잘 알아 두어야한다. 용도별로 사용할 수 있는 Collectors의 메서드는 기능별로 다음과 같다.
- 스트림을 배열이나 컬렉션으로 변환 : toArray(), toCollection(), toList(), toSet(), toMap()
- 요소의 통계와 연산 메소드와 같은 동작을 수행 : counting(), maxBy(), minBy(), summingInt(), averagingInt() 등
- 요소의 소모와 같은 동작을 수행 : reducing(), joining()
- 요소의 그룹화와 분할 : groupingBy(), partitioningBy()
``` java
strList.stream().map(String::toUpperCase).collect(Collectors.joining("/"));	 	// Hwang/Hong/Kang
strList.stream().collect(Collectors.toMap(k -> k, v -> v.length()));	// {Hong=4, Hwang=5, Kang=4}

intList.stream().collect(Collectors.counting());
intList.stream().collect(Collectors.maxBy(Integer::compare));
intList.stream().collect(Collectors.reducing((a,b) -> a+b));	// 6
intList.stream().collect(Collectors.summarizingInt(x -> x));	//IntSummaryStatistics{count=3, sum=6, min=1, average=2.000000, max=3}

Map<Boolean, List<String>> group = strList.stream().collect(Collectors.groupingBy(s -> s.startsWith("H")));
group.get(true).forEach(System.out::println);  // Hwang, Hong

Map<Boolean, List<String>> partition = strList.stream().collect(Collectors.partitioningBy(s -> s.startsWith("H")));
partition.get(true).stream().forEach(System.out::println);  // Hwang, Hong
```
</div>
</details>

___
### 스트림 파이프라인

스트림 파이프 라인은 지연 평가된다. 평가는 종단 연산이 호출될 때 이루어지며 종단 연산에 쓰이지 않는 데이터 원소는 계산에 쓰이지 않는다. 종단 연산이 없으면 아무일도 수행되지 않는다. 따라서 종단 연산을 빼먹는 일이 절대 없도록 한다.

스트림 API는 메서드 연쇄를 지원하는 플루언트(fluent) API 이다. 즉, 파이프라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다. 파이프라인 여러개를 연결해 표현식 하나로 만들 수도 있다. 기본적으로 스트림 파이프라인은 순차적으로 실행된다. 파이프라인을 병렬로 실행하려면 파이프라인을 구성하는 스트림 중 하나에서 `parallel` 메서드를 호출해주기만 하면 되나, 효과를 볼수 있는 상황은 많지 않다.

다음 코드는 중간 연산만 실행되는 코드이다. 종단 연산을 추가하면 순차적으로 수행되는 것을 알 수 있다.

``` java
import java.util.stream.Stream;

public class StreamTest1 {

    public static void main(String[] args) {
        Stream.of("d2", "a2", "b1", "b3", "c")
                .map(s -> {     //중간연산
                    System.out.println("map: " + s);
                    return s.toUpperCase();
                })
                .filter(s -> {  //중간연산
                    System.out.println("filter: " + s);
                    return s.startsWith("A");
                });
                //}).forEach(s -> System.out.println("필터 결과 : " + s)); //종단 연산
    }

}
```

> map: d2
filter: D2
map: a2
filter: A2
최종 결과 : A2
map: b3
filter: B3
map: c
filter: C
___
### 스트림 사용시 주의

대부분의 연산은 스트림으로 구현할 수 있다. 하지만 과도한 스트림은 읽기도 어렵고 유지보수도 힘들다. 또한 성능상 좋지 않을 수 있다. 아래 코드처럼 스트림을 과하게 사용하면 읽기 힘들다.
스트림을 처음 사용하면 모든 반복문을 스트림으로 바꾸고 싶지만 가독성과 유지보수 측면에서 손해를 볼 수 있기 때문에 중간 정도 복잡한 작업에서 스트림과 반복문을 적절히 조합해서 사용하는게 최선이다. 즉, 기존 코드는 스트림으로 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영해야 한다.

#### 과한 스트림 사용
```java
public class AnagramsStream {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(
                            groupingBy(word -> word.chars().sorted()
                                    .collect(StringBuilder::new,
                                            (sb, c) -> sb.append((char) c),
                                            StringBuilder::append).toString()))
                    .values().stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .map(group -> group.size() + ": " + group)
                    .forEach(System.out::println);
        }
    }
}
 
```
#### 짧고 명확한 스트림 사용
```java
public class AnagramsStream {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);

        // 파일을 열고, 파일의 모든 라인으로 구성된 스트림을 얻고, 스트림 변수를 words 로 한다.
        try (Stream<String> words = Files.lines(dictionary)) {
            // 스트림 파이프라인 
            words.collect(groupingBy(word -> alphabetize(word))) // map 으로 모은다.
                    .values()
                    // values()가 반환한 값 : Stream<List<String>>
                    .stream()
                    .filter(group -> group.size() >= minGroupSize)
                    .forEach(group -> System.out.println(group.size() + ": " + group));
        }
    }

    private static String alphabetize(String s) {
        char[] a = s.toCharArray();
        Arrays.sort(a);
        return new String(a);
    }
}
```
___
### 자바는 char용 스트림을 지원하지 않는다.

```"Hello Words".chars().forEach(x -> System.out.println((char) x));```

`char` 값을 처리하기 위해서는 위 코드처럼 명시적 형변환을 해줘야 한다. 명시적 형변환을 해주지 않으면, `chars()` 메서드가 반환하는 스트림의 원소는 `char` 가 아닌 `int` 값이기 때문에 원하지 않는 결과가 나온다. 

```java
@Override
public IntStream chars() {
    return StreamSupport.intStream(
        isLatin1() ? new StringLatin1.CharsSpliterator(value, Spliterator.IMMUTABLE) 
        			: new StringUTF16.CharsSpliterator(value, Spliterator.IMMUTABLE),
        false);
}
```
___
### 스트림이 적절할 때

아래의 일 중 하나를 수행하는 로직이라면 스트림을 적용하기에 좋은 후보이다.

1. 원소들의 시퀀스를 일관성 있게 변환할 때

2. 원소들의 시퀀스를 필터링할 때

3. 원소들의 시퀀스를 연산 후 결합할 때

4. 원소들의 시퀀스를 모을 때

5. 원소들의 시퀀스 중 특정 조건을 만족하는 원소를 찾을 때
___

### 스트림으로 처리하기 어려운 경우
> 원본 스트림을 계속 써야할 때 : 스트림은 중간 연산을 지나고 나면 원래 스트림을 잃는 구조이다.

스트림으로 처리하기 어려운 일을 예로 들어보면, 한 데이터가 파이프라인의 여러 단계를 통과할때 이 데이터의 각 단계에서의 값들에 동시에 접근하기는 어려운 경우다. 스트림 파이프라인은 일단 한 값을 다른 값에 매핑하고 나면 원래의 값은 잃는 구조기 때문이다. 원래 값과 새로운 값의 쌍을 저장하는 객체를 사용하여 해결할 순 있지만, 만족스러운 해법은 아니다. 이 방식은 코드 양도 많고 지저분하여 스트림을 쓰는 주 목적에서 완전히 벗어난다.
___
### 스트림 vs. 반복문

둘 중 어느 것을 사용해야할지 바로 알기 어려운 작업이 많다. 카드 덱을 초기화하는 작업이 그 예이다. 카드는 숫자/무늬를 묶은 불변 값 클래스이고, 숫자와 무늬는 모두 열거타입이라 한다. 이 작업은 두 집합의 원소들로 만들 수 있는 가능한 모든 조합을 계산하는 문제다. 수학자들은 이를 두 집합의 데카르트 곱이라고 한다.

```java
private static List<Card> newDeck() {
    List<Card> result = new ArrayList<>();
    for (Suit suit : Suit.values())
        for (Rank rank : Rank.values())
            result.add(new Card(suit, rank));
    return result;
}
```

이를 스트림으로 구현하면
``` java
private static List<Card> newDeck() {
    return Stream.of(Suit.values())
        .flatMap(suit -> Stream.of(Rank.values())
        .map(rank -> new Card(suit, rank)))
        .collect(toList());
}
```
중간 연산을 사용한 `flatMap` 은 스트림의 원소 각각을 하나의 스트림으로 매핑한 다음 그 스트림들을 다시 하나의 스트림으로 합친다.

#### 정리

반복문 사용 코드와 스트림 사용 코드 중 어떤게 더 좋아보일지는 개인 취향과 프로그래밍 환경의 문제다. 반복문 방식은 더 단순하고 더 자연스러워보인다. 따라서 반복문과 스트림 중의 선택은 개인의 선택이고, 확신이 드는 코드를 선택하면 된다. 주변 동료들이 스트림 코드를 이해할 수 있고 선호한다면 스트림 방식을 선택할 수도 있는 것이다.
