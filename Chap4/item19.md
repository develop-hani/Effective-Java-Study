## CH4 클래스와 인터페이스
> +  _추상화의 기본 단위인 클래스와 인터페이스는 자바 언어의 심장과도 같다_
+ _자바 언어의 클래스와 인터페이스 설계에 사용하는 강력한 요소를 적절히 활용하여 쓰기 편하고, 견고하며, 유연한 클래스와 인터페이스 만드는 방법을 안내한다_

### 아이템 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 상속을 고려한 설계

item 18에서 상속에서 어떤 문제가 생길 수 있는지 알아봤다. 그럼에도 불구하고 상속을 사용하려면, 상속용 부모클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용)에 대해 문서로 남겨야 한다. 어떤 순서로 호출하고, 각각의 호출 결과가 이어지는 처리에 어떤 영향을 주는지도 담아야 한다. 

### Implementation Requirements

API 문서의 메서드 설명에서 `Implementation Requirements` 라는 절이 있다. 메서드 주석에 `@implSpec` 태그를 붙이면 자바독 도구가 생성해준다. 여기에 재정의 했을 때 발생할 수 있는 내용을 담는 경우가 많다.

`java.util.AbstractCollection`
> `public boolen remove(Onject 0)`
이 컬렉션 안에 `Object.equls(o,e)`가 참인 원소 e가 하나 이상 있다면 그 중 하나를 제거한다.
주어진 원소가 컬렉션 안에 있었다면 `true`를 반환한다.
`Implementation Requirements`메서드는 컬렉션을 순회하며 주어진 원소를 찾도록 구현되어 있다. 주어진 원소를 찾으면 반복자의 `remove` 메서드를 사용해 컬렉션에서 제거한다. 이 커렉션이 주어진 객체를 갖고 있으나, 이 컬렉션의 `iterator` 메서드가 반환한 반복자가 `remove` 메서드를 구현하지 않았다면 `UnsupportedOperationException`을 던지니 주의해야 한다.

위와 같은 설명은 `iterator` 메서드를 재정의 하면 `remove` 메서드의 동작에 영향을 줌을 확실히 알 수 있다. 
하지만 좋은 API문서는 어떻게 하는지가 아니라 무엇을 하는지를 적어야 한다. 상속이 캡슐화를 해치기 때문에 안전하게 상속할 수 있게하기 위한 어쩔 수 없는 선택이다.

### hook을 만들기

효율적인 하위 클래스를 만들 수 있게 하려면 클래스 내부 동작 과정에 끼어들 수 있는 훅(hook) 메서드를 잘 선별하여 protected 메서드 형태로 공개해야 할 수도 있다.

![](https://velog.velcdn.com/images/lcy923/post/bc204024-0fca-411d-a8ec-f23e71e330aa/image.png)

표시된 주석을 보면 `removeRange` 메서드는 이 리스트 또는 부분 리스트의 `clear` 메서드에서 호출한다고 나와있다. 또한 리스트 구현의 내부 구조의 이점을 잘 활용하여 `removeRange` 메서드를 재정의하면 이 리스트 또는 부분 리스트의 `clear` 메서드 성능을 향상 시킬 수 있다 라고 나와있다. 

이 메서드를 제공한 이유는 단지 하위 클래스에서 부분리스트의 `clear` 메서드를 고성능으로 만들기 쉽게 하기 위해서이다. 

protected 메서드와 필드는 공개 API이기 때문에 영원히 책임져야 한다. 상속용 클래스를 시험하는 방법은 직접 하위 클래스를 만들어보는 것이 유일하다. 또한 상속용 클래스는 배포 전에 반드시 하위 클래스를 만들어 검증해야한다. 놓친 `protected` 멤버는 검증 도중 빈자리가 확연히 드러날 것이고 반대로 여러 하위 클래스를 만들면서 전혀 쓰이지 않는 `protected` 멤버는 `private`이었야 할 가능성이 크다.

### 상속용 클래스의 생성자는 재정의 가능한 메서드(non-private, non-final, non-static)를 호출하면 안 된다

상위 클래스의 생성자가 하위 클래스의 생성자보다 먼저 실행되므로, 하위 클래스에서 재정의 해버린 메서드가 하위 클래스 생성자보다 먼저 호출된다. 이 때 하위 생성자에서 초기화 하는 값에 의존한다면 의도대로 동작하지 않을 것이다.

``` java
public class Super {
	// 잘못된 예-생성자가 재정의 가능 메서드를 호출
	public Super() {
		overrideMe();
	}
	
	public void overrideMe() {
    	System.out.println("super method");
	}
}

// overrideMe재정의, 상위 클래스 생성자가 호출해 오동작 일으키는 메서드

public final class Sub extends Super {
	// 초기화되지 않은 final 필드. 생성자에서 초기화
	private String str;
    public Sub() {
        str = "Sub String";
    }

	// 재정의 가능 메서드. 상위 클래스의 생성자가 호출
	@Override
    public void overrideMe() {
        System.out.println(str);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
    }
}
```
하위 클래스의 생성자 보다 상위 클래스의 생성자가 먼저 호출되는데, 상위 클래스의 생성자에서 하위 클래스의 재정의 된 메서드를 호출 하여 String 값이 초기화가 되기도 전에 접근하여 null이 출력이 되었다. 이와 같은 현상은 `Cloneable`의 `clone`과 `Serializable`의 `readObject`에서도 발생한다. 이는 두 메서드가 생성자와 비슷한 새로운 객체를 생성하는 효과를 가지기 때문이다. 

다음 코드는 상위 클래스의 `readObject`에서 재정의 가능한 메서드를 호출하는 코드이다.
```java 
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;

public class SerializableFoo implements Serializable {

    private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException{
        ois.defaultReadObject();
        overrideMe();
    }
    public void overrideMe() {
        System.out.println("This is SuperFoo's overrideMe");
    }
}

public class SerializableSubFoo extends SerializableFoo {
    String str;

    public void setStr(String str) {
        this.str = str;
    }

    @Override
    public void overrideMe() {
        System.out.println("This is SubFoo's overrideMe");
        if (str == null) {
            throw new NullPointerException();
        }
        System.out.println(str);
    }
}
```
``` java
class SerializableSubFooTest {


    @DisplayName("역직렬화시 readObject가 재정의된 메소드 호출")
    @Test
    void name() throws IOException, ClassNotFoundException{
        SerializableSubFoo subFoo = new SerializableSubFoo();
        subFoo.setStr("hi!!!!");

        //직렬화
        byte[] serializedFoo;
        try (ByteArrayOutputStream baos = new ByteArrayOutputStream()) {
            try (ObjectOutputStream oos = new ObjectOutputStream(baos)) {
                oos.writeObject(subFoo);
                serializedFoo = baos.toByteArray();
            }
        }

        // 역직렬화
        assertThatThrownBy(() -> {
            byte[] deserializedMember = Base64.getDecoder().decode(Base64.getEncoder().encodeToString(serializedFoo));
            try (ByteArrayInputStream bais = new ByteArrayInputStream(deserializedMember)) {
                try (ObjectInputStream ois = new ObjectInputStream(bais)) {
                    Object objectMember = ois.readObject();
                    SerializableSubFoo deserialized = (SerializableSubFoo) objectMember;
                    assertSame(deserialized,subFoo);
                }
            }
        }).isInstanceOf(NullPointerException.class);
    }
}
```

하위 클래스를 역직렬화 할때 상위 클래스의 `readObject`가 호출될 때 하위 클래스의 `overrideMe` 메서드를 호출하여 `NullPointerException`을 던진다.

`Cloneable`과 `Serializable` 인터페이스는 상속용 설계를 어렵게 한다. 둘 중 하나라도 구현한 클래스를 상속할 수 있게 설계하는 것은 일반적으로 좋지 않은 생각이다. `clone`과 `readObejct` 메서드도 생성자 처럼 기능할 수 있기 때문에, 재정의한 메서드를 호출해서는 안 된다. 특히 `clone` 메서드는 원본 객체한테까지 피해을 줄 수 있으므로 조심해야 한다.

`Serializable`을 구현한 상속용 클래스가 `readResolve`나 `writeReplace` 메서드를 갖는다면 반드시 `protected` 로 선언해야 한다.

### 지켜야 할 원칙

상속을 고려하지 않은 구체클래스는 상속을 금지하자. `final` class로 만들거나, 생성자를 `private` 나 `package-private`로 선언하고 정적 팩토리 메서드를 활용하자.

상속을 통한 확장보다는, 핵심 기능을 정의한 인터페이스가 있고 클래스가 그 인터페이스를 구현하도록 하자. `List`, `Set`, `Map`이 좋은 사례이다.

인터페이스를 구현하지 않은 구현체의 경우 이런 제약이 불편하기 때문에, 재정의 가능한 메서드를 줄이기 위해 자기사용하는 `public` 메서드를 `private` 메서드로 대체하자. 

클래스의 동작을 유지하면서 재정의 가능 메서드를 사용하는 코드를 제거해야할 때는 `private` '도우미 메서드'를 만들어 재정의 가능 메서드의 기능을 옮기고 도우미 메서드를 호출하도록 수정한다. 

```java

public class Super {
    public Super() {
//      overrideMe();
        helpMethod();
    }

    public void overrideMe() {
        helpMethod();
    }

    //도우미 메서드
    private void helpMethod() {
        System.out.println("super method");
    }
}
public class Sub extends Super{
    private String str;
    public Sub() {
        str = "Sub String";
    }

    @Override
    public void overrideMe() {
        System.out.println(str);
    }


    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```
