# equlas를 재정의하려거든 hashCode도 재정의하라.



equals를 재정의한 클래스의 hashCode 메서드를 재정의 하지 않으면 동등한 객체를 다른 객체로 판단할 것이다.
&rarr;일반 규약을 어김

**따라서 equlas를 재정의한 클래스 모두에게 hashCode도 재정의해야 한다.**

## Object 명세에서 발췌한 규약
- equals 비교에 사용되는 정보가 변경되지 않았다면, 에플리케이션이 실행되는 동안 객체의 hashCode 값을 항상 같아야 한다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equlas(Object)가 두 객체를 다르게 판단했다면, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

**즉 논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

### 동등객체인지 비교하는 방법<br>
1.객체를 식별할 때 먼저 hashCode()를 실행하여 리턴된 해시코드 값이 같은지 먼저 본다.<br>
&rarr; 해쉬코드가 다르면 다른 객체로 판단, equals() 시도조차 X<br>
2.해시코드 값이 같으면 equals()를 비교하고 equals()가 맞아야 동등 객체로 판단한다.

&rarr; equals()가 성립하는 같은 객체를 HashMap에서 찾으려해도 hashCode() 재정의 하지 않으면 hash값이 다르게 나와(두번 째 규약 위반) null이 나온다.

## hashCode 구현 방법

### 간단한 방법 
```java
//최악의 구현 방법 - 사용 금지
@Override
public int hashCode() { return 42; }

```

&rarr; 사용 가능하지만 객체가 많아지면 느려져서 쓸 수 없음.

### 해시 코드 작성 요령


1. int 변수 result를 선언한 후 값 c로 초기화한다. **c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드**
2. 해당 객체의 나머지 핵심 필드(f)에 대해 다음 작업을 수행한다.
	1. 해당 필드의 해시코드 c를 계산한다.<br>

		a. 기본 타입 필드: Type.hashCode(f)를 수행 (Type은 해당 기본 타입의 박싱 클래스)
		
    		b. 참조 타입 필드
    		- 참조 필드 클래스의 필드들이 재귀적으로 hashCode 호출
    		- 복잡하면 표준형 만들어 표준형의 hashCode(호출)
    		- 필드 값이 null이면 0 사용
    			
    		c. 배열 필드
    		- 각각을 핵심 원소처럼 다룬다.
    		- 원소가 하나도 없다면 0 사용
 		- 모든 원소가 핵심 원소라면 Arrays.hashCode 사용 
 		
	2. 단계 2.1에서 계산한 해시코드 c로 result를 갱신
    	ex) result = 31 * result + c;
        
3. result를 반환한다.

- 핵심 필드: equals에서 비교에 사용된 필드
파생 필드와 equals 비교에 사용하지 않은 필드는 계산에서 제외한다.

- 31의 장점
1. 소수이다.
2. 최적화 하기 쉽다. (31 * i -> (i << 5) - i) 로 표현 가능)
3. 2의 곱이 아니다.

**좋은 해시 코드는 서로 다른 인스턴스에 다른 해시코드를 반환한다.**


```java
@Override
public int hashCode() {		// 위의 해시코드 작성법으로 만든 코드
	int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```



## Object 클래스의 해시코드를 계산해주는 정적 메서드 hash 사용

```java
@Override
public int hashCode() {
	return Objects.hash(lineNum, prefix, areaCode);
}
```

간단하지만 속도가 느리다.
&rarr; 입력 인수를 담기 위해 배열이 만들어지고 박싱과 언박싱을 거처야 하므로

## hashCode의 캐시와 지연 초기화

- 캐시
클래스가 불변이고 타입의 객체가 주로 해시의 키로 사용될 것 같으면 인스턴스가 만들어 질 때 해시코드를 계산해두어야 한다.

- 지연초기화 

해시의 키로 사양되지 않는 경우라면 hashCode가 처음 불릴 때 계산하는 지연 초기화 방법을 사용한다.

```java
private int hashCode	//자동으로 0으로 초기화, 불변일 경우 캐시처럼 사용 가능

@Override
public int hashCode() {
	int result = hashCode;
    if (result == 0) {	// 지연 초기화
    	result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
    }
    return result;

}
```

단 위에 코드에서 스레드 안정성을 추가해줘야 한다.
또 hashCode 필드의 초기값은 흔히 생성되는 객체의 해시코드와는 달라야 한다.






## 성능을 높인답시고 해시코드르 계산할 때 핵심 필드를 생략하지 마라

해시 품질이 나빠져 해시 테이블의 성능이 심각해짐
&rarr; 몇 개의 해시코드에 집중될 수 있다.
- 해시코드가 집중되는 경우: 자바 2 전의 String이 16개의 문자로만 해시코드를 계산해 String의 길이가 길어지면 성능이 나빠졌다.

## hashCode가 반환하는 값의 생성 규칙을 공표하지 말자.

**hashCode의 반환하는 값의 생성 규칙을 API 사용자에게 공표하지 말자.<br>
그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.**
