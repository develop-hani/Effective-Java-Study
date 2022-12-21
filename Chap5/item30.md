## 아이템 30. 이왕이면 제네릭 메서드로 만들라

클래스와 마찬가지로, 메서드도 제네릭으로 만들 수 있다. 매개변수화 타입을 받는 정적 유틸리티 메서드는 보통 제네릭이다.
![](https://velog.velcdn.com/images/lcy923/post/d11ca047-86d5-47f4-b974-906936b7325b/image.png)

ex. `Collections` 의 '알고리즘' 메서드(binarySearch, sort 등 ..)

``` java
public static <T>
    int binarySearch(List<? extends Comparable<? super T>> list, T key) {
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key);
        else
            return Collections.iteratorBinarySearch(list, key);
    }
```

모든 원소가 `T`의 수퍼타입과 비교 가능한 `Comparable`을 상속하고 있는 `list`와 `T`타입 `key` 를 비교한다.


### 로 타입 사용 - 수용불가!

다음은 두 집합의 합집합을 반환하는 문제가 있는 메서드이다.

``` java
public static Set union(Set s1, Set s2) {
    Set result = new HashSet(s1);
    result.addAll(s2);

    return result;
}
```
컴파일은 되지만, 경고가 두 개 발생한다.

```
Union.java:5 warning: [unchecked] unchecked call to
HashSet(Collection<? extends E?) as a member of raw type HashSet
	Set result = new HashSet(S1);
			     ^
```

```
Union.java:5 warning: [unchecked] unchecked call to
addAll(Collection<? extends E?) as a member of raw type HashSet
	result.addAll(S2);
    			  ^
```

컴파일은 되지만 경고가 발생할 것이다. 경고를 없애려면 이 메서드를 타입 안전하게 만들어야 한다. 메서드 선언에서의 세 집합(입력 2개, 반환 1개) 의 원소 타입을 타입 매개변수로 명시하고, 메서드 안에서도 이 타입 매개변수만 사용하게 수정하면 된다.

타입 매개변수들을 선언하는 타입 매개변수 목록은 메서드의 제한자 반환 타입 사이에 온다. 타입 매개변수의 명명규칙은 제네릭 메서드나 제네릭 타입이나 똑같다.

### 제네릭 메서드

#### 1. 단순한 제네릭 메서드 - 모든 매개변수 타입이 같아야 한다.
``` java
public class Item30 {
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
}
```
단순한 제네릭 메서드라면 이 정도면 충분하다. 이 메서드는 경고 없이 컴파일되며, 타입 안전하며, 쓰기도 쉽다.

``` java
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
    Set<E> guys = Set.of("톰", "딕", "해리");
    Set<String> stooges = Set.of("래리", "모에", "컬리");
    Set<String> aflCio = union(guys, stooges);
    System.out.println(aflCio);
}
```

다음은 위의 메서드를 사용하는 간단한 프로그램이다. 이 프로그램을 실행하면 `[모에, 톰, 해리, 래리, 컬리, 딕]`이 출력된다.

직접 형변환하지 않아도 어떤 오류나 경고 없이 컴파일되지만, 단점으로는 유연성이 떨어진다. 이를 한정적 와일드카드 타입(아이템 31)을 통해 더 유연하게 개선할 수 있다. 

#### 2. 제네릭 싱글턴 팩터리 : 요청 타입 변수에 맞게 객체의 타입을 바꿔줌

때때로 불변 객체를 여러 타입으로 활용할 수 있게 만들어야 할 때가 있다. 제네릭은 런타임에 타입 정보가 소거(아이템 28)되므로 하나의 객체를 어떤 타입으로든 매개변수화 할 수 있다. 하지만 이렇게 하려면 요청한 타입 매개변수에 맞게 매번 그 객체의 타입을 바꿔주는 정적 팩터리를 만들어야 한다. 이 패턴이 제네릭 싱글턴 팩터리이다.
ex. `Collections.reverseOrder`(아이템42), `Collections.emptySet`

제네릭 싱글턴 팩터리 : 제네릭으로 타입설정 가능한 인스턴스 만들어두고, 반환 시에 제네릭으로 받은 타입을 이용해 타입을 결정

``` java
public class GenericFactoryMethod {
public static final Set EMPTY_SET = new HashSet();

    public static final <T> Set<T> emptySet() {
        return (Set<T>) EMPTY_SET;
    }
}
'''
'''java
@Test
public void genericTest() {
    Set<String> set = GenericFactoryMethod.emptySet();
    Set<Integer> set2 = GenericFactoryMethod.emptySet();
    Set<Elvis> set3 = GenericFactoryMethod.emptySet();

    set.add("ab");
    set2.add(123);
    set3.add(Elvis.INSTANCE);

    String s = set.toString();
    System.out.println("s = " + s);
}
```
> s = [ab, item3.Elvis@3439f68d, 123]

항등함수 객체는 상태가 없으니 요청할 때마다 새로 생성하는 것이 낭비다. 

``` java
private static UnaryOperator<Object> IDENTITY_FN = (t) -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
}
```

`IDENTITY_FN`을 `UnaryOperator<T>`로 형변환하면 비검사 형변환 경고가 발생한다. `T`가 어떤 타입이든 `UnaryOperator<Object>`는 `UnaryOperator<T>`가 아니기 때문이다.
하지만 항등함수란 입력 값을 수정 없이 그대로 반환하는 특별한 함수이므로, `T`가 어떤 타입이든 `UnaryOperator<T>`를 사용해도 타입 안전하다. 따라서 `@SuppressWarnings` 애너테이션을 추가하면 오류나 경고 없이 컴파일된다.

다음 코드는 이전 코드의 제네릭 싱글턴을 `UnaryOperator<String>`과 `UnaryOperator<Number>`로 사용하는 모습이다. 지금까지와 마찬가지로 형변환을 하지 않아도 컴파일 오류나 경고가 발생하지 않는다.

``` java
public static void main(String[] args) {
        String[] strings = {"삼베", "대마", "나일론"};
        UnaryOperator<String> sameString = identityFunction();
        for (String s : strings) {
            System.out.println(sameString.apply(s));
        }

        Number[] numbers = {1, 2.0, 3L};
        UnaryOperator<Number> sameNumber = identityFunction();
        for (Number n : numbers) {
            System.out.println(sameNumber.apply(n));
        }
}
```


#### 3. 재귀적 타입 한정 : 타입 매개변수의 허용 범위를 한정

자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있다. 이를 재귀적 타입 한정(recursive type bound)이라고 한다. 재귀적 타입 한정은 주로 타입의 자연적 순서를 정하는 `Comparable` 인터페이스와 함께 쓰인다. 

``` java
public interface Comparable<T> {
    int compareTo(T o);
}
```
여기서 타입 매개변수 `T`는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다. 실제로 거의 모든 타입은 자신과 같은 타입의 원소와만 비교할 수 있다. 따라서 `String`은 `Comparable<String>`을 구현하고 `Integer`는 `Comparable<Integer>`를 구현하는 식이다.

#### 재귀적 타입 한정을 이용해 상호 비교

`Comparable`을 구현한 원소의 컬렉션을 입력받는 메서드들은 주로 그 원소들을 정렬 혹은 검색하거나, 최솟값이나 최댓값을 구하는 식으로 사용된다. 이 기능을 수행하려면 컬렉션에 담긴 모든 원소가 상호 비교될 수 있어야 한다. 다음은 이 제약을 코드로 표현한 모습이다.

``` java
public class RecursiveTypeBoundEx {
    public static <E extends Comparable<E>> E max(Collection<E> c);
}
```
#### 컬렉션에서 최댓값 반환 - 재귀적 타입 한정 사용

``` java
public class RecursiveTypeBoundEx {
    public static <E extends Comparable<E>> E max(Collection<E> collection) {
        if (collection.isEmpty()) {
            throw new IllegalArgumentException("컬렉션이 비어 있습니다.");
        }

        E result = null;
        for (E e : collection) {
            if (result == null || e.compareTo(result) > 0) {
                result = Objects.requireNonNull(e);
            }
        }
        return result;
    }
}
```
이 메서드에서는 빈 컬렉션일 경우 `IllegalArgumentException`을 던진다. 따라서 `Optional<E>`를 반환하도록 하는 것이 더 낫다.
재귀적 타입 한정은 훨씬 복잡해질 가능성이 있긴 하지만, 다행히 그런일은 잘 일어나지 않는다.


### 핵심 정리
  
- 제네릭 타입과 마찬가지로, 클라이언트에서 입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드보다 제네릭 메서드가 더 안전하고 사용하기도 쉽다.
- 타입과 마찬가지로, 메서드도 형변환 없이 사용할 수 있는 편이 좋으며, 많은 경우 그렇게 하려면 제네릭 메서드가 되어야 한다.
- 타입과 마찬가지로, 형변환을 해줘야 하는 기존 메서드는 제네릭하게 만들도록 한다.
- 제네릭하게 만들면 기존 클라이언트는 그대로 둔 채 새로운 사용자에게 편의를 제공해줄 수 있다.
