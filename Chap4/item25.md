# 톱레벨 클래스는 한 파일에 하나만 담으라

소스 파일 하나에 톱레벨 클래스를 여러 개 선언하더라도 자바 컴파일러는 불평하지 않는다.<br>
**하지만 아무런 득이 없고 위험하다.**<br>
&rarr; 한 클래스를 여러 가지로 정의할 수 있고, 그 중 어느 소스 파일을 먼저 컴파일하냐에 따라 달라짐.

## 톱레벨 클래스 여러개 선언하는 예
```java
public class Main {
	public static void main(String[] args) {
    	System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```
- Main 클래스는 다른 톱레벨 클래스 2개를 참조한다.


```java
// 두 클래스가 한 파일(Utensil.java)에 정의되었다.
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}
```
```java
// 두 클래스가 한 파일(Dessert.java)에 정의되었다.
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}

```

- 컴파일러가 javac Main.java Dessert.java
&rarr; 컴파일 오류
- 컴파일러가 javac Main.java 나 javac Main.java Utensil.java
&rarr; pancake
- 컴파일러가 Dessert.java Main.java
&rarr; potpie

**컴파일러에 어느 소스 파일을 먼저 건네느냐에 따라 동작이 달라짐**

## 해결책

- 톱레벨 클래스를 서로 다른 소스 파일로 분리하자.
**&rarr; 소스 파일 하나에는 반드시 톱레벨 클래스를 하나만 담자.**
- 굳이 여러 톱레벨 클래스를 한 파일에 담고 싶다면 정적 멤버 클래스를 사용하자.
