finalizer와 cleaner 사용을 피하라
=
### finalize 메서드
객체가 소멸될때 호출 되기로 약속 한 메서드

## 자바가 제공하는 두 가지 객체 소멸자
### finalizer
- 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요
### cleaner
- 자바9에서 finalizer를 사용자제 API로 지정하고 그 대안으로 소개한 객체 소멸자
- finalizer보다 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요
>C++에서 특정객체와 관련된 자원을 회수하는 파괴자와는 다른 개념

## finalizer와 cleaner의 단점
### 단점1. 즉시 수행된다는 보장이 없다.
- 객체에 접근할 수 없게 된 후 finalizer나 cleaner가 실행되기까지 얼마나 걸릴지 알 수 없음\
⇒finalizer와 cleaner로는 제때 실행되어야 하는 작업은 할 수 없다.

- finalizer나 cleaner가 언제 실행되는가는 가비지 컬렉터 알고리즘에 달렸으며, 이는 GC의 구현마다 천차만별
⇒제공자가 테스트한 JVM에서는 제대로 작동하던 프로그램이 클라이언트의 시스템에서는 문제가 발생할 수도 있다

- 클래스에 finalizer를 달아두면 그 인스턴스의 자원 회수가 제멋대로 지연될 수 있다\
 -finalizer 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아서 실행될 기회를 제대로 얻지 못하기 때문
 -cleaner는 자신이 수행될 스레드를 제어할 수 있다는 점에서 조금 낫지만 즉각 수행되리나는 보장은 역시 없다.
### 단점2. 수행 여부조차 보장이 안된다.
접근할 수 없는 일부 객체에 딸린 종료 작업을 전혀 수행하지 못한채 프로그램이 중단될수 있다.
⇒프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안된다
>System.gc나 System.runFinalization 메서드들이 finalizer와 cleaner가 실행될 가능성을 높여줄 수는 있지만 보장해주진 않는다.
>System.runFinalizerOnExit와 Runtime.runFinalizerOnExit이 finalizer와 cleaner의 실행을 보장해주기는 하지만 ThreadStop이라는 심각한 결함이 있다.
### 단점3. finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
잡지 못한 예외 때문에 해당 객체는 자칫 마무리가 덜 된 상태로 남을 수 있다.
→스레드가 그러한 객체를 사용하려 한다면 어떻게 동작할지 예측조차 할 수 없다.
(보통의 경우엔 잡지 못한 예외가 스레드를 중단시키고 스택 추적 내용을 출력하지만 finalizer 실행 중 같은 일이 일어나면 경고조차 출력하지 않는다.)\
cleaner는 자신의 스레드를 통제하기 때문에 이런 문제는 발생하지 않는다.
### 단점4. 심각한 성능 문제
finalizer를 사용하여 객체를 생성하고 파괴하면 try-with-resources로 자신을 닫도록 한 AutoCloeseable 객체를 생성하고 가비지 컬렉터가 수거하는데 비해 50배 가까이 느리다.
cleaner 또한 클래스의 모든 인스턴스를 수거하는 형태로 사용하면 성능은 finalizer와 비슷한 정도이다.

안전망 형태로만 사용하면(메인이 아닌 보조 역할) 5배 느려진 속도(그냥 사용했을때보다 10배정도 빠른)로 사용이 가능하다.
### 단점5. finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
[finalizer attack](https://yangbongsoo.tistory.com/8) 이 일어나면 GC에 수집되어야 할 객체가 GC에 수집되지 않게 되어 여러가지 문제를 일으킬 수 있다.(링크 참고)

------
## finalizer와 cleaner의 대안은 뭐가 있을까?
AutoCloseable을 구현해주고 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출하면 된다.(일반적으로 예외가 발생해도 종료되도록 try-with-resources(아이템 9)를 사용한다.)
>close 메서드에서 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드가 이 필드를 검사해 객체가 닫힌 후 불렸다면 IllegalStateException을 던지게 하는 식으로 각 인스턴스는 자신이 닫혔는지 추적하도록 하는 것이 좋다,
## finalizer와 cleaner 쓸데가 있긴 할까?
### 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할
finalizer나 cleaner가 즉시 호출되리라는 보장은 없지만 자원회수를 늦게라도 하는것이 아예 안하는 것보다는 낫기 때문에 사용한다. 
이러한 안전망 형태로 사용할때는 그럴만한 값어치가 있는지 잘 생각하고 사용하자.
>FileInputStream, FileOutputStream, ThreadPoolExecutor와 같은 몇몇 자바 클래스는 안전망 역할의 finalizer를 제공한다.
## 네이티브(native peer)와 연결된 객체
>### 네이티브 피어 : 일반 자바 객체가 네이티브 메서드(C/C++ 코드를 자바에서 이용할 수 있는 방법)를 통해 기능을 위임한 네이티브 객체
이러한 네이티브 피어는 자바 객체가 아니니 가비지 컬렉테가 그 존재를 알지 못하기 때문에 finalizer나 cleaner를 사용해서 처리할 수 있다.\
  -단, 성능저하를 감수할 수 있고 네이티브 피어가 즉시 회수되어야 할만한 심각한 자원을 가지고 있지 않는 경우에만 사용한다.\
  -이런 조건에 부합하지 않는다면 위에서 설명한 close메서드를 사용한다.
## cleaner 사용법
다음은 방(Room) 자원을 수거하기 전에 반드시 청소(clean)해야 한다고 가정한 예시 클래스이다. Room 클래스는 AutoCloseable을 구현한다.
 ```java
public class Room implements AutoCloseable {
  private static final Cleaner cleaner = Cleaner.create();
  // 청소가 필요한 자원. 절대 Room을 참조해서는 안 된다!
  // 순환참조가 생겨 GC가 Room 인스턴스를 회수해갈 기회가 오지 않는다.
  private static class State implements Runnable {//정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖기 때문에 static으로 선언
      int numJunkPiles; // 방(Room) 안의 쓰레기 수
      State(int numJunkPiles) {
          this.numJunkPiles = numJunkPiles;
      }
      // close 메서드나 cleaner가 호출한다.
      @Override public void run() {
          System.out.println ("방 청소");
          numJunkPiles = 0;
      }
  }
  // 방의 상태. cleanable과 공유한다.
  private final State state;
  // cleanable 객체. 수거 대상이 되면 방을 청소한다.
  private final Cleaner.Cleanable cleanable;
  public Room(int numJunkPiles) {
      state = new State(numJunkPiles);
      cleanable = cleaner.register(this, state);
  }
  @Override public void close() {
  cleanable.clean();
  }
}
```
static으로 선언된 중첩 클래스인 State는 Runnable을 구현하고 그 안의 run 메서드는 cleanable에 의해 딱 한번만 호출 될 것이다. 
이 cleanable 객체는 Room 생성자에서 cleaner에 Room과 State를 등록할 때 얻는다. 
Room의 close 메서드가 호출될때, 혹은 GC가 Room을 회수할 때까지 클라이언트가 close를 호출하지 않는다면 cleaner가 (안전망 역할로) run 메서드를 호출할 것이다.

만약 클라이언트가 모든 방 생성을 try-with-resources 블록으로 감쌌다면 자동청소는 필요하지 않다. 다음이 그 예이다.
```java
public class Adult {
    public static void main(String[] args) {
        try(Room myRoom = new Room(7)){
            System.out.println("안녕~");
        }
    }
}
```
>Adult 프로그램은 "안녕~"을 출력한 후, 이이서 "방 청소"를 출력한다
다음은 결코 방 청소를 하지 않는 프로그램이다.
```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("아무렴~");
    }
}
```
>이 프로그램에서 "아무렴"에 이어 "방 청소"는 무조건 출력되지는 않는다. 앞서 말한 예측할 수 없다고 한 상황이다.
