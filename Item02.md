생성자에 매개변수가 많다면 빌더를 고려하라
=
정적 팩터리와 생성자에는 선택적 매개변수가 많을 경우 적절히 대응하기 어렵다는 제약이 있다.   
예를 들어 식품 포장의 영양정보를 표현하는 클래스를 생각해 보자.
영양정보는 몇개의 필수 항목과 20개가 넘는 선택항목으로 이뤄지는데 선택항목중 대다수의 값은 0이다.   
이런 클래스용 생성자 혹은 정적 팩터리는 어떤 모습일까?
### 점층적 생성자 패턴
적 생성자 패턴(telescoping constructor pattern)은 아래 코드와 같이 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 
필수 매개변수와 선택 매개변수 2개를 받는 생성자, ... 형태로 선택 매개변수를 전부 받는 생성자까지 늘려가는 방식이다.
<details>
<summary></summary>
<div markdown="1">
  
```
public class NutritionFacts {//분량상 매개변수가 4개까지 늘어난 코드이다.

    private final int servingSize; // （ml, 1회 제공량） 필수
    private final int servings; // （회, 총 n회 제공량） 필수
    private final int calories; // （1회 제공량당）
    private final int fat; // （g/1회 제공량）
    private final int sodium; // （mg/1회 제공량）
    private final int carbohydrate; // （g/1회 제공량）

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
이 클래스의 인스턴스를 만들려면 원하는 매개변수를 모두 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.
>NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27); //이런 생성자는 이 코드의 지방과 같이 설정하길 원치 않는 값 또한 지정해줘야 할 수 있다.
  
</div>
</details>

이러한 점층적 생성자 패턴은 사용할 수는 있지만, 매개변수가 많아지면 클라이언트가 코드를 작성하거나 읽기 기하급수적으로 어려워진다.

### 자바빈즈패턴
자바빈즈 패턴(JavaBeans pattern)은 매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식이다.
<details>
<summary></summary>
<div markdown="1">
  
```
public class NutritionFacts {
    // 매개변수들은 (기본값이 있다면) 기본값으로 초기화된다.
    private int servingSize = -1; // 필수; 기본값 없음
    private int servings = -1; // 필수; 기본값 없음
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // 세터 메서드들
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate (int val) { carbohydrate = val; }
}
```
  
```
NutritionFacts cocaCola = new NutritionFactsO;
cocaCola.setSe rvingSize(240);
cocaCola.setServings(8);
cocaCola.setCato ries(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```
  
</div>
</details>

점층적 생성자 패턴의 단점들을 자바빈즈 패턴으로 보완할 수 있지만 자바빈즈 패턴 또한 자신만의 단점을 가지고 있다.   
**자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지 일관성(consistency)이 무너진 상태에 놓이게 되고 그로인해 클래스를 불변(아이템 17)으로 만들 수 없다.**
이러한 단점을 완화하고자 생성이 끝난 객체를 수동으로 얼리고 얼리기 전에는 사용할 수 없도록 하기도 하지만 이 방식은 다루기 어려워 거의 쓰이지 않는다.

### 빌더 패턴
빌더 패턴(Builder pattern)은 점층적 생성자 패턴의 안정성과 자바빈즈 패턴의 가독성을 겸비하였다. 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 
생성자(혹은 정적 패터리)를 호출해 빌더 객체를 얻은 후 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다. 그리고 매개변수가 없는 build 메서드를 호출해 
우리에게 필요한 객체를 얻는다.
```
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) 
        { calories = val; return this; }

        public Builder fat(int val) 
        { fat = val; return this; }

        public Builder sodium(int val) 
        { sodium = val; return this; }

        public Builder carbohydrate(int val) 
        { carbohydrate = val; return this; }

        public NutritionFacts build() 
        { return new NutritionFacts(this); }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
빌더의 세터 메서드들은  빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 다음은 이 클래스를 사용하는 클라이언트 코드의 모습이다.
>NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)\
>　　　　　　　　　.calories(100).sodium (35).carbohydrate (27).build();

잘못된 매개변수를 초기에 조치하기 위해서는 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고, build 메서드가 호출하는 생성자에서 
여러 매개변수에 걸친 불변식(invariant)을 검사해야 한다. 공격에 대비해 불변식을 보장하려면 빌더로부터 매개변수를 복사한 후 해당 객체 필드들도 검사해야 한다.
검사해서 잘못된 점이 발견되면 어떤 매개변수가 잘못됐는지를 자세히 알려주는 메시지를 담아 IllegalArgumentException을 던지면 된다.

**빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.** 각 계층의 클래스에 관련 빌더를 멤버로 정의하자. 추상 클래스는 추상 빌더를, 구체 클래스는(concrete class)는 
구체 빌더를 갖게 한다.

<details>
<summary></summary>
<div markdown="1">
다음은 피자의 다양한 종류를 표현하는 계층구조의 루트에 놓인 추상 클래스이다

```
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }
        
        abstract Pizza build();
        
        // 하위 클래스는 이 메서드를 재정의(overriding)하여
        // "this"를 반환하도록 해야 한다.
        protected abstract T self();
    }
    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // 아이템 50 참조
    }

```

Pizza.Builder 클래스는 재귀적 타입 한정(아이템 30)을 이용하는 제너릭 타입이다. 여기에 추상 메서드인 self를 더해 하위 클래스에서 형변환 없이 메서드 연쇄를 할수 있도록 한다.

아래는 Pizza의 하위 클래스 두개이다.

```
public class NyPizza extends Pizza {//크기 매개변수를 필수
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;
    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);//requireNonNull -> 매개변수가 Null인지 아닌지 확인 후 세팅해줌
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }
        @Override protected Builder self() { return this; }
    }
    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```
public class Calzone extends Pizza {//소스를 안에 넣을지 선택 하는 매개변수 필수
    private final boolean sauceinside;
    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceinside = false; // 기본값
        public Builder saucelnside() {
            sauceinside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }
        @Override protected Builder self() { return this; }
    }
    private Calzone(Builder builder) {
        super(builder);
        sauceinside = builder.sauceinside;
    }
}
```
각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 하여 클라이언트가 형변환에 신경쓰지 않고 빌더를 사용할 수 있게 한다.
>하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌, 그 하위 타입을 반환하는 기능을 공변환 타이핑(covariant return typing)이라 한다.

다음은 두 피자의 클라이언트 코드의 모습이다
>NyPizza pizza = new NyPizza.Builder(SMALL)\
　　　　　　.addTopping(SAUSAGE).addTopping(ONION).build();   
> Calzone calzone = new Calzone.Builder()\
　　　　　　. addTopping (HAM) .saucelnside() .build();


</div>
</details>

빌더를 이용하면 생성자와는 달리 가변인수(varagas) 매개변수를 여러개 사용할 수 있다.(생성자나 정적 팩터리는 가변인자를 맨 마지막 매개변수에 한번 밖에 못받음) 각각을 메서드로 나눠 선언하거나 위 예의 addToping 처럼 메서드를 여러 번 호출하도록 하고 각 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다.

빌더 패턴의 단점으로는 객체를 만들기에 앞서 빌더를 만들어야 하기 때문에 성능에 민감한 경우 문제가 될 수 있다. 또한 점층적 생성자 패턴보다 코드가 장황해 질 수 있기 때문에 
매개변수가 4개 이상이거나 매개변수가 많아질 가능성이 있는경우 사용하는 것이 좋다.
