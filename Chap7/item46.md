# 스트림에서는 부작용 없는 함수를 사용하라

## 스트림

스트림은 함수형 프로그래밍에 기초한 패러다임이다.<br>
스트림의 변환단계는 가능한 한 이전 단계의 결과를 받아 처리하는 순수 함수여야 한다.

> 순수함수: 오직 입력만이 결과에 영향을 주는 함수를 말한다. 다른 가변 상태를 참조하지 않고 변경하지 않는다.

```java
// 안 좋은 스트림의 예
Map<String, Long> freq = new HashMap();
try (Stream<String> words = new Scanner(file).tokens()) {
  words.forEach(word -> {
      freq.merge(word.toLowercase(), 1L, Long::sum);
    });
}

```
- 위의 코드는 스트림에서 외부 상태를 수정하고 forEach에서 모든 작업의 연산이 포함되어 있어서 안 좋은 스트림 패러다임의 예이다.<br>
- **foreach 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자**

```java
// 스트림을 제대로 활용한 좋은 예
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
  freq= words
    .collect(groupingBy(String::toLowerCase, counting()));
}
```

---

## Collector

>Collector(수집기): 스트림의 요소를 어떤식으로 도출할지 지정한다.

Collector 클래스는 메서드를 통해 어떤식으로 도출할지 정한다.

### 컬렉션으로 도출해주는 Collector 메서드 

- toList(), toSet(), toCollection(collectionFactory)


```java
// 빈도표에서 가장 흔히 단어 10개를 뽑아내는 파이프라인, Collector의 toList 사용
List<String> topten = freq.keySet().stream()
  .sorted(comparing(freq::get).reversed()
  .limit(10)
  .collect(toList());
```

단어들을 스트림으로 받아서 빈도수를 역순으로 나열한 후 가장 높은 빈도 수 10개의 단어를 List로 호출

---

### toMap

toMap의 함수는 총 3개로 정의되어 있다.


1. toMap(Function<? super T, ? extends K> **keyMapper**, Function<? super T, ? extends U> **valueMapper**)

2. toMap(Function<? super T, ? extends K> **keyMapper**, Function<? super T, ? extends U> **valueMapper**, BinaryOperator< U> **mergeFunction**)

3. toMap(Function<? super T, ? extends K> **keyMapper**, Function<? super T, ? extends U> **valueMapper**, BinaryOperator< U> **mergeFunction**, Supplier<M> **mapFactory**)<br>
&rarr;mapFactory는 Map의 특정 구현체를 직접 지정할 수 있게 해준다.

#### collector를 통해 문자열을 열거 타입 상수에 매핑하는 예, 1번 사용

```java 
private statoc final Map<String, Operation> stringToEnum = 
  Stream.of(value()).collect(
    toMap(Object::toString, e -> e); // toMap(keyMapper, valueMapper)

```

1번 같은 ToMap은 스트림 원소 다수가 키에 매핑되면 IllegalStateException이 발생한다.

이때 mergeFunction이 들어간 toMap을 사용하면 된다.<br>
같은 키를 공유하는 값들을 병합해서 값으로 사용한다.


#### mergeFunction이 들어간 toMap 사용 예시

```java
Map<Artist, Album> topHits = album.collect(
  toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
```

위의 코드는 병합할 때 sales 값을 비교해서 가장 큰 것의 값으로 병합했다.<br>
artist 중에 가장 많이 팔린 앨범이 키의 값으로 들어간다.

#### mergeFunction의 들어간 toMap의 다른 예

```java
toMap(keyMapper, valueMapper, (oldVal, newVal) -> newVal)
```

---

### groupingBy

1. groupingBy(Function<? super T,? extends K> **classifier**)


2. groupingBy(Function<? super T,? extends K> **classifier**, Supplier<M> **mapFactory**, Collector<? super T,A,D> **downstream**)<br>
&rarr; mapFactory는 map의 특정 구현체를 직접 지정하게 해준다.

3. groupingBy(Function<? super T,? extends K> **classifier**, Collector<? super T,A,D> **downstream**)

- groupingBy 메서드는 classifier로 받고 classifier로 받은 원소를 모아 놓은 Map을 담은 Collector를 반환한다.

#### 매개변수가 하나인 groupingBy

```java
words.collect(groupingBy(word -> alphabetsize(word)))
```
알파벳화한 단어를 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵 생성

#### downStream을 사용하는 groupingBy 예

리스트 이외의 값으로 매핑하고 싶은 경우 downStream을 사용하면 된다.<br>
downStream이 ToSet을 받으면 List 대신 Set을 값으로 매핑한다.<br>

```java 
Map<String, Long> freq = words
  .collect(groupingBy(String::toLowerCase, counting()));
```

counting을 다운스트림으로 사용하면 해당 키의 속하는 원소의 개수와 매핑해준다.

---
### 기타 Collector 메서드 

- minBy, maxBy: 비교자를 이용해 스트림에서 값이 가장 적은, 가장 큰 원소를 반환한다.

- joining: 원소들을 연결하는 수집기를 반환한다. 인수가 1개면 구분문자를 넣어주고, 인수가 3개면 구분 문자와 처음과 끝에 문자를 붙여준다.
