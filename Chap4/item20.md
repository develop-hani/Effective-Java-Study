## CH4 클래스와 인터페이스
> +  _추상화의 기본 단위인 클래스와 인터페이스는 자바 언어의 심장과도 같다_
+ _자바 언어의 클래스와 인터페이스 설계에 사용하는 강력한 요소를 적절히 활용하여 쓰기 편하고, 견고하며, 유연한 클래스와 인터페이스 만드는 방법을 안내한다_

### 아이템 20. 추상 클래스보다는 인터페이스를 우선하라.

### 다중 구현 메커니즘 : 추상 클래스 vs 인터페이스

자바 8부터 인터페이스에 디폴트 메서드(default method)를 제공하기 때문에, 두 방식 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다. 

가장 큰 차이점은 추상 클래스가 정의한 타입을 구현하는 클래스가 반드시 추상 클래스의 하위 클래스가 되어야 한다는 점이다. 자바가 단일 상속만 지원하므로, 추상클래스 방식은 새로운 타입을 정의하는 데 커다란 제약을 안게 되는 셈이다.

반면 인터페이스가 선언한 메서드를 모두 정의하고 그 일반 규약을 잘 지킨 클래스라면 다른 어떤 클래스를 상속했든 같은 타입으로 취급된다.

#### 1. 기존 클래스에 손쉽게 새로운 인터페이스를 구현해넣을 수 있다.

인터페이스가 요구하는 메서드를 추가하고, 클래스 선언에 `implements` 구문만 추가하면 끝이다.

`Comparable`, `Iterable`, `AutoCloseable` 인터페이스가 새롭게 추가 되었을 때 표준 라이브러리의 기존 클래스가 이 인터페이스를 구현한 채로 릴리즈 됐다. 

추상 클래스의 경우 두 클래스가 같은 추상 클래스를 확장하고자 한다면 그 추상 클래스가 계층구조상 두 클래스의 공통 조상이어야 한다. 새로 추가된 추상 클래스의 모든 자손이 이를 사옷갛게 되는 것이 적절하지 않음에도 강제될 수 있다.

#### 2. 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다. 

믹스인이란 클래스가 구현할 수 있는 타입으로, 믹스인을 구현한 클래스에 원래의 '주된 타입' 외에도 특정 선택적 행위를 제공한다고 선언하는 효과를 준다. 그 예로 `Comparable`은 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언하는 믹스인 인터페이스다.

추상 클래스는 기존 클래스에 덧씌울 수 없기 때문에 불가능하다. 자바는 단일 상속을 지원하기 때문에 한 클래스가 두 부모를 가질 수 없고 부모와 자식이라는 클래스 계층에서 믹스인이 들어갈 합리적인 위치가 없다.

``` java
public class Mixin implements Comparable {
	@Override
	public int compareTo(Object o) {
    	return 0;
    }
}
```
위처럼 Comparable을 구현한 클래스는 같은 클래스 인스턴스끼리는 순서를 정할 수 있는 것을 알 수 있다.

#### 3. 계층구조 없는 타입 프레임워크를 만들 수 있다.

물론 계층이 적절한 개념도 존재하지만, 현실에는 계층을 엄격히 구분하기 어려운 개념도 있다. 책에 등장하는 `Singer`, `Songwriter` 인터페이스를 생각해보자.

``` java
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
```

현실에는 싱어송라이터도 있으므로, 해당 개념을 구현하려면 다음처럼 새로운 계층을 만들면 된다. 심지어 `Singer`와 `Songwriter` 모두를 확장하고 새로운 메서드까지 추가한 제3의 인터페이스를 정의할 수도 있다.

``` java
public interface SingerSongwriter extends Singer, Songwriter {
	AudioClip strum();
	void actSensitive();
}
```
추상 클래스로 만들게 된다면
``` java
public abstract class Singer {
    abstract void sing(String s);
}

public abstract class SongWriter {
    abstract void compose(int chartPosition);
}

public abstract class SingerSongWriter {
    abstract void strum();
    abstract void actSensitive();
    abstract void Compose(int chartPosition);
    abstract void sing(String s);
}
```
이처럼 `Singer`와 `Songwriter` 클래스 모두 상속할 수 없어 `SingerSongWriter` 라는 또 다른 추상 클래스를 만들어 클래스 계층을 표현할 수 밖에 없다. 만약 이런 `Singer`와 `Songwriter` 같은 속성이 많아진다면 계층구조를 만들기 위한 많은 조합이 필요해지며, 이는 결국 고도비만 계층구조가 될 것이다. 이를 조합폭발이라 부른다.


### 래퍼 클래스 관용구(아이템 18)와 함께 사용하면 인터페이스는 기능을 향상시키는 안전하고 강력한 수단이 된다.

타입을 추상 클래스로 정의한 경우 그 타입에 기능을 추가하는 방법은 상속 뿐이다. 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 깨지기도 쉽다.

### 디폴트 메서드

인터페이스의 메서드 중 구현 방법이 명백한 것이 있다면, 그 구현을 디폴트 메서드로 제공할 수 있다. 디폴트 메서드를 제공할 때는 상속하려는 사람을 위한 설명을 `@implSpec` 자바독 태그를 붙여 문서화해야 한다.(아이템 19)

```java
코드21-1
default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ) {
    	if (filter.test(it.next())) {
        	it.remove();
            result = true;
        }
    }
    return result;
}
```
하지만 디폴트 메서드에도 단점은 존재한다. Object의 equals, hashcode 같은 메서드는 디폴트 메서드로 제공해서는 안 된다. 또한 public이 아닌 정적 멤버도 가질 수 없다. 또한 본인이 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다. 

### 추상 골격 구현 클래스

디폴트 메서드가 가지고 있는 단점을 극복하기 위해, 인터페이스와 추상 골격 구현 클래스를 함께 제공하는 방식으로 인터페이스와 추상 클래스의 장점을 모두 취할 수도 있다. 

인터페이스로는 타입을 정의하고, 골격 구현 클래스는 나머지 메서드를 구현한다. 이렇게 해두면 골격 구현 클래스를 확장하는 것 만으로 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다. 이를 템플릿 메서드 패턴 이라 부른다. 이런 추상 골격 구현 클래스를 보여주는 좋은 예로는 컬렉션 프레임워크의 `AbstractList`, `AbstractSet` 클래스이다. 이 두 추상 클래스는 각각 `List`, `Set` 인터페이스의 추상 골격 구현 클래스이다.


```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);

    //다이아몬드 연산자를 이렇게 사용하는 건 자바 9부터 가능
    //더 낮은 버전을 사용한다면 <Integer>로 수정
    return new AbstractList<>() {
        @Override public Integer get(int i) {
            return a[i];
        }

        @Override public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        @Override public int size() {
            return a.length;
        }
    };
}
```

#### 구현하기
골격 구현 작성은 다음과 같은 과정을 거쳐 만들면 된다.

1. 인터페이스를 살펴 다른 메서드들의 구현에 사용되는 기반 메서드를 선정한다.

2. 골격 구현에서 기반메서드가 추상메서드가 된다.

3. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드는 모두 디폴트 메서드로 제공한다. equals, hashcode와 같은 Object의 메서드는 디폴트 메서드로 제공하면 안 된다는 걸 유념하자.

4. 만약 인터페이스의 메서드 모두가 기반 메서드나 디폴트 메서드가 된다면, 굳이 골격 구현 클래스를 만들 필요가 없다. 기반 메서드나 디폴트 메서드로 만들지 못한 메서드가 남아있다면, 이 인터페이스를 구현하는 골격 구현 클래스를 하나 만들어 남은 메서드를 작성해 넣는다.

#### 예제의 상황은 자판기 인터페이스와 그것을 구현하는 음료수 자판기, 커피 자판기이다.

```java
public interface Vending {
    void start();
    void chooseProduct();
    void stop();
    void process();
}
```
```java
public class BaverageVending implements Vending {
    @Override
    public void start() {
        System.out.println("vending start");
    }

    @Override
    public void chooseProduct() {
        System.out.println("choose menu");
        System.out.println("coke");
        System.out.println("energy drink");
    }

    @Override
    public void stop() {
        System.out.println("stop vending");
    }

    @Override
    public void process() {
        start();
        chooseProduct();
        stop();
    }
}

public class CoffeeVending implements Vending {
    @Override
    public void start() {
        System.out.println("vending start");
    }

    @Override
    public void chooseProduct() {
        System.out.println("choose menu");
        System.out.println("americano");
        System.out.println("cafe latte");
    }

    @Override
    public void stop() {
        System.out.println("stop vending");
    }

    @Override
    public void process() {
        start();
        chooseProduct();
        stop();
    }
}
```
두 구현체 모두 `Vending` 인터페이스를 구현한다. 그런데 상품을 선택하는 `chooseProduct` 메서드를 제외하고 전부 다 같은 동작을 한다. 중복 코드를 제거하기 위해 인터페이스를 추상 클래스로 대체하지 않고 추상 골격 구현을 이용하면 다음과 같다.

```java
public abstract class AbstractVending implements Vending {
    @Override
    public void start() {
        System.out.println("vending start");
    }

    @Override
    public void stop() {
        System.out.println("stop vending");
    }

    @Override
    public void process() {
        start();
        chooseProduct();
        stop();
    }
}
```

```java
public class BaverageVending extends AbstractVending implements Vending {
    @Override
    public void chooseProduct() {
        System.out.println("choose menu");
        System.out.println("coke");
        System.out.println("energy drink");
    }
}

public class CoffeeVending extends AbstractVending implements Vending {
    @Override
    public void chooseProduct() {
        System.out.println("choose menu");
        System.out.println("americano");
        System.out.println("cafe latte");
    }
}
```
더 나아가 `Vending`을 구현하는 구현 클래스가 `VendingManuFacturer`라는 제조사 클래스를 상속받아야 해서 추상 골격 구현을 확장하지 못하는 경우는 어떻게 해야 하는가?

```java
public class VendingManufacturer {
    public void printManufacturerName() {
        System.out.println("Made By JavaBom");
    }
}

public class SnackVending extends VendingManufacturer implements Vending {
    InnerAbstractVending innerAbstractVending = new InnerAbstractVending();

    @Override
    public void start() {
        innerAbstractVending.start();
    }

    @Override
    public void chooseProduct() {
        innerAbstractVending.chooseProduct();
    }

    @Override
    public void stop() {
        innerAbstractVending.stop();
    }

    @Override
    public void process() {
        printManufacturerName();
        innerAbstractVending.process();
    }

    private class InnerAbstractVending extends AbstractVending {

        @Override
        public void chooseProduct() {
            System.out.println("choose product");
            System.out.println("chocolate");
            System.out.println("cracker");
        }
    }
}
```
이처럼 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 `private` 내부 클래스를 정의하고 각 메서드 호출을 내부 클래스의 인스턴스에 전달하여 골격 구현 클래스를 우회적으로 이용하는 방식을 사용한다. 이를 시뮬레이트한 다중상속(simulated multiple inheritance)라고 한다.

마지막으로 단순 구현(simple implementation)은 골격 구현의 작은 변종으로 골격 구현과 같이 상속을 위해 인터페이스를 구현한 것이지만, 추상 클래스가 아니라는 점에서 큰 차이점이 있다. 단순 구현은 추상 클래스와 다르게 그대로 써도 되거나 필요에 맞게 확장해도 된다. 단순 구현의 좋은 예로 `AbstractMap.SimpleEntry`가 있다.


#### 주의사항

추상 골격 구현 또한 상속을 사용하는 걸 가정하므로 item19에서 강조한 설계 및 문서화 지침에 따라야 한다. 
