# 지연 초기화는 신중히 사용하라


## 지연 초기화

> 지연 초기화: 필드의 초기화 시점을 그 값이 처음 필요할때까지 늦추는 기법

- 지연 초기화는 정적 필드와 인스턴스 필드 모두에 사용할 수 있다.
- 지연 초기화는 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

### 지연 초기화 성능

- 지연 초기화는 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만<br>
그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다.

- 지연 초기화는 초기화 빈도와 비용에 따라 성능을 느려질 수도 있다.

### 지연 초기화가 필요한 경우

- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮고, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 좋을 수 있다.<br>
&rarr; 하지만 성능은 직접 측정해 봐야 한다.

## 멀티스레드 환경에서의 초기화 방법

- 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.<br>
&rarr; 그렇지 않으면 버그가 발생한다.


### 일반적인 초기화 방법

**대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.**

```java
// 인스턴스 필드를 초기화하는 일반적인 방법
private final FieldType field = computeFieldValue();
```

### synchronized 접근자를 활용한 지연 초기화 방법

**지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 synchronized를 단 접근자를 사용해라.**

```java
// 인스턴스 필드의 지연 초기화 -sychronized 접근자 방식
private FieldType field;

private synchronized FieldType getField() {
  if (field == null)
    field = computeFieldValue();
  return field;
}

```

- 위 두 방법은 정적 필드에도 똑같이 적용된다.


### 성능을 위한 정적 필드 지연 초기화

- **성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.**

```java
// 정적 필드용 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

//getField가 처음 호출될 때 FieldHolder가 초기화된다.
private static FieldType getField() { return FieldHolder.field; } 
```

- 클래스는 클래스가 처음 쓰일 때 비로소 초기화된다는 특성을 이용한 관용구이다.
- getField 메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다.




 
### 성능을 위한 인스턴스 필드 지연 초기화

 - 일반적인 VM은 클래스를 초기화 할때만 필드 접근을 동기화한다.
 클래스 초기화가 끝난 후에는 VM이 동기화 코드를 제거하여, 그다음부터는 아무런 검사나 동기화 없이 필드에 접근하게 된다.

- **성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라.**

```java
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result != null)	// 첫 번째 검사 (락 사용 안 함)
    return result;
        
  synchronized(this) {
    if (field == null)
      field = computeFieldValue();	// 두번째 검사 (락 사용)
    return field;
  }

}
```
- 초기화된 필드에 접근할 때의 동기화 비용을 줄여준다.
- 두 번째 검사할 때 초기화되지 않았을 때만 필드를 초기화한다.
- 필드가 초기화된 후로는 동기화하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다.
- result를 사용해 필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장해준다.
&rarr; 성능을 높여줌.


### 반복해서 초기화해도 상관없는 인스턴스 필드 초기화

```java
//단일검사 관용구 - 초기화가 중복해서 일어날 수 있다.
private volatile FieldType field;

private FieldType getField() {
  FieldType result = field;
  if (result == null)
    field = result = computeFieldValue();
  return result;
}

```
- 짜릿한 단일검사의 필드 선언 관용구라고 불린다. 
- 필드 접근 속도를 높여주지지만, 초기화가 스레드당 최대 한 번 더 이뤄질 수 있다.
- 거의 사용하지 않는다.

--- 

- 이 방법들은 모두 기본 타입 필드에도 적용할 수 있다.
