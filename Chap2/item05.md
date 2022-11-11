## CH2 객체 생성과 파괴
+  _객체를 만들어야 할 때와 만들지 말아야 할 때를 구분하는 법_
+ _올바른 객체 생성 방법과 불필요한 생성을 피하는 방법_
+ _제때 파괴됨을 보장하고 파괴 전에 수행해야 할 정리 작업을 관리하는 요령_

### 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
대부분의 클래스는 여러 리소스에 의존한다. SpellChecker 그리고 Dictionary 를 예로 들고 있다. SpellChecker가 Dictionary를 사용하고, 이를 의존하는 리소스 또는 의존성이라고 부른다. 이때 Spellchecker를 다음과 같이 구현하는 경우가 있다.

### 부적절한 구현
#### static 유틸 클래스(아이템4)
```java
import java.util.List;

public class SpellChecker {

	private static final Lexicon dictionary = new KoreanDictionary();
    
    private SpellChecker() {
    	//Noninstantiable
    }
    
    public static boolean isValid(String word) {
    	throw new UnsupportedOperationException();
	}
    
    public static List<String> suggestions(String typo) {
    	throw new UnsupportedOperationException();
    }
    
    public static void main(String[] args) {
    	SpellChecker.isValid("hello");
    }
}

interface Lexion {}

class KoreanDictionary implements Lexicon {}
class EnglishDictionary implements Lexicon {}
```
KoreanDictionary 객체로 고정되어 있기 때문에 EnglishDictionary 와 같은 다른 객체로 변경하기가 쉽지 않다. 자연스럽게 코드는 유연하지 않을 뿐만 아니라 테스트할 때에도 객체 변경이 자유롭지 못해 테스트가 어렵다.

##### 싱글톤으로 구현하기(아이템3)
```java
import java.util.List;

public class SpellChecker {

	private final Lexicon dictionary = new KoreanDictionary();
    
    private SpellChecker() {
    	//Noninstantiable인스턴스화 방지용 생성자
    }
    
    public static final SpellChecker INSTANCE = new SpellChecker() {};
    
    public boolean isValid(String word) {
    	return true;
	}
    
    public List<String> suggestions(String typo) {
    	return new ArrayList<>();
    }
    
    public static void main(String[] args) {
    	boolean isChecked = SpellChecker.INSTANCE.isValid("hello");
        System.out.println("isChecked = " 
        + is Checked);
    }
}

interface Lexion {}

class KoreanDictionaty implements Lexicon {}

class EnglishDictionaty implements Lexicon {}

```
싱글톤 구현 역시 다른 객체를 대체하기가 쉽지 않은 구조이다. 

**어떤 클래스가 사용하는 리소스에 따라 행동을 달리 해야 하는 경우 static util 클래스와 싱글톤을 사용하는 것이 부적절하다. **

그런 요구 사항을 만족할 수 있는 간단한 패턴으로 생성자를 사용해 새 인스턴스를 생성할 때 사용할 리소스를 넘겨주는 방법이 있다.

### 적절한 구현
제시하는 해결책으로는 인스턴스를 생서할 때 생성자에게 필요한 자원을 넘겨주는 의존 객체 주입 방식을 소개한다. 

```java
import java.util.List;

public class SpellChecker {
	
	private final Lexicon dictionary;
    
    private SpellChecker(Lexicon dictionary) {
    	this.dictionary = Objects.requireNonNull(dictionary);
    }
    
    public boolean isValid(String word) {
    	return true;
	}
    
    public List<String> suggestions(String typo) {
    	return new ArrayList<>();
    }
    
    public static void main(String[] args) {
    	Lexicon lexicon = new Lexicon();

        SpellChecker spellCkecker = new SpellChecker(lexicon);
        boolean isChecked = spellChecker.isValid("hello");
        System.out.println("isChecked = " + isChecked);
    }
}

interface Lexion {}

class KoreanDictionary implements Lexicon {}

//test 코드로 사용할 경우
class TestDictionary implements Lexicon {}

```
자원을 사용하는 쪽(해당 코드 내에서는 main)에서 생성하여 클래스에 넘겨주는 방식(생성자로 주입)이다. 여기서 의존관계는 객체의 사용관계라고 생각하면 된다. 이런 구조라면, SpellChecker 클래스를 수정하지 않고 여러 Dictionary 객체를 변경할 수 있게 된다.

위와 같은 의존성 주입은 생성자, 스태틱 팩토리(아이템1) 그리고 빌더(아이템2)에도 적용할 수 있다.

이 패턴의 변종으로 리소스 팩토리를 생성자에 전달하는 방법도 있다. 이 방법은 자바 8에 들어온 ``Supplier<T>`` 인터페이스가 그런 팩토리로 쓰기에 완벽하다. ``Supplier<T>`` 를 인자로 받는 메서드는 보통 bounded wildcard type(아이템31)으로 입력을 제한해야 한다.

``Supplier<Lexicon>`` 은 메서드 레퍼런스(람다)형태로 표현이 가능하다. 


```java
public class SpellChecker {
	
    private final Lexicon dictionary;
    
    public SpellChecker(Supplier<Lexicon> dictionary) {
    	this.dictionary = Objects.requireNonNull(dictionary.get());
    }
    
    public boolean isValid(String word) {
    	return true;
    }
    
    public List<String> suggestions(String typo) {
    	return new ArrayList<>();
    }
    
    public static void main(String[] args) {
    	Supplier<Lexicon> dictionary = new Supplier<Lexicon>() {
        	@Override
            public Lexicon get() {
            	return new KoreanDictionary();
            }
        };//lambda 변경 가능
        SpeelChecker spellChecker = new SpellChecker(dictionary);
        boolean isChecked = sepellChecker.isValid("hello");
        System.out.println("isChecked : " + isChecked);
    }
}

interface Lexicon {}
class KoreanDictionary implements Lexicon {}
class EnglishDictionary implements Lexicon {}
```


의존성 주입이 유연함과 테스트의 용이함을 크게 향상시켜주지만, 의존성이 많은 큰 프로젝트의 경우 코드가 장황해질 수 있다. 그 점은 대거 스프링과 같은 프레임워크를 사용해 해결할 수 있다.
  
요약하면, 의존하는 리소스에 따라 행동을 달리하는 클래스를 만들 때에는 싱글톤이나 스태틱 유틸 클래스를 사용하지 말자. 그런 경우에는 리소스를 생성자나 팩토리로 전달하는 의존성 주입을 사용해 유연함, 재사용성, 테스트 용이성을 향상 시키자.

