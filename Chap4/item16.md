# 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 잘못 설계된 클래스

```java
class Point {
	public double x;
	public double y;
}
```

- 데이터 필드에 직접 접근할 수 있어 캡슐화의 이점을 누리지 제공하지 못한다.
    - API를 수정하지 않고 내부 표현을 바꿀 수 없다.
    - 불변식을 보장할 수 없다.
    - 외부에서 필등 접근할 때 부수 작업을 수행할 수도 없다.
- 필드를 private으로 바꾸고 public 접근자(getter)를 추가한다.

## 해결1: 접근자, 변경자(mutator) 메서드를 활용한 데이터 캡슐화

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

- getter/setter를 이용하여 내부 표현을 바꾸거나 부수 작업을 수행할 수 있다.
- 클라이언트의 메서드를 통해서만 필드에 접근이 가능하므로 불변식이 보장된다.

## 해결2: package-private 클래스, private 중첩 클래스

- package-private
    - 같은 패키지, 같은 클래스에서만 접근 가능하고, 외부 접근이 불가능하다.
- private 클래스 중첩
- 외부 클래스에서 직접 필드에 접근할 수 없다.

## public 클래스의 필드가 불변일 때, 직접 노출하는 것은?

```java
public final class Time {
    private static final int HOURS_PER_DAY    = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("Hour: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("Min: " + minute);
        this.hour = hour;
        this.minute = minute;
    }

    // 나머지 코드 생략
}
```

- 불변식은 보장할 수 있다.
- 그러나 다른 문제점들은 여전하다.
    - API를 변경하지 않고 표현 방식을 바꿀 수 없다.
    - 필드를 읽을 때 부수 작업을 수행할 수 없다.

## 요약

- public 클래스에서 가변 필드를 직접 노출하면 안된다.
- package-private 클래스와 private 클래스에서는 필드를 노출하는 것이 나을 때도 있다.
