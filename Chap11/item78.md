## 아이템78. 공유 중인 가변 데이터는 동기화해 사용하라
___
## 동기화(synchronize)

동기화는 배타적 실행 뿐만 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다. 자바에서 메서드나 블록을 한 스레드가 수행하도록 보장하려면 `synchronized` 키워드를 사용하면 된다. 동기화를 제대로 사용하면 어떤 메서드도 객체의 상태가 일관되지 않은 순간을 볼 수 없다. 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종결과를 같게 한다.

___
## CPU-RAM 아키텍쳐와 동시성 프로그래밍에서 발생할 수 있는 문제점

현대 컴퓨터의 `CPU`와 `RAM`의 관계도를 그려보면 다음과 같은 그림이 될 것이다. 설명의 편의성을 위해 `2CPU 2CORE 2THREAD` 모델을 예로 들면,

![](https://velog.velcdn.com/images/lcy923/post/0ca7f994-18a6-4276-ba87-56a4ed07562b/image.png)


`CPU`가 어떤 작업을 처리하기 위해 데이터가 필요할 때, `CPU`는 `RAM`의 일부분을 고속의 저장 장치인 `CPU Cache Memory`로 읽어 들인다.

이 읽어들인 데이터로 명령을 수행하고, `RAM`에 저장하기 위해서는 데이터를 `CPU Cache Memory`에 쓴 다음 `RAM`에 쓰기 작업을 수행한다. 그러나 `CPU`가 캐시에 쓰기 작업을 수행했다고 해서 바로 `RAM`으로 쓰기 작업을 수행하지 않는다. 반대로 읽기 작업도 해당 데이터가 `RAM`에서 변경이 되었다고 해도, 언제 `CPU Cache Memory`가 아닌 `RAM`에서 데이터를 읽어 들여서 `CPU Cache Memory`를 업데이트할 지 보장하지 않는다.

동시성 프로그래밍에서는 `CPU`와 `RAM`의 중간에 위치하는 `CPU Cache Memory`와 병렬성이라는 특징때문에 다수의 스레드가 공유 자원에 접근할 때 두 가지 문제가 발생할 수 있다.

**1. 가시성 문제**

**2. 원자성(동시 접근) 문제**

___
### 가시성(visibility)의 문제와 volatile 키워드

여러 개의 스레드가 사용됨에 따라, `CPU Cache Memory`와 `RAM`의 데이터가 서로 일치하지 않아 생기는 문제를 의미한다. 이를 해결하기 위해서는 가시성이 보장되어야 하는 변수를 `CPU Cache Memory`가 아니라 `RAM`에서 바로 읽도록 보장해야 한다.

이때 변수에 `valatile` 키워드를 붙임으로써 가시성을 보장할 수 있다.

```private static volatile boolean isStop;```

그러나 가시성만 보장된다고 동시성이 보장되는 것은 아니다. 간단한 예를 살펴 보자.
___
_전철 비용을 계산하는 프로그램을 작성한다고 가정하겠다. 이때 나이에 따라서 70세 미만과 70세 이상의 표 값이 다른 상황이고, 날짜를 실시간으로 반영하여 비용을 계산해야 한다._
___
각 스레드의 역할은 다음과 같다.

-**Thread1**
- 고객의 나이를 읽는다
- 읽어온 나이를 기준으로 비용을 계산한다
- 비용을 반환한다

-**Thread2**
- 현재 년도를 지속적으로 읽는다
- 해가 바뀌면 고객의 나이를 계산한다
- 바뀐 고객의 나이를 저장한다

`volatile`을 통해 가시성을 해결했지만, 해가 바뀌는 시점에 문제가 발생한다.

- 나이가 69세인 고객이 계산을 진행할 때, 첫 번째 스레드에서는 RAM에서 나이가 69세임을 가져와 나이에 따라 비용을 계산하는 메소드를 실행한다.

- 그러나 마침 해가 딱 바뀌어 년 바뀜을 계산하는 두 번째 스레드에서 RAM에 해당 고객의 나이를 70살로 수정한다.

- 첫 번째 스레드는 나이가 70세로 바뀐 것을 모르고 69세 기준으로 전철 비용을 계산하여 잘못된 비용을 반환한다.

위 예제에서 알 수 있듯이, 가시성이 보장된다고 동시성이 보장되는 것은 아니다. `volatile` 키워드는 어디까지나 `volatile` 변수를 메인 메모리로부터 읽을 수 있게 해 주는 것이 전부이고, 다른 스레드에 의해 이 값이 언제든 바뀔 수 있다. 즉, 가시성이란 공유 데이터를 읽는 경우의 동시성만 보장하는 것이라 생각하면 된다.

___
``` java
public class StopThread {

    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested)
                i++;
        });

        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
메인 스레드가 1초 후 `stopRequest`를 `true`로 설정하면 `backgroundThread`는 반복문을 빠져나올 것처럼 보일 것이다. 하지만 실행시켜보면 다음의 코드는 영원히 수행될 수도 있다. 
![](https://velog.velcdn.com/images/lcy923/post/2840a3af-d0c4-494c-8d39-3fae7a1a4357/image.png)

`CPU1`에서 수행된 스레드를 `backgroundThread`, `CPU2`에서 수행된 스레드를 `mainThread`라고 하자. `mainThread`는 `CPU Cache Memory 2`와 `RAM`에 공유 변수인 `stopRequested`를 `true`로 쓰기 작업을 완료했으나, `backgroundThread`는 `CPU Cache Memory 1`에서 읽은 여전히 업데이트되지 않은 `stopRequested`값을 사용한다. 이 값은 `false`이므로 무한 루프를 수행하게 된다. 즉, `mainThread`가 수정한 값을 `backgroundThread`가 언제쯤에나 보게 될지 보증할 수 없다. 이러한 문제점을 **가시성의 문제**라고 한다.

이 문제를 해결하기 위해서는 `stopRequested` 변수를 `volatile`로 선언하면 된다. `volatile`로 선언된 변수에 대해서는 다음 그림과 같이 `CPU Cache Memory`를 거치지 않고 `RAM`으로 직접 읽고 쓰는 작업을 수행하게 된다.

![](https://velog.velcdn.com/images/lcy923/post/c75ac64a-2313-4cc2-88ff-cdb6b573e70d/image.png)

___
### 원자성 문제(동시 접근의 문제)와 synchronized 키워드

원자성은 가시성과 멀티 스레드 환경에서 스레드간 공유 메모리 이슈를 발생한다는 점에서 공통점이 있다. 하지만 시스템 관점에서 보면 두 개념은 다르다.

가시성은 `CPU - Cache - Memory` 관계 상의 개념이다.

원자성은 한 줄의 프로그램 문장이 컴파일러에 의해 기계어로 변경되면서, 이를 기계가 순차적으로 처리하기 위한 여러 개의 `Machine Instruction`이 만들어져 실행되기 때문에 일어나는 현상이다.

예를 들어 프로그램 언어적으로 `i++` 문장은 다음과 같은 기계가 수행하는 명령어로 쪼개진다.
- i를 메모리로부터 읽는다.
- 읽은 값에 1을 더한다.
- 연산한 값을 메모리에 저장한다.

멀티 스레드 환경에서는 한 스레드가 각 기계 명령어를 수행하는 동안에 다른 스레드가 개입하여 공유 변수에 접근하여 같은 기계 명령어를 수행할 수 있으므로 값이 꼬이게 된다. (race condition)

___
원자성 문제를 해결하기 위해서는 `synchronized` 또는 `atomic`을 사용해야 한다.

참고로 원자성 문제를 `synchronized` 또는 `atomic`을 통해 해결한다면 가시성의 문제도 해결된다. `synchronized` 블럭을 들어가기 전에 `CPU Cache Memory`와 `Main Memory`를 동기화 해주며, `atomic`의 경우에는 CAS 알고리즘에 의해 원자성 문제와 `CPU Cache Memory`에 잘못된 값을 참조하는 문제를 동시에 해결해주기 때문이다.

___

`volatile`을 사용하면 동기화를 생략해도 된다. 다만 주의해서 사용해야 한다. 아래와 같은 예제에서 문제점을 찾아볼 수 있다.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```
코드상으로 증가 연산자(++)는 하나지만 실제로는 `volatile` 필드에 두 번 접근한다. 먼저 값을 읽고, 그런 다음에 1을 증가한 후 새로운 값을 저장하는 것이다. 따라서 두 번째 스레드가 첫 번째 스레드의 연산 사이에 들어와 공유 필드를 읽게 되면, 첫 번째 스레드와 같은 값을 보게 될 것이다. `volatile` 키워드는 어디까지나 `volatile` 변수를 메인 메모리로부터 읽을 수 있게 해 주는 것이 전부다.

이처럼 잘못된 결과를 계산해내는 오류를 **안전 실패(safety failure)** 라고 한다. 이 문제는 메서드에 `synchronized`를 붙이고 `volatile` 키워드를 공유 필드에서 제거하면 해결된다. 다음의 코드는 `synchrozied`를 이용해 위의 문제를 해결한 수정된 코드이다.

``` java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    increment()
    return nextSerialNumber;
}

private static synchronized void increment() {
        ++nextSerialNumber;
}
```
또한 **`synchrozied`는 가시성의 문제도 해결한다. **`synchronized`블록을 들어가기 전에 `CPU Cache Memory`와 `Main Memory`를 동기화해주기 때문이다.

추가로 `synchrozied`와 `volatile`은 jvm의 최적화 기법을 방지하는 역할을 한다.

___
## 원자적(atomic)

자바 언어의 명세상으로 `long`과 `double` 을 제외한 변수를 읽고 쓰는 것은 원자적이다. 즉, 동기화 없이 여러 스레드가 같은 변수를 수정하더라도 항상 어떤 스레드가 정상적으로 저장한 값을 읽어오는 것을 보장한다는 것이다.

하지만 스레드가 필드를 읽을 때 항상 **수정이 완전히 반영된** 값을 얻는다 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 **보이는가**는 보장하지 않는다. 따라서 원자적 데이터를 쓸 때도 동기화해야 한다.

`atomic` 또한 멀티 스레드 환경에서 원자성을 보장하기 위해 나온 개념이다. 동기화(blockin)가 아닌 CAS(Compared And Swap)라는 알고리즘으로 작동하여 원자성을 보장한다. CAS 알고리즘이란 `volatile`에서 설명했던 `CPU Cache Memory`와 `RAM`을 비교하여 일치한다면 `CPU Cache Memory`와 `RAM`에 적용하고, 일치하지 않는다면 재시도함으로써 어떠한 스레드에서 공유 자원에 읽기/쓰기 작업을 하더라도 원자성을 보장한다. 

아래와 같이 사용한다.

```
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

___
## 정리

이번 아이템에서 언급한 문제들을 피하기 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는것이다. 불변 데이터만 공유하거나 아무것도 공유하지 말자. 다시 말해, 가변 데이터는 단일 스레드에서만 쓰도록 하자. 

한 스레드가 데이터를 다 수정한 후 다른 스레드에 공유할 때는 해당 객체에서 공유하는 부분만 동기화해도 된다. 그러면 그 객체를 다시 수정할 일이 생기기 전까지 다른 스레드들은 동기화 없이 자유롭게 값을 읽어나갈 수 있다. 이런 객체를 사실상 불변이라고 하고, 다른 스레드에 이런 객체를 건네는 행위를 **안전 발행(safe publication)** 이라고 한다.  

객체를 안전하게 발행하는 방법은 많다.

1. 정적 필드

2. volatile 필드

3. final 필드

4. 락을 통해 접근을 하는 필드

5. 동시성 컬렉션에 저장하는 방법
