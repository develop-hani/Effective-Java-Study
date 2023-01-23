문자열 연결은 느리니 주의하라
=
문자열 연결 연산자 +는 여러 문자열을 하나로 합쳐주는 편리한 수단이다. \
하지만 문자열 연결 연산자로 n개의 문자열을 잇는 시간은 n^2에 비례한다.(문자열 양쪽의 내용을 모두 복사해야하기 때문)\
->연결할 품목이 많은 경우 String 대신 StringBuilder의 append를 사용하자.
### StringBuilder의 appen 메서드 사용법
아래와 같이 StringBuilder의 append 메서드를 사용하여 합친 후 String 클래스에 대입하는 방식으로 쓰면된다.
```java
StringBuilder sample = new StringBuilder();
sample.append("abc");
sample.append("def");
sample.append("ghi");


Strint str = new String(sample);  //
```
