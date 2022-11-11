생성자 대신 정적 팩터리 메서드를 고려하라
=
정적 팩터리 메서드(static factory method)는 클라이언트가 클래스의 인스턴스를 얻을수 있는 생성자 이외의 또다른 방법이다.

클래스는 클라이언트에게 public 생성자 대신(혹은 생성자와 함께) 정적 팩터리 메서드를 제공할수 있다.

정적 팩터리 메서드에는 생성자와 비교했을때 장점과 단점이 모두 존재한다.

장점1. 이름을 가질수있다.
-
생성자 넘기는 매개변수와 생성자 자체만으로는 객체의 특성을 제대로 설명하지 못한다.
이와 다르게 정적 팩터리 메서드는 이름을 어떻게 짓느냐에 따라 객체의 특성을 쉽게 묘사할 수 있다.
> 생성자 BigInteger(int , int , Random )과 정적 팩터리 메서드 BigInteger.probablePrime 중 어느 쪽이 '값이 소수인 BigInteger를 반환한다'라는 의미를 전달하는데 효과적인가
하나의 시그니처로는 하나의 생성자만 만들수 있는데 반해 이름을 가질 수 있는 정적 팩터리 메서드에는 이런 제약이 없다.
```java
class Client{
    String name;
    String id;

    public Client(String name) {
        this.name = name;
    }

    public Client(String id) {
        this.id = id;
    }
}
```
위 코드는 String 변수 하나를 매개변수로 받는 생성자가 두개 존재하게 되어 오류가 발생한다.

이런경우 아래와 같이 정적 팩터리 메서드를 활용할 수 있다.
```java
class Client{
    String name;
    String id;

    public static Client Client_name(String name) {
        Client client = new Client();
        client.name = name;
        return client;
    }

    public static Client Client_id(String id) {
        Client client = new Client();
        client.id = id;
        return client;
    }
}
```
장점2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
-
불변 클래스(아이템 17) 또는 같은 객체가 자주 요청되는 상황에서 불필요한 객체 생성을 피할 수 있다.
대표적인 예로 Boolean.valueOf(boolean)은 객체를 아예 생성하지 않는다.
```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean FALSE;
}
```

장점3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
-
이 능력은 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 제공한다. 이 유연성은 API를 만들때 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있게 한다.

자바 8에서 인터페이스에 정적 메서드를 선언할 수 있게 되며 리턴 타입에는 인터페이스만 노출 시키고 리턴은 실제 구현체 객체를 리턴하게 하여 클라이언트가 인터페이스만으로 다루게 된다.(아이템 64)

장점4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
-
장점3의 이유로 리턴 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환해도 되기 때문에 다음 릴리스에서는 코드의 수정없이 또 다른 클래스의 객체를 반환하도록 할 수 있다.
>EnumSet 클래스(아이템 36)는 원소의 수에 따라 적으면 RegularEnumSet 혹은 많으면 JumboEnumSet의 인스턴스를 반환한다. 만약 원소의 수가 적을때 RegularEnumSet을 사용할 이점이 사라진다면 다음 릴리스때 이를 삭제하고 JumboEnumSet만 반환하도록 해도 문제가 없다. 클라이언트는 RegularEnumSet과 JumboEnumSet을 알 수도, 알 필요도 없다.

장점5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
-
장점 3, 4와 맥락을 같이하는 장점이다. 이런 유연함은 서비스 제공자 프레임워크(service provider framework)를 만드는 근간이 된다. 이의 대표적인 예로 JDBC가 있다.

<details>
<summary></summary>
<div markdown="1">

서비스 제공자 프레임워크는 구현체의 동작을 정의하는 ```서비스 인터페이스```, 제공자가 구현체를 등록할 때 사용하는 ```제공자 등록 API```, 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 ```서비스 접근 API``` 이렇게 3개의 핵심 컴포넌트로 이뤄진다. 클라이언트는 서비스 접근 API를 사용할때 원하는 구현체의 조건을 명시할 수 있다.

</div>
</details>

단점1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들수 없다.
-
컬렉션 프레임워크의 유틸리티 구현 클래스들을 상속할수 없다는 이야기다. 하지만 이는 상속보다 컴포지션 사용하도록 유도하고 불변타입(아이템 17)으로 만들려면 관점에 따라서는 단점으로 받아들이지 않을 수도 있다.

단점2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
-
정적 팩터리 메서드는 생성자 처럼 API 설명에 명확히 드러나지 않아 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야한다.

다음은 현재 정적 팩터리 메서드에 흔히 사용하는 명명 방식이다.
>from： 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드\
>of： 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드\
>valueOf： from과 of의 더 자세한 버전\
>instance 혹은 getlnstance： (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지는 않는다.\
>create 혹은 newlnstance： instance 혹은 getlnstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.\
>getType： getlnstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”은 팩터리 메서드가 반환할 객체의 타입이다.\
>newType: newlnstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. “Type”은 팩터리 메서드가 반환할 객체의 타입이다.\
>type : getType과 newType의 간결한 버전


