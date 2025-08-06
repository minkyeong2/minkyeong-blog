---
title: "try-with-resources란?"
datePublished: Wed Aug 06 2025 05:39:19 GMT+0000 (Coordinated Universal Time)
cuid: cmdzjg2iw000502jvgo3mfr1j
slug: try-with-resources
tags: java, try-with-resources

---

Java에서는 파일, 소켓, DB커넥션 등 외부 자원을 사용할 경우 반드시 사용 후 자원을 닫아줘야 합니다.

그렇지 않으면 메모리 누수, 파일 잠금, 성능 저하 등의 문제가 발생할 수 있습니다.

이를 안전하고 간결하게 처리할 수 있도록 Java 7부터 도입된 문법이 바로 try-with-resources 입니다.

---

## 기존 방식의 한계

그동안은 아래처럼 try-catch-finally 구문을 사용해 닫는 방식이 일반적이었습니다.

```java
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("sample.txt"));
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (br != null) {
        try {
            br.close(); // 직접 닫아줘야 함
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

하지만 이 방식은 코드가 길어지고, close() 과정에서도 예외가 발생할 수 있어 복잡하고 실수하기 쉬운 구조입니다.

---

## try-with-resources 소개

Java 7 부터 도입된 try-with-resources는 자원을 try()블록 안에서 선언하면 자동으로 close()가 호출되는 구조입니다.

```java
try (BufferedReader br = new BufferedReader(new FileReader("sample.txt"))) {
    String line = br.readLine();
    System.out.println(line);
} catch (IOException e) {
    e.printStackTrace();
}
```

* finally 블록 없이도 안전하게 자원 해제가 가능함
    
* 코드가 훨씬 간결하고 가독성이 좋아짐
    
* 자원 해제 누락을 방지해 안정적인 코드 작성 가능
    

---

## 사용 조건

try-with-resources에서 사용할 수 있는 객체는 다음 인터페이스 중 하나를 구현해야 합니다.

* `java.lang.AutoCloseable` (Java 7 이상)
    
* [`java.io`](http://java.io)`.Closeable` (IO 관련 클래스 대부분)
    

→ close() 메서드를 가진 객체만 사용 가능합니다.

---

## 여러 자원 한 번에 다루기

```java
try (
    InputStream in = new FileInputStream("input.txt");
    OutputStream out = new FileOutputStream("output.txt")
    // ; 로 구분해 여러 자원을 동시에 선언 가능
) {
    byte[] buffer = new byte[1024];
    int bytesRead;
    while ((bytesRead = in.read(buffer)) != -1) {
        out.write(buffer, 0, bytesRead);
    }
}
```

‘;’ 로 구분해서 여러 리소스를 한 번에 선언할 수 있습니다. 리소스는 선언 순서의 역순으로 close() 됩니다.

---

## 사용자 정의 리소스 사용하기

직접 만든 클래스도 AutoCloseable 또는 Closeable을 구현하면 try-with-resource에서 사용할 수 있습니다.

```java

public class MyResource implements AutoCloseable {
    @Override
    public void close() {
        System.out.println("MyResource closed");
    }
}
```

```java

try (MyResource resource = new MyResource()) {
    // 작업 수행
}
```

---

## Java 9 이후 - 기존 변수도 사용 가능

Java 9 부터는 try()안에 기존에 선언된 final 또는 effectively final 변수도 사용할 수 있습니다.

```java

final Scanner scanner = new Scanner(new File("test.txt"));
PrintWriter writer = new PrintWriter(new File("out.txt"));
try (scanner; writer) {
    // 사용 가능
}
```

* **effectively final**: 명시적으로 `final`은 아니지만, 이후 값을 변경하지 않는 변수
    

참고 자료 : [https://www.baeldung.com/java-try-with-resources](https://www.baeldung.com/java-try-with-resources)