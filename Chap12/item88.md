# readObject 메서드는 방어적으로 작성하라

```java

public final class Period {
  private final Date start;
  private final Date end;

  public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end   = new Date(end.getTime());

    if (this.start.compareTo(this.end) > 0)
        throw new IllegalArgumentException(
      this.start + "가 " + this.end + "보다 늦다.");
   }

  public Date start() { return new Date(start.getTime()); }
  public Date end() { return new Date(end.getTime()); }
    // 나머지 코드 생략
}

```

- Period를 직렬화할 때 불변식을 더는 보장 못한다.<br>
&rarr; readObject 메서드가 또 다른 public 생성자이기 때문이다.

- readObject를 생성자와 같이 주의를 기울여서 만들지 않으면 공격자는 아주 손쉽게 해당 클래스의 불변식을 깨뜨릴 수 있다.

- readObject는 매개변수로 바이트 스트림을 받는 생성자이다.<br>
&rarr; 공격자가 불변식을 깨기 위한 바이트 스트림을 건네면 문제가 된다.

---

## 직렬화에서 불변식을 보장하기 위한 방법

### 유효성 검사 - 아직 부족!

- 공격자의 바이트 스트림으로부터 보호하기 위해서는 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다.

```java
//유효성 검사를 수행하는 readObject 메서드 - 아직 부족하다!
private void readObject(ObjectInputStream s) 
    throw IOException, ClassNotFoundException {
        
  s.defaultReadObject();
    
    //불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

- 유효성을 검사하면 공격자가 혀용되지 않는 Period 인스턴스를 생성하는 일을 막을 수 있지만, 아직 문제가 있다.

- 정상 Period 인스턴스에서 시작된 바이트 스트림 끝에 private Data 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.

- 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 '악의적인 객체 참조'를 읽어 Period 객체의 내부 정보를 얻을 수 있다.


### 방어적 복사

- '악의적인 객체 참조'를 읽어 객체의 내부 정보를 얻어 불변식을 가변식으로 바꾸는 것을 막기 위해서는 방어적 복사를 해야 한다.
- **객체는 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.**<br>
&rarr; readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

```java
private void readObject(ObjectInputStream s) 
    throws IOException, ClassNotFoundException {
	
  s.defaultReadObject();
    
  // 가변 요소들을 방어적으로 복사한다.
  start = new Date(start.getTime());
  end = new Date(end.getTime());
    
  //불변식을 만족하는지 검사한다.
  if (start.compareTo(end) > 0)
    throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}

```

- 방어적 복사는 유효성 검사보다 앞서 수행한다.
- final 필드는 방어적 복사가 불가능하다.<br>
&rarr; 공격 위험에 노출되는 것 보다 낫다.

---
## readObject 메서드를 써도 좋을지를 판단하는 방법

- transient 필드를 제외한 모든 필드의 값을 매개변수로 받아 유효성 검사 없이 필드에 대입하는 public 생성자를 추가해도 괜찮은지 확인한다.<br>
&rarr; 괜찮지 않다면 readObject 메서드를 만들어 모든 유효성 검사와 방어적 복사를 수행하거나 직렬화 프록시 패턴을 사용한다.

- readObject 메서드도 생성자처럼 재정의 가능 메서드를 호출해서는 안 된다.

---
결론: 생성자를 통해 필드 값을 불변을 만드는 것과 똑같이 readObject도 유효성 검사와 방어적 복사를 사용해 불변으로 만든다.
