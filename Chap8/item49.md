# 49. 매개변수가 유효한지 검사하라

## ⭐ 선요약

메서드나 생성자 생성 시에 매개변수의 제약을 문서화하고 메서드 시작 부분에서 명시적으로 검사하라

메서드와 생성자는 매개변수의 값을 **몸체가 실행되기 전**에 확인한다.

### public, protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화해야 한다.

```java
/**
* (현재 값 mod m) 값을 반환한다. 이 메서드는 
* 항상 음이 아닌 BigInteger를 반환한다는 점에서 remainder 메서드와 다르다.
*
* @param m 계수 (앵수여야 한다.)
* @return 현재 값 mod m
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
*/
public BigInteger mod(BigInteger m) {
    if (m.signum() < 0)
        throw new ArithmeticException("계수(m)는 양수여야 합니다. " + m);
    ...
}
```

위 코드의 주석에는 ArithmeticExceiption에 대해서는 나와있지만, NullPointerException에 대해서는 나와있지 않다.

⇒ 그 클래스의 모든 public 메서드에 적용되는 사항은 클래스 수준의 주석으로 작성하는 것이 훨씬 깔끔하다.

자바 7에 추가된 `java.util.Objects.requireNonNull` 메서드는 유연하고 사용하기도 편하니, 더 이상 null 검사를 수동으로 하지 않아도 된다.

```java
this.startegy = Objects.requireNonNull(strategy, "전략");
```

---

### 공개되지 않은 멤서드라면 메드가 호출되는 상황을 통제할 수 있고, 단언문(assert)를 사용해 매개변수의 유효성을 검증할 수 있다.

```java
private static void sort(long[] a, int offset, int length) {
    assert a != null;
    assert offset >= 0 && offset <= a.length;
    assert length >= 0 && length <= a.length - offset;
    ...
}
```

**단언문이 일반적인 유효성 검사와 다른 점**

- 실패하면 `AsserrtionError`를 던진다.
- 런타임에 아무런 효과도, 아무런 성능저하도 없다.

---

### 예외 상황

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때
- 계산 과정에서 암묵적으로 검사가 수행될 때
