Serializable을 구현할지는 신중히 결정하라
=
Serializable 구현의 문제점
-
### 1. Serializable을 구현하면 릴리스 한 뒤에는 수정하기 어렵다.
클래스가 Serializable을 구현하면 직렬화된 바이트 스트림 인코딩(직렬화 형태)도 하나의 공개 API가 되어 클래스가 지원하는 한 그 직렬화 형태도 영원히 지원해야한다.
커스텀 직렬화 형태를 설계하지 않고 자바의 기본 방식을 사용하면 직렬화 형태는 최초 적용 당시 클래스의 내부 구현 방식에 영원히 묶여 버린다. 
기본 직렬화 형태에서는 클래스의 private과 package-private 인스턴스 필드들마저 API로 공개되는 꼴이된다.

뒤늦게 클래스 내부 구현을 손보면 원래의 직렬화 형태와 달라져 직렬화와 역직렬화의 버전이 달라져 실패하게 된다. 
따라서 직렬화 가능클래스를 만들고자 한다면 길게 보고 감당할 수 있을 만큼 고품질의 직렬화 형태도 주의해서 함께 설계해야 한다.

### 2. Serializable을 구현하면 버그와 보안 구멍이 생길 위험이 높아진다.
직렬화는 '객체는 생성자를 사용해 만드는게 기본'이라는 언어의 기본 메커니즘을 우회하는 객체 생성 기법이다. 
따라서 역직렬화는 일반 생성자의 문제가 그대로 적용되는 '숨은 생성자'인 샘인데, 
숨어있기 때문에 "생성자에서 구축한 불변식을 모두 보장해야 하고 생성 도중 공격자가 객체 내부를 들여다볼 수 없도록 해야한다."는 사실을 떠올리기 어렵다.
즉, 기본 역직렬화를 사용하면 불변식 깨짐과 허가되지 않은 접근에 쉽게 노출된다.

### 3. Serializable 해당 클래스의 신버전을 릴리스할 때 테스트할 것이 늘어난다.
직렬화 가능 클래스가 수정되면 신버전 인스턴스를 직렬화한 후 구버전으로 역직렬화할 수 있는지, 그 반대도 가능한지 검사해야 한다.
양방향 직렬화/역직렬화가 모두 성공하고, 원래의 객체를 충실히 복제해내는지를 반드시 확인해야 한다.
따라서 테스트해야 할 양이 직렬화 가능 클래스의 수와 릴리스 횟수에 비례해 증가한다.

## Serializable을 사용해야 하는경우
- 객체를 전송하거나 저장할 때 자바 직렬화를 이용하는 프레임워크용으로 만든 클래스
- Serializable을 반드시 구현해야 하는 다른 클래스의 컴포넌트로 쓰일 클래스
Serializable 구현에 따르는 비용과 이득을 잘 저울질 해야한다.\
역사적으로 '값' 클래스와 컬렉션 클래스들은 Serializable을 구현하고, 스레드 풀처럼 '동작'하는 객체를 표현하는 클래스들은 대부분 Serializable을 구현하지 않았다.

## 상속
### 상속용으로 설계된 클래스나 인터페이스 대부분은 Serializable을 확장해서는 안된다. 
이 규칙을 어길시 그런 클래스를 확장하거나 인터페이스를 구현하는것에 큰 부담이 생기게된다. 
Serializable을 구현한 클래스만 지원하는 프레임워크를 사용하는 등의 다른 방법이 없는 경우가 아니라면 이 규칙을 어기지 않아야 한다.

### 작성하는 클래스의 인스턴스 필드가 직렬화와 확장이 모두 가능하다면
- 인스턴스 필드 값 중 불변식을 보장해야 할 게 있다면 finalizer 공격을 예방하기 위해,
반드시 finalize 메서드를 자신이 final로 재정의하여 하위 클래스에서 finalize 메서드를 재정의하지 못하게 해야한다.
- 인스턴스 필드 중 기본값으로 초기화 뒤면 위배되는 불변식이 있다면 클래스에 다음의 readObjectNoData 메서드를 반드시 추가해야 한다. 
기존의 직렬화 가능 클래스에 직렬화 가능 상위 클래스를 추가하는 드문 경우를 위한 메서드다.
```java
private void readObjectNoData() throws InvalidObjectException {
  throw new InvalidObjectException("스트림 데이터가 필요합니다.");
}
```
### Serializable을 구현하지 않는다면
상속용 클래스인데 직렬화를 지원하지 않는다면 그 하위 클래스에서 직렬화를 지원하려 할 때 부담이 늘어난다. 
이런 클래스를 역직렬화하려면 그 상위 클래스는 매개변수가 없는 생성자를 제공해야 하는데 없다면 하위클래스에서는 어쩔수 없이 직렬화 프록시 패턴을 사용해야한다.

## 내부 클래스
내부 클래스에 대한 기본 직렬화 형태는 분명하지 않기 때문에 내부 클래스는 직렬화를 구현하지 말아야 한다. 
단, 정적 멤버 클래스는 Serializable을 구현해도 된다.
