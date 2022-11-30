# 15. 클래스와 멤버의 접근 권한을 최소화하라

**정보 은닉, 캡슐화**

잘 설계된 컴포넌트는 모든 내부 구현을 완벽히 숨겨, 구현과 API를 깔끔히 분리한다.

오직 API를 통해서만 다른 컴포넌트와 소통하며 서로의 내부 동작 방식에는 전혀 개의치 않는다.


## 정보 은닉의 장점

- 시스템 개발 속도 향상
    
    여러 컴포넌트를 병렬로 개발할 수 있다.
    
- 시스템 관리 비용 절감
    
    각 컴포넌트의 역할을 파악하기 쉽고, 다른 컴포넌트로의 교체 부담이 적다.
    
- 성능 최적화에 도움
    
    다른 컴포넌트의 영향을 주지 않고 해당 컴포넌트만 최적화할 수 있다.
    
- 소프트웨어 재사용성 향상
- 큰 시스템을 제작하는 난이도 저하
    
    시스템 전체가 완성되지 않아도 개별 컴포넌트의 동작을 검증할 수 있다.
    


## 자바의 정보 은닉을 위한 장치: 접근 제어

**정보 은닉을 위해 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.**

### 톱레벨 클래스와 인터페이스에 부여할 수 있는 접근 수준

- **public**
    - 공개 API로 영원히 관리해야한다.
- **package-private**
    - 해당 패키지 내에서만 사용할 수 있다.
    - 클라이언트에 피해 없이 수정, 교체, 제거할 수 있다.
    - 한 클래스에서만 사용되는 package-private 톱레벨 클래스나 인터페이스는 이를 사용하는 클래스 안에 private static으로 중첩시킨다.
    - 외부 패키지에서 사용하지 않는 public 클래스는 package-private으로 접근 수준을 축소시킨다.

### 멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준

- **private**
    - 클래스의 공개 API를 세심히 설계한 후, 그 외의 모든 멤버는 private으로 만들자.
- **package-private**
    
     - 같은 패키지의 다른 클래스가 접근해야하는 멤버는 package-private으로 풀어주자.
    
- **protected**
    - 이 멤버를 선언한 하위 클래스에서 접근할 수 있다.
    - public 클래스에서 protected 멤버는 공개 API이므로 적을 수록 좋다.
- **public**
    - 모든 곳에서 접근할 수 있다.


## 컴포넌트 설계 방법

- 접근성을 좁히지 못하는 제약: 리스코프 치환 원칙
    - 상위 클래스의 메서드를 오버라이딩할 때, 접근 클래스를 상위 클래스보다 좁게 설정할 수 없다.
- 테스트를 위해 공개 API로 만들면 안 된다.
- public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.
    - 해당 필드와 관련된 모든 것을 불변식으로 보장할 수 없다.
    - 일반적으로 스레드는 안전하지 않다.
- public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공하면 안 된다.
    - 문제점 - 클라이언트가 내용을 수정할 수 있다.
        
        ```java
        public class ArrayTest {
        
            public static final String [] VALUES = {"A","B","C"};
        
            public static void main(String[] args) {
                ArrayTest.VALUES[1] = "D"; // 사용자가 내용 임의로 수정
            }
        }
        ```
        
    - 해결책1: public 배열을 private으로 만들고 public 불변 리스트를 추가한다.
        
        ```java
        public class SafeClass {
            private static final String[] VALUES = {"A", "B", "C"};
            public static final List<String> LISTVALUES = Collections.unmodifiableList(Arrays.asList(VALUES));
        
        }
        ```
        
    - 해결책2: 배열을 private으로 만들고 복사본을 반환하는 public 메서드를 추가한다(방어적 복사).
        
        ```java
        public class ArrayToClone {
            private static final String[] VALUES = {"A", "B", "C"};
            public static final String[] values(){
              return VALUES.clone();
            }
        }
        ```
        


## 자바9 - 모듈 시스템

- 모듈: 패키지들의 묶음
- 자신에 속하는 패키지 중 공개(export)할 것들을 (관례상 module-info.java에 )선언
    - protected, public 멤버라도 공개하지 않았다면 외부에서 접근 불가능
    - 모듈 내부에서는 상관없이 접근 가능
    - 모듈의 jar 파일 경로에 따라 의도하지 않은 접근이 일어날 수 있다.
