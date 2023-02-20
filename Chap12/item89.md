## 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

### 싱글턴

- 인스턴스를 오직 하나만 생성하도록 하는 방법
- 하나의 인스턴스를 갖기 때문에 상태 값을 갖지 않는 무상태 객체 또는 프로젝트 설계상 유일한 인스턴스로 컴포넌트 역할을 하는 클래스가 적당하다.

### 싱글턴을 만드는 세 가지 방법

1. 생성자의 접근 제한자를 `private`으로 설정하여 외부에서 객체를 생성하지 못하도록 하고, 인스턴스에 접근하는 메서드를 `public static final`로 제공하는 방법

2. 위와 동일하게 생성자의 접근 제한자를 `private`으로 설정, 외부에서 접근하는 방식을 정적 팩토리 메서드를 `public static` 멤버로 제공

3. 원소가 하나인 열거 타입을 선언

### 싱글턴을 직렬화하는 방법

- `Serializable` 구현하는 것만 으로는 싱글턴 직렬화를 유지할 수 없다.
- 모든 인스턴스 필드를 일시적(`transient`)라고 선언 후, `readResolve` 메서드를 제공해야 한다.

___

## readResolve() 가 동작하는 방식


![](https://velog.velcdn.com/images/lcy923/post/e3ea71b1-8971-4c19-b69c-790c230b0496/image.png)

### readResolve : 역직렬화 후 생성된 객체를 싱글턴으로 유지하는 방법

1. 역직렬화 한 새로 생성된 객체를 인수로 `readResolve()`를 호출

2. 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환
	- 이미 존재하는 인스턴스가 존재하는 경우, 기존 인스턴스 반환
    - 인스턴스가 없으면 새로운 인스턴스 생성하여 반환
    
3. 새로 생성된 객체의 참조는 유지하지 않으므로 바로 가비지 컬렉션 대상이 된다

#### 싱글턴 클래스 : 해당 클래스는 데이터를 갖고 있지 않아 직렬화 형태를 제공할 필요가 없다.

``` java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    
    public void leaveTheBuilding() {...}
}
```
#### Serializable 구현한 싱글턴 클래스

- 해당 클래스는 `Serializable` 인터페이스를 구현하였지만 의도한 대로 싱글턴 직렬화를 기대할 수 없다.

- `favoriteSongs`라는 데이터를 갖는 필드가 존재하기 때문에 모든 인스턴스 필드를 `transient`로 선언해야할 필요가 있다.

- `readResolve`를 인스턴스 통제 목적으로 사용하는 경우, 객체 참조 타입 인스턴스 필드는 모두 `transient`를 선언해야 한다.

``` java
public class Elvis implements Serializable {

	public static final Elvis INSTANCE = new ELVIS();
    
    private String[] favoriteSongs = {"Hound Dong", "Heartbreak Hotel"};
    
    private Elvis(){}
    
    public void printFavorites() {
    	System.out.println(Arrays.toString(favoriteSongs));
    }
}
```

#### Serializable 구현 및 readResolve 메서드를 제공하는 싱글턴 클래스

- `Serializable` 인터페이스를 구현하고, `readResolve()`메서드를 제공하여 싱글턴 직렬화를 기대할 수 있다.

- `readResolve()` 는 기존에 인스턴스가 존재하는 경우, 기존의 인스턴스를 반환하고 역직렬화한 객체는 무시한다.

- 그러나 아직 `favoriteSongs` 필드에 대한 `transient` 처리가 되지 않았기 때문에 역직렬화 시 공격에는 취약한 상태이다.

``` java
public class Elvis implements Serializable {

	public static final Elvis INSTANCE = new Elvis;
    
    private Elvis() {}
    
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
    
    public void printFavorites(){
    	System.out.println(Arrays.toString(favoriteSongs));
    }
    
    private Object readResolve() {
    	return INSTANCE;
    }
}
```
___
### 역직렬화의 취약점

바로 위의 코드와 같이 싱글턴이 `non-transient` 참조 필드를 갖고 있는 경우, 
 - `readResolve` 메서드가 실행되기 전에 역직렬화 된다.
 - 그렇게 되면 '잘 조작된 스트림'을 써서 해당 참조 필드의 내용이 역직렬화 되는 시저멩 그 역직렬화된 인스턴스의 참조를 훔쳐올 수 있다.
 
#### Stealer 클래스 : 참조 필드를 훔치는 방식으로 공격하는 클래스
![](https://velog.velcdn.com/images/lcy923/post/19d29cd7-004b-4c50-9db6-43c94d27e9de/image.png)

```
main 함수 출력 결과

[Hound Dog, Heartbreak Hotel]
[A Fool Such as I]
```
출력 결과를 보아 Elvis는 싱글톤으로 설계했는데 다른 인스턴스가 된 걸 확인 가능하다.

이유는 다음과 같다.

1. `private String [] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};`라는 `non-transient` 참조 필드가 있다.

2. 직렬화된 스트림에서 위 필드를 이 도둑 클래스의 인스턴스로 교체한다.

3. 바이트 코드 자체에서 싱글턴(`Elvis`)의 비휘발성 필드(`String [] favoriteSongs`) 부분을 도둑의 인스턴스(`ElvisStealer`)로 교체한다.

4. 바이트 코드를 바꾸면서 `Elvis(new).favoriteSongs = ElvisStealer(new)`가 되어있는 상황에서 `readResolve`를 호출하게 되면 `Elvis(new).favoriteSongs = ElvisStealer(new).readResolve()` 와 같은 흐름으로 코드가 실행이 된다.

5. 따라서 `Elvis` 역직렬화를 할 때 `String[] favoriteSong`을 역직렬화 하려고 보니 `ElvisStealer`로 참조가 걸려있어서 `EvlisStealer로 readResolve` 코드를 호출하게 된다.

6. 결국 `ElvisStealer`의 `readResolve`에서 반환하는 `String` 이 반환되는 구조이다.

___


### 싱글턴을 보장하는 열거 타입 클래스

```java
public enum ElvisType {
	
    INSTANCE; // 선언한 상수 이외에는 다른 객체는 존재하지 않음을 보장해준다.
    
    private String{} favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
    
    public void pringFavorites() {
    	System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
`enum`을 사용하면 모든 것이 해결된다. 자바가 선언한 상수 외에 다른 객체가 없음을 보장해주기 때문이다. 물론 `AccessibleObject.setAccessible` 메서드와 같은 리플렉션을 사용했을 때는 예외다.

물론 인스턴스 통제를 위해 `readResolve` 메서드를 사용하는 것이 중요할 때도 있다. 직렬화 가능 인스턴스 통제 클래스를 작성해야 할 때, 컴파일 타임에는 어떤 인스턴스들이 있는지 모를 수 있다. 이 때는 열거 타입으로 표현하는 것이 불가능하기 때문에 `readResolve` 메서드를 사용할 수 밖에 없다.


## 정리

- 불변식을 지키기 위 해 인스턴스를 통제해야 한다면 가능한 열거 타입을 사용하기
- 열거 타입이 안되면 `readResolve` 메서드를 작성해서 넣고 모든 참조 타입 인스턴스 필드를 `transient`로 선언해야 한다

    
    
