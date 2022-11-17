## 아이템 12. 항상 toString을 재정의하라 Always override toString

### toString()

Object에서 정의한 toString()을 재정의하고 사용하지 않으면 `클래스 이름@16진수`로 표현한 해시코드 포맷으로 반환된다. toString의 규약에 따르면 간결하면서 사람이 읽기 쉬운 형태의 유익한 정보를 반환해야 한다.

equals나 hashCode처럼 시스템에 오작동을 일으키지는 않지만, item 12 규칙이 주는 개발상의 편의는 막대하므로 중요하다. 해당 객체를 사용하기에도, 디버깅하기에도 즐겁다.

``` java
System.out.println(phoneNumber);

--- 출력 결과 ---
// 재정의 하지 않은 경우
PhoneNumber@f32as225d

// 재정의한 경우 
707-867-5309
```


### 구현할 때 신경 쓸 점

#### 1. 그 객체가 가진 정보 모두를 반환하는 것이 좋다

- 그러나 객체가 거대하거나 객체의 상태가 문자열로 표현하기에 적합하지 않다면 요약 정보를 담는 것도 좋다

-`맨허튼 거주자 전화번호부(총 1487574개)` `Tread[main,5,main]`

- 이상적으로는 스스로 완벽히 설명하는 문자열이어야 한다(Thread는 그런 점에서 맞지 않음)

```java
///가진 정보를 모두 담지 않았을 때 문제가 되는 예시
Assertions failure: expected {abc, 123}, but was {abc, 123}
```

#### 2. 포멧을 문서화 할 지 정하자

- 전화번호나 행렬같은 VO는 문서화(주석 달기) 하기를 권한다. 포맷을 명시하면 그 객체는 표준적이고 명확하며 사람이 읽을 수 있게 된다. 그대로 입출력에 사용하거나 CSV파일 같은 곳에 저장할 수도 있다.

- 포맷을 명시하기로 했다면, 그 포맷에 맞는 문자열과 객체를 상호 전환할 수 있는 정적 팩토리나 생성자를 함께 제공해 주면 좋다.

```java
System.out.println(phoneNumber); // 123-123-1234
PhoneNubmer newNumber = PhoneNumber.of("123-123-1234");
```
- 포맷을 한 번 명시하면 평생 그 포맷에 얽매이게 된다. 향후 릴리스에서 포맷을 바꾸면 이를 사용하던 코드와 데이터들은 엉망이 될 것이다. 포맷을 명시하지 않는다면, 향후 릴리스에게 변경할 유연성을 갖게 된다.

- 포맷을 명시하든 아니든, 의도는 명확히 밝혀야 한다.

``` java
/**
 * Returns the string representation of this phone number.
 * The string consists of twelve characters whose format is
 * "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
 * prefix, and ZZZZ is the line number. Each of the capital
 * letters represents a single decimal digit.
 *
 * If any of the three parts of this phone number is too small
 * to fill up its field, the field is padded with leading zeros.
 * For example, if the value of the line number is 123, the last
 * four characters of the string representation will be "0123".
 */
@Override public String toString() {
	return String.format("%03d-%03d-%04d",areaCode, prefix, lineNum);
}

/**
 * Returns a brief description of this potion. The exact details
 * of the representation are unspecified and subject to change,
 * but the following may be regarded as typical:
 *
 * "[Potion #9: type=love, smell=turpentine, look=india ink]"
 */
@Override public String toString() { ... }
```

#### 3. toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자

- 그렇지 않으면 이 객체를 사용하는 프로그래머가 불필요한 parsing 작업을 해야한다. 접근자를 제공하지 않으면, toString 의 포맷이 사실상 준-표준 API처럼 동작하게 된다.
