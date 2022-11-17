## Comparable 인터페이스란,

- 객체 간의 비교를 할 수 있는 인터페이스이다.
    
    Comparable의 유일한 method `compareTo()` 는 단순 동치성 비교에 더해 순서까지 비교할 수 있으며 제네릭하다.
    
- compareTo를 구현한 객체들의 배열은 쉽게 정렬할 수 있다.
    
    e.g)`Arrays.sort(a)`, String 타입의 문자열을 알파벳 순서대로 정렬 등
    

## CompareTo의 규약

객체와 주어진 객체의 순서를 비교한다.

객체가 주어진 객체보다 작으면 음의 정수, 같으면 0, 크면 양의 정수 반환한다.

비교할 수 없을 땐 `ClassCastException`

- 대칭성
    
    `sgn(x.compareTo(y)) == -sng(y.compareTo(x))`
    
- 추이성
    
     `x.compareTo(y) > 0 && y.compareTo(z) > 0 then x.compareTo(z) > 0`
    
- 반사성
    
    `x.compareTo(y) == 0 then sgn(x.compareTo(z)) == sgn(y.compareTo(z))`
    
- 필수는 아니지만 권장
    
    `x.compareTo(y) == 0 then x.equals(y) == true`
    

## CompareTo 작성 요령

전체적으로 `equals()` 와 유사하다.

- 입력 인수의 타입을 확인하거나 형변환할 필요가 없다.
    
    Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일타임에 정해지기 때문이다.
    
- `compareTo()`는 각 필드가 동치인지 비교하는 것이 아니라 그 순서를 비교한다.

## CompareTo의 구현

- 클래스에 핵심 필드가 여러 개라면 가장 핵심적인 필드부터 비교해나가자.
    
    핵심적인 필드부터 비교해나가며 비교 결과가 0이 아니라면 바로 반환한다.
    
    ```java
    public class Person {
    	private int age; // 1순위
    	private double height; // 2순위
    	private String name; // 3순위
    }
    ```
    
    ```java
    public int compareTo(Person p) {
    	int result = Integer.compare(age, p.age);
    	if(result == 0) {
    		result = Doubld.compare(height, p.height);
    	}
    	if(result == 0) {
    		result = name.compareTo(p.name)l
    	}
    	return result;
    }
    ```
    
- 비교자 생성 메서드(Comparator Construction Method)를 이용한 `compareTo()` 구현
    
    자바 8에서 method 연쇄 방식으로 비교자를 생성하여 사용하였다.
    
    ```java
    private static final Comparator<Person> COMPARATOR = 
    	Comparator.comparingIng(Person::getAge)
    		.thenComparingDouble(Person::getHeight)
    		.thenComparing(person -> person.getName());
    public int compareTo(Person p) {
    	return COMPARATOR.compare(this, p);
    }
    ```
    

## **⭐ 핵심정리**

- 순서를 고려해야하는 클래스를 작성한다면 Comparable을 구현하여 compareTo의 이점을 누릴 수 있다.
- 하지만 정렬의 기준이 고정적이 아니라면 다른 방식을 고려해 볼 수 있다.
