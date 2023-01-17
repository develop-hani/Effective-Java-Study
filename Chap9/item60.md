정확한 답이 필요하다면 float와 double은 피하라
=
float와 double타입은 과학과 공학 계산용으로 설계되어 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 설계되었다.\
-> 정확한 결과가 필요할때 특히 금융관련 계산과는 맞지 않는다.(0.1 혹은 음의 거듭제곱수를 표현할 수 없다.)
```java
System.out.println(1.03 - 0.42); // 0.6100000000000001 출력
System.out.println(1.00 - 9 * 0.10); //0.09999999999999998 출력
```
> 위와 같이 기대한 값과 다른 값을 출력한다.
```java
//1달러를 가지고 10센트, 20센트, 30센트, ... 1달러 짜리의 사탕을 하나씩 살수 있을때까지 사는 프로그램
public static void main(String[] args) {
  double funds = 1.00;
  int itemsBought = 0;
  for (double price = 0.10; funds >= price; price += 0.10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러):" + funds);
```
>사탕 4개를 구입한 후 잔돈이 0달러 남기를 기대했지만 사탕 3개를 구입한 후 잔돈은 0.3999999999999999달러가 남는다. 

-> 금융 계산에는 BigDemical, int 혹은 long을 사용해야 한다.
## BigDemical, int, long
- BigDemical의 경우 기본타입보다 쓰기 훨씬 불편하고 훨씬 느리다.
- int, 혹은 long 타입을 쓸경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야한다.\
아래는 각각 위의 사탕 코드를 BigDemical, int를 사용해 수정한 코드이다.
```java
public static void main(String[] args) {
  final BigDemical TEN_CENTS = new BigDemical(".10");//계산시 부정확한 값이 사용되는걸 막기 위해 문자열을 받는 생성자를 사용
  
  int itemsBought = 0;
  BigDemical funds = new BigDemical("1.00");
  for (BigDemical price = TEN_CENTS; 
          funds.compareTo(price) >= 0;
          price = price.add(TEN_CENTS)) {
    funds = funds.subtract(price);
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러):" + funds);
```
```java
public static void main(String[] args) {
  // 소수점 관리를 직접하는 대신 달러대신 센트로 바꿔 수행
  int itemsBought = 0;
  int funds = 100;
  for (ㅑㅜㅅ price = 10; funds >= price; price += 10) {
    funds -= price;
    itemsBought++;
  }
  System.out.println(itemsBought + "개 구입");
  System.out.println("잔돈(달러):" + funds);
```
