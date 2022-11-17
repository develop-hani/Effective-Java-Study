# equals는 일반 규약을 지켜 재정의하라

equals를 재정의 하지 않으면 인스턴스는 오직 자기 자신만 같게 된다.

## equals를 재정의하지 않는 경우

- 각 인스턴스가 본질적으로 고유하다.
	- 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 해당 ex) Thread

- 인스턴스의 '논리적 동치성'을 검사할 일이 없다.
	- 설계자가 재정의하는 방식을 원하지 않거나 필요하지 않다고 판단하는 경우
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 맞는다.
	- Set, List, Map은 상위 클래스의 equals를 사용한다.
- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
	- 실수로 호출되는 것을 막고 싶다면 AssertionError()를 던져라.

## equals를 재정의하는 경우

논리적 동치성을 비교해야 하는 경우에 상위 클래스의 equals가 재정의되지 않았을 때 equals를 재정의한다. 

- 논리적 동치성을 확인하도록 재정의 목적
	- 인스턴스 값 비교가 가능
	- Map의 키와 Set의 원소로 사용할 수 있다.

인스턴스가 둘 이상 만들어지지 않는 클래스(인스턴스 통제 클래스)이거나 Enum이면 equals를 재정의하지 않아도 된다.
&rarr; Object.equals가 논리적 동치성까지 확인해준다.

## equals 메서드 일반 규약

- 반사성: null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true이다.
- 대칭성: null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true이다.
- 추이성: null이 아닌 모든 참조 값 x,y,z에 대해 x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true이다.
- 일관성: null이 아닌 모든 참조 값 x,y에 대해 x.equals(y)를 반복해서 호출하면 항상 true이거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해 x.equals(null)은 false이다.

수 많은 클래스들이 이 규약을 지킨다고 가정하고 동작하므로 equals를 재정의하지 않으면 큰일난다.


## 반사성

객체가 자기 자신과 같아야 한다는 뜻이다. 
이것을 어기면 컬렉션에 넣은 후 contains 메서드를 호출하면 인스턴스가 없다고 답한다.

## 대칭성
서로에 대한 동치 여부에 똑같이 답해야 한다.

- 대칭성을 어기는 예

```java

public final class CaseInsensitiveString {
	private final String s;
    
    public CaseInsensitiveString(String s) {
    	this.s = Objects.requireNonNull(s);
    }
    
    @Override
    public boolean equals(Object o) {	//잘못된 예
    	if (o instanceof CaseInsensitiveString)
        	return s.equalsIgnoreCase(
            (CaseInSensitiveString) o).s);
        if (o instanceof String)
        	return s.equalsIgnoreCase((String) o);
        return false;
    }
}

```
```java
	CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
    String s = "polish";
```

cis.equals(s) 는 true 지만 s.equals(cis)는 false를 반환한다.
&rarr; String.equals()는 CaseInsensitiveString의 존재를 모르므로 false를 반환한다.

 - 리스트를 사용하는 경우에는 JDK나 버전에 따라 결과가 다를 수 있다.
 **equals 규약을 어기면 그 객체를 사용하는 다른 객체들이 어떻게 반응할지 알 수 없다.**

- 해결 방법

```java
    @Override
    public boolean equals(Object o) {
    	return o instanceof CaseInsensitiveString &&
        	((CaseInsnsitiveString o).s.equalsIgnoreCase(s);
    }

```
Stirng과 CaseSensitiveString의 비교가 항상 false가 나오도록 막는다. (대칭성 유지)

## 추이성

<details>
  <summary>추이성이 어긋나는 예시 클래스</summary>
  
```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
  
public class ColorPoint extends Point {

    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;

        // o가 일반 Point면 색상을 무시하고 비교한다.
        if (!(o instanceof ColorPoint))
            return o.equals(this);

        return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```
  
</details>


```java
@Override
public boolean equals(Object o) {
	if(!(o instnaceof Point)
    	return false;
    
    // o가 일반 Point면 색상을 무시하고 비교한다.
    if(!o( instanceof ColorPoint)
    	return o.equals(this);
        
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

```java
ColorPoint p1 = new ColorPoint(1, 2, color.Red);
Point p2 = new Point(1,2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

위에 equals는 추이성이 어긋난다.
p1과 p2, p2와 p3는 색상을 고려하지 않지만, p1과 p는 색상을 고려하기 때문이다.
추이성이 어긋날 뿐만 아니라 다른 하위 클래스가 같은 방식의 equals()를 비교하면 StackOverFlowError를 일으킨다.
**구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.**


instanceof 대신 getClass()를 사용하면 같은 구현 클래스에만 true를 반환하지만, 
리스코프 치환 원칙에 어긋날 수 있기 때문에 사용할 수 없다.
&rarr; getClass()를 사용해 equals를 재정의하면 하위 클래스와 상위 클래스의 비교는 항상 false가 나오기 때문에


**&rarr;상속을 사용하기 보단 컴포지션을 사용하자.**

- 컴포지션 사용 예
```java
public class ColorPoint {
	private final Point point;
    private final Color color;
    
    public ColorPoint(int x, int y, Color color) {
    	point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }
	
    // 이 ColorPoint의 Point 뷰를 반환한다.
    public Point asPoint() {
    	return point;
    }
    
    @Override
    public boolean equals(Object o) {
    	if(!(o istanceof ColorPoint))
        	return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
        
    }
    
}

```

변수를 객체의 인스턴스로 사용하면 사용된 객체의 equals 끼리만 비교할 수 있다.


## 일관성

- 불변 객체는 equals가 참이면 항상 참이고, 거짓이면 항상 거짓이다.

- 클래스가 가변이든 불변이든 **equals의 판단에 신뢰할 수 없는 자원이 끼어들게 해서는 안된다.**
&rarr; 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다. 외부에서 계산된 값은 신뢰할 수 없음.


## null-아님

- 모든 객체는 null과 같지 않아야 한다.

- 명시적 null 검사보단 instanceof 를 사용해서 묵시적으로 null 검사를 하자.


## equals 구현 방법

1. == 연사자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

- float, double을 제외하고 기본 타입은 모두 == 연산자로 비교한다.
&rarr; float과 double은 Float.compare(float, float)와 Double.compare(double,double)를 사용한다.
- 배열 필드는 원소 각각을 앞서의 지침대로 비교한다. 모든 원소가 핵심 필드라면 Arrays.equals 메서드들 중 하나를 사용하자.
- 비교하기 복잡한 필드는 그 필드의 표준형을 저장해둔 후  표준형끼리 비교하자.

## equals 성능 개선

- 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하자.
- 파생필드를 굳이 비교할 필요 없지만, 파생 필드를 비교하는 쪽이 빠를 때는 파생 필드를 비교한다.

## equals 재정의 시 주의할 점

- equals를 구현한후, 대칭성, 추이성, 일관성을 확인해본다.
- AutoValue를 사용하면 equals를 알아서 작성해 준다.
- equals를 재정의할 땐 hashCode도 반드시 재정의하자.
- Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
&rarr; 재정의가 아니라 다중정의가 된다.
&rarr; @Override 어노테이션으로 실수를 예방할 수 있다.
