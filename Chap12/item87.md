# 커스텀 직렬화 형태를 고려해보라

클래스가 Serializeable을 구현하고 직렬화 형태를 사용하면,
다음 릴리스 때 버리려 한 현재의 구현에 영원히 발이 묶이게 된다.<br>
&rarr; 충분히 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.

## 기본 직렬화

> 기본 직렬화 형태: 직렬화 하려는 시점의 특정 데이터와 관련된 모든 데이터를 직렬화 하는 것.

- 논리적 데이터: 실제로 객체를 표현하는데 쓰이는 데이터

**객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.**

### 기본 직렬화 형태에 적합한 예

```java
//기본 직렬화 형태에 적합한 후보
public class Name implements Serializable {
    
  private final String lastName;
  private final String firstName;
  private fianl String middleName;
     
 }
```

- 성명은 논리적으로 이름, 성, 중간이름으로 구성되는데 이것을 인스턴스 필드들로 반영했으므로 기본 직렬화 형태에 적합하다.

- **기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.**
---

### 기본 직렬화 형태에 적합하지 않은 예

```java
public final class StringList implements Serializable {
  private int size = 0;
  private Entry head = null;
    
  private static class Entry implements Serializable {
    String data;
    Entry next;
    Entry previous;
    
  }
    
  ...	// 나머지 코드들

}
```

- 논리적으로 이 클래스는 일련의 문자열을 표현한다.
물리적으로는 문자열들을 이중 연결 리스트로 연결했다.
기본 직렬화 형태는 특정 데이터와 관련된 모든 데이터를 직렬화한다.<br>
&rarr; 연결된 모든 정보를 직렬화해야 하므로 기본 직렬화 형태로 적합하지 않다.

---
### 기본 직렬화가 적합하지 않은데 사용하면 생기는 문제점

1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
2. 너무 많은 공간을 차지할 수 있다.
3. 시간이 너무 많이 걸릴 수 있다.
4. 스택 오버플로를 일으킬 수 있다.

---

## 커스텀 직렬화

- 기본 직렬화 형태에 적합하지 않은 형태는 커스텀 직렬화 형태를 갖춘다.
- transient 한정자는 해당 인스턴스 필드가 기본 직렬화 형태에 포함되지 않는다는 표시다.

```java
// 합리적인 커스텀 직렬화 형태를 갖춘 StringList
public final class SringList implements Serializable {
  private transient int size = 0;
  private transient Entry head = null;
    
  //이제는 직렬화되지 않는다. -> transient
  private static class Entry {
    String data;
    Entry next;
    Entry previous;
  }

  public final void add(string s) {...}
    
  private void writeObject(ObjectOutputStream s) 
      throws IOException {
    s.defaultWriteObject();
    s.writeInt(size);
        
    //모든 원소를 올바른 순서로 기록한다.
    for (Entry e = head; e != null; e = e.next)
      s.writeObject(e.data);
  }
    
  private void readObject(ObjectInputStream s)
      throw IOException, ClassNotFoundException {
    s.defaultReadObject();
    int numElements = s.readInt();
        
        //모든 원소를 읽어 이 리스트에 삽입한다.
    for (int i = 0; i < numElements; i++)
      add((String) s.readObject());          
  }
    
    ...나머지 코드는 생략 
}
```

> defaultWriteObject, defaultReadObject: transient로 선언하지 않은 필드를 직렬화, 역직렬화한다.

- 필드 모두가 transient더라도 wirteObject와 readObject는 각각 먼저 defaultWriteObject와 defaultReadObject를 호출한다.<br>
&rarr; 향후 릴리스에 transient가 아닌 인스턴스 필드가 추가되더라도 상호호환된다.


- 기본 직렬화 대신 커스텀 직렬화를 사용하면 성능이 좋아진다.

- 해시 테에블을 사용한 객체는 직렬화 할때마다 해시 값이 달라질 수 있으므로, 기본 직렬화를 사용하면 더 심각한 문제가 발생할 수 있다.

---

### transient

- transient로 선언해도 되는 필드는 모두 transient를 붙여야 한다.
- 해당 객체의 논리적 상태오 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.
- 기본 직렬화를 사용한다면 transient 필드들은 모두 기본값으로 초기화된다.<br>
&rarr; 기본값을 사용하지 않으려면 readObject에서 원하는 값으로 복원해라.


---


## UID의 중요성

- **어떤 직렬화 형태를 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자.**

- UID를 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행한다.<br>
&rarr; 미리 부여해서 성능을 높이자. 무작위로 사용해도 된다.
- 버전이 다른 클래스가 같은 직렬 UID를 사용하면 호환성이 유지된다.
- **구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.**
