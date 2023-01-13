# 가변인수는 신중히 사용하라

가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다.<br>
가변인수 메서드를 호출하면, 가장 먼저 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다.
```java
//가변 인수 활용하기 좋은 예
static int sum(int... args) {
  int sum = 0;
  for (int arg : args)
    sum += arg;
  return sum;
}
```

## 인수가 1개 이상이어야 할 때 가변인수 사용 방법

```java
static int min(int firstArg, int ...remainingArgs) {
  int min = firstArg;
  for (int arg : remainArgs)
    if (arg < min)
      min = arg;
  return min;
}
```

첫번째는 평범한 인수를 받고, 두번째부터 가변 인수로 받게 한다.
- 장점
	- 인수를 0개 넣을 경우 컴파일 타임에서 문제를 바로 발견할 수 있다.
	- for-each 문을 사용할 수 있다.

---

## 가변인수 성능 문제

가변인수 메서드는 호출될 때마다 배열을 새로 하나 할당하고 초기화한다.
따라서 성능을 높이기 위한 방법이 필요하다.
- 가변인수 성능을 높이기 위한 패턴
1. 보통 인수의 개수가 적게 사용되는 경우, 인수의 개수가 적게 필요한 메서드는 다중정의를 이용한다.
2. 일정 인수의 개수 이상부터 사용 빈도가 적어지는 부분부터는 가변 인수 메서드를 사용해서 가변 인수 메서드가 배열을 할당하는 경우를 줄인다.

```java
// 예시
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int....rest) {}
