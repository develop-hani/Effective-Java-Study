# [Item 06] 불필요한 객체 생성을 피하라
- 똑같은 기능의 객체를 재사용하는 것이 나을 때가 많다.
- 불변 객체는 언제든 재사용할 수 있다.
</br>

## 1. String

### ❌ 불필요한 객체 생성

```java
String s = new String("java");
```

- 문장이 실행될 때마다 같은 기능을 하는 객체 인스턴스를 새로 생성한다.

### ⭕ 하나의 인스턴스 재활용

```java
String s = "java";
```

- 같은 가상 머신 안에서 이와 **똑같은 문자열 리터럴을 사용하는 모든 코드가 같은 객체를 사용**함을 보장한다.
</br>

## 2. 정적 팩터리 메서드

### ❌ 생성자

- `Boolean(String)`
- 생성자는 호출할 때마다 새로운 객체를 생성한다.

### ⭕ 팩터리 메서드

- `Boolean.valueOf(String)`
- 정적 팩터리 메서드를 통해 불변 클래스에서 불필요한 객체 생성을 피할 수 있다.
</br>

## 3. 비싼 객체

- 생성 비용(메모리, 시간)이 큰 객체
- 비싼 객체가 반복해서 필요한 상황에서는 **캐싱하여 재사용**하길 권한다.

### ❌ `Pattern` 을 입력받은 정규 표현식

- Pattern은 입력받은 정규표현식에 해당하는 유한 상태머신을 만들어 인스턴스 생성비용이 높다.

```java
static boolean isRomanNumeral(String s){
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"+
            "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

- `s.matches` 는 내부적으로 `Pattern` 인스턴스를 생성한다.
    
    이는 한번 쓰이면 곧바로 Garbage Collection의 대상이 된다.
    

### ⭕ 캐싱하여 재활용

- 이런 경우, `Pattern` 인스턴스를 클래스 초기화 과정에서 직접 생성하여 캐신하고, 메소드를 호출할 때마다 재사용할 수 있다.
    
    ```java
    public class RomanNumerals {
    		// **클래스 초기화 과정에서 캐싱한다.**
        private static final Pattern ROMAN = Pattern.compile(
                "^(?=.)M*(C[MD]|D?C{0,3})"
                        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
        )
        
    		// **메서드가 호출될 때마다 재사용한다.**                
        static boolean isRomanNumeralFast(String s) {
            return ROMAN.matcher(s).matches();
        }
    }
    ```  
</br>

## 4. 오토 박싱

- 의도치 않은 오토박싱이 숨어들지 않도록 주의하자.
- 기본 타입과 박싱된 기본 타입을 섞어서 사용할 때 자동으로 상호 변환해주는 기술

### ❌ 오토 박싱

```java
public static long sum(){
    **Long** sum = 0L;
    for (long i = 0; i<= Integer.MAX_VALUE; i++)
        sum += i;

    return sum;
}
```

- Long으로 선언하여 불필요한 `Long` 인스턴스가 $2^{31}$개 생성된다.
    
    (`long` 타입인 i가 `Long` 타입인 sum 인스턴스에 더해질 때마다 생성)
    

### ⭕ 기본 타입 활용

- 위의 예시에서 `Long` 을 `long` 으로 바꿨을 때 훨씬 효율적이다.
</br>

# 정리

- 객체를 생성하는 것이 프로그램의 **명확성, 간결성, 기능**을 위한 것이라면 좋은 일이다.
- 그렇다고 단순히 객체 생성을 피하고자 자신만의 객체 풀(pool)을 만들지는 말자.

### 재사용 vs. 방어적 복사

- 재사용: 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라
- 방어적 복사: 새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라

⇒ 상황에 맞추어 판단하고 사용하자.
