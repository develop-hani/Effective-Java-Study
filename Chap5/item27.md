## 아이템27. 비검사 경고를 제거하라

제네릭을 사용하면 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등의 수많은 경고를 볼 수 있다. 

대부분의 비검사 경고는 쉽게 제거가 가능하다.

``` java
Set<Lark> exaltation = new HashSet();
_______________________________________________________________
Venery.java:4: warning: [unchecked] unchecked conversion
	Set<Lark> exaltation = new HashSet();
    					   ^
	required: Set<Lark>
    found:	HashSet
```
```
Set<Lark> exaltation = new HashSet<>();
```
컴파일러가 알려준 대로 수정하면 경고가 사라진다.

### 할 수 있는 한 모든 비검사 경고를 제거하라.

타입 안정성이 보장된다. 즉, 런타임에 ClassCastException이 발생할 일이 없고 의도한 대로 잘 동작하리라 확신할 수 있다.

### 경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.

안전하다고 검증된 비검사 경고를 숨기지 않고 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다. 제거하지 않은 수만은 경고 속에 새로운 경고가 파묻힐 것이기 때문이다.

### @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자.

보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될 것이다. 자칫 심각한 경고를 놓칠 수 있으므로 절대 클래스 전체에 적용해서는 안된다.
```
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
_______________________________________________________________
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
       @SuppressWarnings("unchecked")
       T[] result = (T[]) Arrays.copyOf(elementData, size, a.getClass());
        return result;
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```
애너테이션은 선언에만 달 수 있기 때문에 return 문에는 불가능하다. 메서드 전체에 달지 않고 반환값을 담을 지역변수를 하나 선언하고 그 변수에 애너테이션을 달아주는 것이 좋다. 

### @SuppressWarnings("unchecked") 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.

다른 사람이 그 코드를 이해하는 데에 도움이 되며, 다른 사람이 그 코드를 잘못 수정하여 타입 안전성을 잃는 상황을 줄여준다.
