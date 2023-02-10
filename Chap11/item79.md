## 아이템 79. 과도한 동기화는 피하라

과도한 동기화는 성능을 떨어뜨리고, 교착상태에 빠뜨리고, 예측할 수 없는 동작을 일으킬 수 있다.

## 과도한 동기화

응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양보하면 안된다.

## 외계인 메서드

응답 불가와 안전 실패를 유발할 수 있는 메서드. 즉, 동기화된 영역에서 재정의할 수 있는 메서드 혹은 클라이언트가 넘겨준 함수 객체 등을 동기화된 클래스 관점에서 외계인 메서드라고 한다.

동기화된 클래스는 메서드가 무슨 일을 할 지 알지 못하며 통제할 수도 없고 외계인 메서드가 하는 일에 따라 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

### 예제 1
#### ForwardingSet
``` java
class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) {this.s = s;}
    @Override
    public int size() {    return s.size();}
    @Override
    public boolean isEmpty() {return s.isEmpty();}
    @Override
    public boolean contains(Object o) {return s.contains(o);}
    @Override
    public Iterator<E> iterator() {return s.iterator();}
    @Override
    public Object[] toArray() {return s.toArray();}
    @Override
    public <T> T[] toArray(T[] a) {return s.toArray(a);}
    @Override
    public boolean add(E e) {return s.add(e);}
    @Override
    public boolean remove(Object o) {return s.remove(o);}
    @Override
    public boolean containsAll(Collection<?> c) {return s.containsAll(c);}
    @Override
    public boolean addAll(Collection<? extends E> c) {return s.addAll(c);}
    @Override
    public boolean retainAll(Collection<?> c) {return s.retainAll(c);}
    @Override
    public boolean removeAll(Collection<?> c) {return s.removeAll(c);}
    @Override
    public void clear() {s.clear();}
}
```
#### ObservableSet

``` java
// 잘못된 코드 - 동기화 블록 안에서 외계인 메서드를 호출한다.
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }
    
    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }
    
    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for(SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```
관찰자는 `addObserver`와 `removeObserver` 메서드를 호출해 구독을 신청하거나 해지한다. 두 경우 모두 다음 콜백 인터페이스의 인스턴스를 메서드에 건넨다.

``` java
@FunctionalInterface
public interface SetObserver<E> {
    //ObservableSet에 원소가 더해지면 호출된다.
    void added(ObservableSet<E> set, E element);
}
```
___

이제 집합에 추가된 0부터 99까지 정수 값을 출력하다가, 그 값이 23이면 자기 자신을 제거(구독해지)하는 관찰자를 추가해보자.

``` java
public class Dummy {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if(e == 23) {
                    s.removeObserver(this);
                }
            }
        });

        for(int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```

이 프로그램은 0부터 23까지 출력한 후 관찰자 자신을 구독해지한 다음 종료될 것이라 예상된다. 하지만 실제로 실행해 보면 23까지 출력한 후 `ConcurrentModificationException`을 던진다.
> **ConcurrentModificationException**
해당 컬렉션 객체의 데이터 일관성에 대한 Exception이다. 

컬렉션의 객체가 변함으로 인한 비정상적인 환경이 발생했다는 것을 알려준다.
___

이렇게 예외가 발생하는 이유는 `added` 메서드 호출이 일어난 시점이 `notifyElementAdded`가 관찰자들의 리스트를 순회하는 도중이기 때문이다.

`added` 메서드는 `ObservableSet`의 `removeObserver` 메서드를 호출하고, 이 메서드는 다시 `observers.remove` 메서드를 호출한다. 이때 문제가 발생하게 된다. 리스트에서 원소를 제거하려는데, 이 리스트를 순회하는 도중이기 때문에 허용되지 않는 동작으로 인식한다.

`notifyElementAdded` 메서드에서 수행하는 순회는 동기화 블록이므로 수정이 일어나지 않도록 보장하지만, 정작 자신이 콜백을 거쳐 되돌아와 수정하는 것까지는 막지 못한다.

정리하자면,

1. `main`에서 `set.add()`가 호출되면 `ObservableSet`의 재정의된 `add()`가 호출된다.

2. 재정의된 `add`는 `notifyElementAdded()`를 호출한다.

3.`notifyElementAdded()`에서는 관찰자 목록(`List<SetObserver<E>>`)을 순회하며 `added()`를 호출한다.

4. `main`에서 익명 함수로 정의한 `added`가 호출된다. 해당 `added`는 특정 조건에서 `removeObserver` 메서드를 호출한다.

5. 이때, `removeObserver()`가 호출되면 콜백으로 되돌아와 자신을 수정하는 것을 막지 못하므로 원소가 삭제된다.

6. `notifyElementAdded()`에서 순회 중인 동기화 블록에서 `ConcurrentModificationException`이 발생한다.(동기화가 걸려있음에도 원소가 삭제됨 → 동기화 오류)

___

### 예제 2

이번에는 구독해지를 하는 관찰자를 작성할 때, 직접 `removeObserver`를 호출하는 것이 아닌, 실행자 서비스(`ExecutorService`)를 사용해 다른 스레드에게 부탁할 것이다.

``` java
// 백그라운드 스레드를 사용하는 예제
public class Dummy {
    public static void main(String[] args) {
        ObservableSet<Integer> set = new ObservableSet<>(new HashSet<>());
        set.addObserver(new SetObserver<>() {
            public void added(ObservableSet<Integer> s, Integer e) {
                System.out.println(e);
                if(e == 23) {
                    ExecutorService exec = Executors.newSingleThreadExecutor();
                    try {
                        exec.submit(() -> s.removeObserver(this)).get(); // lock 걸림 - 접근 불가
                        // 메인 스레드는 작업을 기다림
                    } catch(ExecutionException | InterruptedException ex) {
                        throw new AssertionError(ex);
                    } finally {
                        exec.shutdown();
                    }
                }
            }
        });
        
        for(int i = 0; i < 100; i++) {
            set.add(i);
        }
    }
}
```
이 프로그램은 예외가 발생하진 않지만, 교착상태에 빠져버린다. 백그라운드 스레드가 `s.removeObserver`를 호출하면 관찰자에 Lock을 시도하지만 메인 스레드가 `Lock`을 가지고 있기 때문에 `Lock`을 얻을 수 없다.

메인 스레드는 백그라운드 스레드가 관찰자를 제거하기만을 기다리고 백그라운드 스레드는 메인 스레드의 `lock`을 획득하기 위해 기다리고 있는 상태, **바로 교착상태다.**

만약 앞선 예제들의 리소스(관찰자)가 일관된 상태가 아닌 임시로 불변식이 깨진 상태라면 어떨까?

Java의 Lock은 재진입을 허용하므로 교착상태에는 빠지지 않는다. 하지만 예외를 발생시킨 예에서는 외계인 메서드를 호출하는 스레드가 이미 Lock을 획득하고 있고 다음 재진입에서도 Lock을 획득한다. 이는 Lock이 제 역할을 하지 못하는 것을 나타내고 재진입 가능 락으로 인해 **응답 불가(교착상태)가 될 상황을 안전 실패(데이터 훼손) 상태로 변질시킬 위험을 나타낸다.**

재진입 가능 락은 교착상태는 방지할 수 있지만 근본적인 해결 방법은 되지 못한다.

> **락의 재진입 **
이미 락을 획득한 스레드는 다른 `synchronized` 블록을 만났을때 락을 다시 검사하지 않고 진입 가능하다.

___

## 해결 방법

### 1. 동기화 블록 밖으로 외계인 메서드 옮기기

외계인 메서드 호출을 동기화 블록 바깥으로 옮기면 위와 같은 문제들을 다행히 어렵지 않게 해결할 수 있다. `notifyElementAdded` 메서드에서라면 `List<SetObserver<E>>`를 복사해 쓰면 락 없이도 안전하게 순회할 수 있다. 이 방식을 사용하면 앞선 예제(예외 발생과 교착상태)의 증상들이 사라진다.

``` java
// 명시적으로 동기화한 곳이 사라졌다.
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }
	
    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(observer);
    }
	
    public void notifyElementAdded(E element) {
        for (SetObserver<E> observer : observers) {
            observers.added(this, element);
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```

### 2. CopyOnWriteArrayList 사용하기

``` java
// 명시적으로 동기화한 곳이 사라졌다.
public class ObservableSet<E> extends ForwardingSet<E> {

    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserser<E>> observers = new CopyOnWriteArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        observers.add(observer);
    }
	
    public boolean removeObserver(SetObserver<E> observer) {
        return observers.remove(observer);
    }
	
    public void notifyElementAdded(E element) {
        for (SetObserver<E> observer : observers) {
            observers.added(this, element);
        }
    }
    
    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if(added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); //notifyElementAdded를 호출
        }
        return result;
    }
}
```

이렇게 하면 내부를 변경하는 작업은 복사본을 만들어 수행하고, 내부 배열은 절대 수정되지 않기 때문에 `Lock`을 사용할 필요가 없어 성능도 매우 빨라진다.

___

## 동기화의 정확성과 효율성 개선

외계인 메서드는 얼마나 오래 실행될지 알 수 없기 때문에 동기화 영역에서 호출된다면 그동안 다른 스레드는 보호된 자원을 사용하지 못하고 대기해야 한다.

동기화 블록 바깥으로 외계인 메서드 옮기기 예제처럼 동기화 영역 바깥에서 호출되는 외계인 메서드를 열린 호출(open call)이라 하고, **열린 호출은 실패 방지 효과 외에도 동시성 효율을 크게 개선시켜준다.**

___

### 동기화의 기본 규칙

동기화 영역에서는 가능한 한 일을 적게 하는 것이 기본 규칙이다. `Lock`을 얻고, 공유 데이터를 검사하고, 필요하면 수정하고, `Lock`을 놓는다.

만약 오래 걸리는 작업이라면 동기화 영역 바깥으로 옮기는 방법을 찾아보자.

### 동기화의 성능 개선

멀티코어가 일반화된 오늘날, 병렬로 실행할 기회를 잃고 모든 코어가 메모리를 일관되게 보기 위한 지연시간이 동기화의 진짜 비용이라고 할 수 있다.

_**가변 클래스를 작성한다면 아래의 두 개의 선택지 중 하나를 따르자.**_

1. 동기화를 전혀 고려하지 말고, 사용하는 클래스가 외부에서 알아서 동기화하게 하자.

2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. (단, 클라이언트가 외부에서 전체에 락을 거는 것보다 효율성이 좋을 때만)

Java의 라이브러리 중 `java.util`은 첫 번째 방법을, `java.util.concurrent`는 두 번째 방법을 택했다. 만약 선택하기 어렵다면 동기화하지 말고 "스레드 안전하지 않다."라고 명시하자.

#### 클래스 내부에서 동기화 하는 경우

락 분할, 락 스트라이핑, 비차단 동시성 제어 등 다양한 기법을 동원해 동시성을 높일 수 있다.

1. 락 분할(Lock Splitting) : 하나의 클래스에서 기능적으로 락을 분리해서 사용하는 것(읽기전용, 쓰기전용)

2. 락 스트라이핑(Lock Striping) : 자료구조 관점에서 한 자료구조 전체가 아닌 일부분에 락을 적용하는 것

3. 비차단 동시성 제어(NonBlocking Concurrency Control) : [링크텍스트](https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/)

[각각에 대한 예시](https://github.com/Java-Bom/ReadingRecord/issues/141)

___

## 정리

자바의 동기화 비용은 바르게 낮아져왔다. 하지만 과도환 동기화를 피하는 일은 오히려 과거 어느 때보다 중요하다.

과도한 동기화가 초래하는 진짜 비용은 락을 얻는데 드는 CPU 시간이 아닌, **'경쟁하느라 낭비하는 시간, 즉 병렬로 실행할 기회를 잃고, 모든 코어가 메모리를 일관되게 보기 위한 지연시간'**이 진짜 비용이다. 

_가변 클래스를 작성하려거든 다음 두 선택지 중 하나를 따르자._

1. 동기화를 전혀 하지 말고, 그 클래스를 동시에 사용해야하는 클래스가 외부에서 알아서 동기화하게 하자. ex) `java.util`

2. 동기화를 내부에서 수행해 스레드 안전한 클래스로 만들자. ex) `java.util.concurrent`

단, 클라이언트가 외부에서 객체 전체에 락을 거는 것보다 동시성을 월등히 개선할 수 있을 때만 2번의 방법을 선택해야한다.

여러 스레드가 호출할 가능성이 있는 메서드가 정적 필드를 수정한다면 그 필드를 사용하기 전에 반드시 동기화를 해야한다. 

클라이언트가 여러 스레드로 복제돼 구동되는 상황이라면 다른 클라이언트에서 이 메서드를 호출하는걸 막을 수 없으니 외부에서 동기화할 방법이 없다.

결과적으로, 이 정적 필드가 `private`여도 서로 관련 없는 스레드들이 동시에 읽고 수정할 수 있게 되어, 사실상 전역변수와 동일해진다는 뜻이다. 
