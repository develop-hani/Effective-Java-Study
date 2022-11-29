상속보다는 컴포지션을 사용하라
=
>상속 -> 인터페이스를 구현하거나 인터페이스가 인터페이스를 확장하는 인터페이스 상속이 아닌 클래스가 다른 클래스를 확장하는 구현상속을 대상으로 얘기한다.
## 상속의 단점
### 메서드 호출과 달리 상속은 캡슐화를 깨트린다.
- 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.\
\
→자신의 다른 부분을 사용하는 (메서드가 같은 클래스 내의 메서드를 사용하는) '자기사용' 여부에 따라 이 메서드를 오버라이딩 했을때 문제가 발생할 수 있다. 
하지만 이 자기사용 여부는 클래스의 내부 구현에 해당하여 알 수 없다.\
→다음 릴리스에서 상위 클래스에 새로운 메서드가 추가됨에 따라 보안 문제가 발생할 수 있다.\
→이를 피하기 위해 메서드를 재정의 하지 않고 추가하는 방식을 선택했다 해도 다음 릴리스때 그 메서드와 같은 이름의 메서드가 다음 릴리스에 추가 된다면 문제가 발생한다.

## 상속의 문제를 피해가는 방법
### 컴포지션 
- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하는 설계

새 클래스의 인스턴스 메서드들은 (private 필드로 참조하는) 기존 클래스에 대응하는 메서드를 호출해 그 결과를 반환한다.
이 방식을 전달(forwarding)이라 하며, 새 클래스의 메서드들은 전달 메서드(forwarding method)라 부른다.
그 결과 새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향받지 않는다.

다음은 컴포지션과 전달 방식을 이용한 코드 예시이다.
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {//래퍼 클래스-상속대신 컴포지션 사용
    private int addCount = 0;
    public InstrumentedSet(Set<E> s) {
        super(s);
    }
    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }
}
```
```java
public class ForwardingSet<E> implements Set<E> {//재사용 할 수 있는 전달 클래스
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }


    public void clear() { s.clear(); }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty() { return s.isEmpty(); }
    public int size() { return s.size(); }
    public Iterator<E> iterator() { return s.iterator(); }
    public boolean add(E e) { return s.add(e); }
    public boolean remove(Object o) { return s.remove(o);}
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c); }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c); }
    public boolean retainAll(Collection<?> c) { return s. retainAll(c); }
    public Object[] toArray() { return s.toArray(); }
    public <T> T[] toArray(T[] a) { return s.toArray(a); }
    @Override public boolean equals(Object o) { return s.equals(o); }
    @Override public int hashCode() { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```
다른 Set 인스턴스를 감싸고 있다는 뜻에서 Instrumented 같은 클래스를 래퍼클래스라 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다.
>래퍼 클래스는 콜백 프레임워크와 어울리지 않는다.\
>콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다. 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고ㅡ 콜백때는 래퍼가 아닌 내부 객체를 호출하게 되어 SELF 문제가 발생한다,
