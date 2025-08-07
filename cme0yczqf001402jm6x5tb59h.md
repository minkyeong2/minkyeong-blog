---
title: "Java에서 Object를  String 으로 변환할 때 주의할 점"
datePublished: Thu Aug 07 2025 05:24:35 GMT+0000 (Coordinated Universal Time)
cuid: cme0yczqf001402jm6x5tb59h
slug: java-object-string
tags: java, string

---

Java에서 Object 타입의 value를 String으로 변환할 때는 보통 두 가지 방식이 사용됩니다.

1. 명시적 타입 캐스팅
    
2. String.valueOf(value) 메서드
    

각 방식은 동작 방식과 안정성 면에서 중요한 차이가 있습니다.

---

## 1\. 명시적 타입 캐스팅

* 전제 조건: value가 반드시 String 타입이어야 함.
    
* 그렇지 않으면 ClassCastException이 발생함.
    

### 예시

```java

Object value = "hello";
String str = (String) value; // OK

Object value = 123;
String str = (String) value; // ClassCastException

```

<mark>💡</mark>**<mark>타입 캐스팅할 때 ClassCastException을 방지하는 방법은?</mark>**

**instanceof** 연산자를 사용하는 방법이 있습니다.

```java
Object value = "hello";

if (value instanceof String) {
    String str = (String) value;
    System.out.println(str); // OK
} else {
    System.out.println("value 는 String이 아닙니다.");
}
```

* instanceof 는 해당 객체가 특정 클래스의 인스턴스인지 확인.
    
* Java 16 부터는 패턴 매칭이 지원되어 더 간결하게 코드 작성 가능.
    

```java
if(value instanceof String str){
	System.out.println(str); // 자동 캐스팅
}
```

---

## 2\. String.valueOf(value)

```java
String str = String.valueOf(value);
```

* 안전한 방식.
    
* 내부적으로 아래와 같은 로직을 수행함.
    

```java

public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

### 특징

* value가 null이면 문자열 "null"을 반환.
    
* value가 다른 타입이면 toString()을 호출.
    
* 예외를 발생시키지 않음.
    

### 예시

```java

Object value = 123;
String str = String.valueOf(value); // "123"

Object value = null;
String str = String.valueOf(value); // "null"

```

<mark>💡String.valueOf(null)이 “null”을 반환하는 것은 문제가 될 수 있다.</mark>

이 방식은 실체 null 객체가 아닌 문자열 “null”을 반환하기 때문에 , 원래 의도와 다른 의미로 해석될 수 있습니다.

따라서 값이 null 일 경우에는 명시적으로 처리해주는 것이 좋습니다.

예를 들어, null 이면 빈 문자열을 사용하고 싶다면 다음과 같이 코드를 작성할 수 있습니다.