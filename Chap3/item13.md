## 아이템 13. Clone 재정의는 주의해서 진행하라

### clone()

`clone()` 메서드의 경우, 복제하는 메서드인데 Object의 메서드가 아니다. `Coneable` 인터페이스의 추상 메서드이다. 그러므로 `Cloneable` 인터페이스가 구현된 경우만 가능한 메서드이다.

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 인터페이스이지만, 아쉽게도 의도한 목적을 제대로 이루지 못했다. `clone()` 메서드가 선언된 곳이 `Cloneable`이 아닌 Object, 그마저도 protected이기 때문이다. 

특이하게도 `Cloneable` 자체에는 메서드가 없고, `Cloneable`을 구현하고 Object의 `clone()` 메서드를 호출하면 `값을 복사한 객체`를, 그렇지 않으면 `CloneNotSupportedException`을 반환한다. _인터페이스가 상위 객체의 메서드의 행위를 변경하므로 좋은 방식은 아니다._

명세에는 없지만 실무에서는 `Cloneable`을 구현한 클래스는 clone 메서드를 public으로 제공한다.

``` java
import java.util.Calendar;

public class Example {
  public static void main(String[] args){
    //TODO AUTO_GENERATED method
    
    Calendar calendar = Calendar.getInstance();
    calendar.clean();
    calendar.set(2000, 9, 23);
    
    System.out.println("나의 생일은 : ");
    System.out.println(calendar.get(Calendar.YEAR) +"년"+ calendar.get(Calendar.MONTH) +"월"+ calendar.get(Calendar.DAY) + "일");
    
    Calendar calendar2 = (Calendar) calendar.clone();
    
    System.out.println("나의 생일은 : ");
    System.out.println(calendar2.get(Calendar.YEAR) +"년"+ calendar2.get(Calendar.MONTH) +"월"+ calendar2.get(Calendar.DAY) + "일");
    }
}


```

라이브러리에서 Calendar 클래스를 찾아 열어보면 java.lang.Cloneable이 나온다.
그렇다면 이 클래스로 객체를 만들면 clone() 메서드를 사용할 수 있다는 말이다.

clone메서드를 직접 사용하는 예제는 다음과 같다.

``` java
public class Student implements Cloneable{
  private Stirng Name;
  
  public Student(String name) {
    Name = name;
  }
  public String getName(){
    return Name;
  }
  
  @Override
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}
```
복제를 하기 위해서는 `Cloneable` 인터페이스를 구현하고 clone메서드를 재정의 해야한다.

재정의 내용은 상위 클래스의 clone을 return 하여 연결시켜주면 된다.
다만, `CloneNotSupportedException` 예외를 메소드 밖으로 던져주는 부분이 포함되어야 한다.

``` java
public class Example {
  public static void main(String[] args){
    Student stu1 = new Student("CY");
    Student stu2 = null;
    
    try{
      stu2 = (Student) stu1.clone();
    }catch(CloneNotSupportedException e) {
      e.printStackTrace();
    }
    
    System.out.println("stu1의 이름 : " + stu1.getName());
    System.out.println("stu1의 이름 : " + stu1.getName());
  }
}

```


### Cloneable의 주의점 및 문제점

1. A 클래스의 clone을 호출 할 때 상위클래스에서 정의한 clone이 호출 된다면, A 클래스가 아닌 상위클래스가 반환되게 된다. 이를 처리하기 위해 공변 반환 타이핑을 이용할 수 있다. 재정의한 메서드의 반환타입은 상위 클래스의 메서드가 반환하는 타입의 하위 타입일 수 있다.

2. `super.clone()`을 호출 하는 방식의 clone은 동일 참조의 필드를 전달해 오류가 생길 수 있다. 이를 막기 위해 재귀적으로 필드에 대한 `clone()`을 호출할 필요가 있다. 

3. `CloneNotSupportedException`을 체크드 예외로 던짐으로써 예외처리를 반드시 하도록 명시한다. `Cloneable`을 구현해 발생할 가능성이 없는 코드에서도!

4. 생성자와 동일한 역할을 하기 때문에 생성자의 역할을 모호하게 한다.

### Cloneable 사용을 지양하자

배열을 제외한 경우 `Cloneable` 사용을 지양해야 한다.`Cloneable`은 다양한 문제를 수반하므로, 새로운 인터페이스를 만들 때 `Cloneable`을 확장해서는 안 되고, 새로운 클래스도 이를 구현해서는 안 된다. 

final 클래스라면 구현해도 크게 위험하지 않지만, 성능 최적화 관점에서 문제가 없을 때 만 드물게 허용해야한다.

기본 원칙은 생성자와 팩터리를 이용하는 것이다.
