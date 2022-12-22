# 33. 타입 안전 이종 컨테이너를 고려하라

### 선요약

- 컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다.
- **컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다.**

---

## 기존 방식의 문제점: 매개변수화할 수 있는 타입 수의 제한

- 제네릭은 `Set<E>`, `Map<K,V>` 등의 컬렉션과 `ThreadLocal<T>`, `AtomicReference<T>` 등의 단일 컨테이너에 흔히 쓰인다.
- 아래의 경우 `intSet` 은 Integer 타입의 값만 포함할 수 있다.
    
    ```java
    Set<Integer> intSet = new HashSet<>();
    ```
    
- 하나의 컨테이너에서 매개변수화 할 수 있는 타입의 수가 제한된다.

---

## 타입 안전 이종 컨테이너 패턴(type safe heterogeneous container pattern)

- 컨테이너 대신 키를 매개변수화한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화 한 키를 함께 제공하여 **제네릭 타입 시스템이 값의 타입이 키와 같음을 보장해주는 패턴**을 말한다.
- 아래와 같이 키를 매개변수화하면, heterogeneous한 타입의 원소를 담을 수 있다.
    
    ```java
    public class Favorites {
        private Map<Class<?>, Object> favorites = new HashMap<>();
    
        public <T> void putFavorite(Class<T> type, T instance) {
            favorites.put(Objects.requireNonNull(type), instance);
        }
    
        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type)); 
        }
    
        public static void main(String[] args) {
            Favorites f = new Favorites();
            
            f.putFavorite(String.class, "Java");
            f.putFavorite(Integer.class, 0xcafebabe);
            f.putFavorite(Class.class, Favorites.class);
           
            String favoriteString = f.getFavorite(String.class);
            int favoriteInteger = f.getFavorite(Integer.class);
            Class<?> favoriteClass = f.getFavorite(Class.class);
            
            System.out.printf("%s %x %s%n", favoriteString,
                    favoriteInteger, favoriteClass.getName());
        }
    }
    ```
    

### 특징

- 와일드카드 타입이 중첩되어 있다. 즉, 맵이 아닌 키가 와일드카드 타입이다.
    
    `private Map<Class<?>, Object> favorites`
    
- 맵은 키와 값 사이의 타입 관계를 보증하지 않는다. 맵 값은 단순히 `Object` 이다.
- `getFavorite` 함수에서 형변환 과정이 필요하다.
    
    Object 타입의 객체(`favorite.get(type)`)를 T로 바꿔 반환한다.
    

### Favorite 클래스의 안정성

- `cast` 에 주목하자.
    
    `cast` 메서드는 `Class` 클래스가 제네릭이라는 이점을 완벽히 활용한다.
    
    ```java
    public class Class<T> {
    	T cast(Object object); // cast의 반환타입은 Class 객체의 타입 매개변수와 같다.
    }
    ```
    
    ⇒ T로 비검사 형변환하는 손실 없이도 Favorites를 타입 안전하게 만들 수 있다.
    
- 인수가 Class 객체가 알려주는 타입의 인스턴스인지 검사하고, 맞다면 그 인수를 반환하고, 아니라면 `ClassCastException`을 던진다.
    
    ```java
    f.putFavorite(Integer.class, "Integer 아니지롱");
    int favoriteInt = f.getFavorite(Integer.class); // ClassCastException 발생
    ```
    

---

## Favorite 클래스의 제약 2가지

### 제약 1: 클라이언트가 Class 객체를 제네릭이 아닌 Raw 타입으로 넘기면 타입안정성이 깨진다.

- 이 경우, 클라이언트 코드에서는 컴파일 시에 비겸사 경고가 뜬다.
- **value 인수로 주어진 타입이 key로 명시한 타입이 같은지 확인하여 타입 안정성을 확보할 수 있다.**
    
    ```java
    public <T> void putFavorite(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }
    ```
    

### 제약 2: 실체화 불가 타입에는 사용할 수 없다

- `String`이나 `String[]`은 사용할 수 있지만 `List<String>`은 컴파일되지 않는다.
    
    `List<String>`용 `Class`객체를 얻을 수 없기 때문이다.
    

---

## 한정적 타입 토큰: 메서드들이 허용하는 타입 제한

- `AnnotatedElement` 인터페이스에 선언된 메서드로 대상 요소에 달려 있는 애너테이션을 런타임에 읽어 오는 기능을 한다.
- 토큰으로 명시한 타입의 애너테이션이 대상 요소에 달려 있다면 그 애너테이션을 반환하고, 없다면 nul을 반환한다.
- Class<?> 타입의 객체가 있고 이를 한정적 타입 토큰을 받는 메서드에 넘기려면 Class 클래스의 asSubclass 메서드를 활용하자.
    
    ```java
    public class PrintAnnotation {
        static Annotation getAnnotation(AnnotatedElement element,
                                        String annotationTypeName) {
            Class<?> annotationType = null; // 비한정적 타입 토큰
            try {
                annotationType = Class.forName(annotationTypeName);
            } catch (Exception ex) {
                throw new IllegalArgumentException(ex);
            }
            return element.getAnnotation(
                    annotationType.asSubclass(Annotation.class));
        }
    }
    ```
    
    호출된 인스턴스 자신의 Class 객체를 인수가 명시한 클래스로 형변환하는 인스턴스 메서드이며 형변환에 성공시 인수로 받은 클래스 객체를 반환하고 실패시 ClassCastException을 던진다.
