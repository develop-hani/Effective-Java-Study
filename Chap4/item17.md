변경가능성을 최소화하라
=
### 불변 클래스 
- 그 인스턴스 내부 값을 수정할 수 없는 클래스
- 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적다
## 클래스를 불변으로 만들기 위한 5가지 규칙
- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다. ex) setter
- 클래스를 확장할 수 없도록 한다.
- 모든 필드를 final로 선언한다.
- 모든 필드를 private으로 선언한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

아래는 불변클래스의 예시이다.
```java
public final class Complex {//복소수
    private final double re;//실수부
    private final double im;//허수부
    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    //접근자 메서드 2개
    public double realPart() { return re; }//실수부 값 반환
    public double imaginaryPart() { return im; }//허수부 값 반환
    //사칙연산 메서드
    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }
    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }
    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }
    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }
    //Object의 메서드 재정의
    @Override public boolean equals (Object o) {
        if (o == this)
            return true;
        if (!(o instanceof Complex))
            return false;
        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0
                && Double.compare(c.im, im) == 0;
    }
    @Override public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }
    @Override public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```
위 클래스에서 사칙연산 메서드들은 자신을 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환한다. 
이처럼 피연산자에 한수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴을 **함수형 프로그래밍**이라 한다.
>함수형 프로그래밍을 사용하면 코드에서 불변이 되는 비율이 높아지는 장점이 있다.
위 클래스에서 메서드 이름으로 동사 대신 전치사를 사용하였는데 이는 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도이다.

## 불변 객체의 장점 및 특징
#### 불변 객체는 단순하여 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다.
- 모든 생성자가 불변식(class invariant)을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.
#### 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.
- 여러 스레드가 동시에 사용해도 절대 훼손되지 않는다.
#### 불변객체는 안심하고 공유할 수 있다.
- 한번 만들 인스턴스를 재활용하는데 안전하고 또 재활용해야 성능을 높일수 있다.
#### 불변객체는 자유롭게 공유할 수 있고, 불변 객체끼리는 내부 데이터를 공유할 수 있다.
- 불변객체 끼리는 내부 데이터를 공유하여도 각각이 불변이기 때문에 다른 객체에서 그 값을 변경할까 걱정할 필요가 없기 때문이다.
#### 객체를 만들 때 불변 객체들을 구성요소로 사용하면 불변식을 유지하기 수월하다.
#### 불변 객체는 그 자체로 실패 원자성을 제공한다
>실패 원자성 : 메서드에서 예외가 발생한 후에도 그 객체는 여전히 호출전과 똑같은 유효한 상태여야 한다는 성질

## 불변 객체의 단점
### 값이 다르면 반드시 독립된 객체로 만들어야 한다.
- 값의 가짓수가 많다면 이들을 모두 만들어야 하므로 큰 비용을 치러야 한다.

### 해결방법
클라이언트가 필요로 할 다단계 연산들을 예측하여 기본 기능으로 제공하는 방법 (다단계 연산속도를 높여주는 가변 동반 클래스를 제공)
- 각 단계마다 객체를 생성하지 않아도 된다.
- 클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 가변 동반 클래스를 package-private으로 제공하고 그렇지 못한다면 public으로 제공한다.

## 불변 클래스를 만드는 설계법들
### 클래스를 final로 선언
- 클래스를 상속하지 못하게 한다.
### 모든 생성자를 private 혹은 package-private으로 만들고 public 정적 팩터리를 제공하는 방법
- public 이나 protected 생성자가 없어 상속 이 불가능하다.
- 바깥에서 볼수 없는 package-private 구현 클래스를 원하는 만큼 활용할 수 있어 훨씬 유연하다.
다음은 앞선 Complex 코드를 이 방식으로 만든 코드이다.
```java
public class Complex {
    private final double re;
    private final double im;
    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }
    public static Complex valueOf(double re, double im){
        return new Complex(re, im);
    }
    //나머지 코드 생략
}
```


