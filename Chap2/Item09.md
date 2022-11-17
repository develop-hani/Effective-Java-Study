try-finally보다는 try-with-resources를 사용하라
=
자바 라이브러리에는 close 메서드를 호출해 직접 닫아 줘야하는 자원이 많다.(InputStream, OutputStream, java.sql.Connection 등) 
자원 닫기는 잊기 쉬워 성능 문제로 이어지기도 한다. 안전망으로 finalizer를 활용하기도 하지만 믿을게 못된다.

## try-finally
전통적으로 예외가 발생하거나 메서드에서 반환되는 경우를 포함해서 제대로 닫힘을 보장하는 수단으로 쓰이던 방법이다.
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
하지만 이 방식은 자원이 늘어남에 따라 지저분해진다는 단점이 있다.
```java
static void copy(String src, String dst) throws IOException {//자원을 두개로 늘려본 예
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close();
        }
    } finally {
        in.close();
    }
}
```
위의 두 코드는 try-finally 문을 제대로 사용했음에도 미묘한 결점이 있다. 
예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데 후에 발생한 예외가 먼저 발생한 예외를 집어삼켜 스택추적내역에 먼저 발생한 예외가 기록되지 않아 디버깅을 어렵게 할 수 있다.
## try-with-resources
try-finally의 문제점을 try-with-resources가 해결하였다. 이 구조를 사용하려면 해당 자원이 AutoCloesable(void를 반환하는 close 메서드 하나만 정의 돼있는) 인터페이스를 구현해야 한다.

아래는 위의 두 코드들을 try-with-resources를 사용해 구현한 예시이다.
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
            new FileReader(path))) {
        return br.readLine();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```
try-finally 보다 짧고 읽기 수월할 뿐 아니라 문제를 진단하기도 훨씬 좋다. 
예외가 발생하면 프로그래머에게 보여줄 예외 하나만 보존되고 여러개의 다른 예외가 숨겨질 수도 있는데 이렇게 숨겨진 예외들도 그냥 버려지지 않고, 스택 추적 내역에 '숨겨졌다'는 꼬리표를 달고 출력된다.

try-with-resources에서도 catch 절을 쓸 수 있다. catch 절 덕분에 try 문을 더 중첩하지 않고도 다수의 예외를 처리할 수 있다.
