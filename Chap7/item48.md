스트림 병렬화는 주의해서 적용하라
=
### 동시성 프로그래밍을 할 때는 안전성과 응답가능 상태를 유지하기 위해 애써야 한다!
## 스트림 병렬화
```java
public class ParallelMersennePrimes {
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel() // 스트림 병렬화
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```
위 코드는 아이템 45의 메르센 소수를 생성하는 프로그램의 성능 향상화를 위해 parallel()함수를 호출(병렬화)하였다.\
성능 향상을 원했지만 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못해 성능이 나빠지다 못해 무한히 계속되게 되었다.\
-> 데이터 소스가 Stream.iterate거나 중간 연산으로 limit를 쓰면 파이프라인 병렬화로는 성능 개선을 기대할 수 없다.\
=> 스트림 파이프라인을 마구잡이로 병렬화하면 성능이 오히려 끔찍하게 나빠질 수 있고, 결과자체가 잘못되거나 예상 못한 동작이 발생할 수 있다.

## 스트림을 병렬화 하면 좋은경우
### 스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap,의 인스턴스거나 배열, int 범위, long, 범위 일때
위 자료구조들의 특징
- 데이터를 원하는 크기로 정확하고 손쉽게 나눌수 있어 일을 다수의 스레드에 분배하기 좋다. (나누는 작업은 Spliterator가 담당)
- 원소들을 순차적으로 실행할 때의 참조 지역성이 뛰어나다. 
(이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻, 참조들이 가리키는 실제 객체가 메모리에서 서로 떨어져 있는 경우는 참조 지역성이 나빠진다.)
### 종단 연산의 동작 방식이 전체 작업에서 차지하는 비율이 낮으며 순차적인 연산이 아닌 경우
- 종단 연산 중 병렬화에 가장 적합한 것은 축소(파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업)이다.
- 가변 축소의 경우 컬렉션들을 합치는 부담이 커 병렬화에 적합하지 않다.
## 병렬화의 이점을 누리기 위해 알아둘 점
- 직접 구현한 Stream, Iterable, Collection이 병렬화의 이점을 제대로 누리게 하려면 sliterator 메서드를 반드시 재정의하고 결과 스트림의 병렬화 가능성을 강도 높게 테스트해야한다,
- 병렬화로 처리 속도가 빨라지더라도 그 정도가 병렬화에 드는 추가 비용을 상쇄 할 수 있어야 성능 향상을 기대할 수 있다. 
(보통 스트림안의 원소 수와 원소당 수행되는 코드 줄 수를 곱한 값이 최소 수십만은 되어야 유의미한 성능 향상이 된다.)
