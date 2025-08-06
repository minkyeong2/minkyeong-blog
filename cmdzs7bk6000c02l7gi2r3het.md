---
title: "Java에서 String이 불변(Immutable)인 이유"
datePublished: Wed Aug 06 2025 09:44:27 GMT+0000 (Coordinated Universal Time)
cuid: cmdzs7bk6000c02l7gi2r3het
slug: java-string-immutable
tags: java, string

---

Java에서 `String`은 **불변 객체(Immutable Object)**입니다.

즉, 한 번 생성된 `String` 객체는 내부 상태를 변경할 수 없습니다.

Java의 창시자 제임스 고슬링(James Gosling)은 다음과 같이 말했습니다.

> “가능하면 언제나 불변 객체를 사용하라.”

그 이유로는 **캐싱, 보안, 재사용성, 동기화, 성능 향상** 등을 언급했죠.

이번 포스팅에서는 왜 String이 불변으로 설계되었는지, 그리고 리터럴과 생성자의 차이, intern() 메서드까지 함께 살펴보겠습니다.

---

## 불변 객체(Immutable Object)란?

불변 객체는 **한 번 생성되면 내부 상태를 절대 변경할 수 없는 객체**입니다.

* 생성 이후 필드 값 변경 불가
    
* 다른 참조로 대체 불가
    

`String`은 대표적인 불변 객체입니다.

---

## String이 불변인 이유

### 1\. 문자열 상수 풀(String Pool) 활용

Java는 **문자열 리터럴을 상수 풀(String Pool)**에 저장하고 공유합니다.

```java
java
복사편집
String s1 = "Hello";
String s2 = "Hello";

System.out.println(s1 == s2); // true
```

* `"Hello"`는 상수 풀에 한 번만 저장됨
    
* `s1`, `s2`는 같은 객체를 참조 → **메모리 절약 + GC 부담 감소**
    

만약 **String이 가변(mutable)**이라면, 하나의 참조가 값을 바꾸면 다른 참조에도 영향을 주기 때문에 풀 공유가 불가능했을 것

---

### 2\. 보안(Security)

`String`은 민감한 데이터를 담습니다.

예: 사용자 이름, 비밀번호, DB 연결 정보 등

```java
java
복사편집
void criticalMethod(String userName) {
    if (!isAlphaNumeric(userName)) {
        throw new SecurityException();
    }

    connection.executeUpdate("UPDATE users SET name='" + userName + "'");
}
```

만약 `String`이 가변이라면, 외부 코드가 `userName`을 조작해서 SQL Injection 등의 보안 문제를 일으킬 수 있습니다.

→ **불변성 덕분에 이런 보안 문제를 방지**할 수 있습니다.

---

### 3\. 스레드 안전(Thread-Safety)

불변 객체는 상태가 바뀌지 않기 때문에,

**동기화 없이 여러 스레드에서 안전하게 공유**할 수 있습니다.

→ 복잡한 락(lock) 없이도 안전하게 사용할 수 있어 **멀티스레드 환경에 유리**합니다.

---

### 4\. 해시코드 캐싱

`String`은 `HashMap`, `HashSet` 등의 해시 기반 컬렉션의 키로 자주 사용됩니다.

* `String`은 `hashCode()`를 오버라이드하여 **한 번 계산한 값은 캐시**함
    
* 값이 바뀌지 않으니 **항상 동일한 해시값**을 보장
    

만약 `String`이 변경된다면 → 해시값이 바뀌고 → 컬렉션에서 찾을 수 없는 상황 발생

---

### 5\. 성능 최적화

* **상수 풀 재사용**
    
* **해시코드 캐싱**
    
* **동기화 생략 가능**
    

이 모든 요소들이 모여, Java 애플리케이션 전반의 성능 향상에 기여합니다.

---

## 리터럴 vs 생성자 방식

`String` 객체는 두 가지 방식으로 생성할 수 있습니다:

---

### 1\. 리터럴 방식

```java

String str1 = "hello";
String str2 = "hello";

System.out.println(str1 == str2); // true
```

* `"hello"`는 상수 풀에 저장됨
    
* str1과 str2는 **같은 객체를 참조**
    

✅ 메모리 절약

✅ 성능 향상

---

### 2\. 생성자 방식 (`new` 키워드)

```java

String str3 = new String("hello");
String str4 = new String("hello");

System.out.println(str3 == str4); // false
```

* `new`를 사용하면 항상 Heap에 새 객체 생성
    
* 내용은 같지만 객체는 다름
    

```java

System.out.println(str3.equals(str4)); // true
```

리터럴 방식보다 **메모리 낭비 가능성** 높음

→ 특별한 이유가 없다면 `new`는 지양하는 것이 좋습니다.

---

## intern() 메서드란?

```java

String s1 = new String("hello");
String s2 = s1.intern();
```

* `s1`은 Heap에 있는 새 객체
    
* `s2`는 `"hello"`라는 **상수 풀의 문자열을 참조**
    

즉, `intern()`은:

> 힙에 있는 문자열을 상수 풀의 동일한 객체로 참조 변경하는 메서드입니다.

---

### 예제

```java
java
복사편집
String a = "hello";                 // 상수 풀
String b = new String("hello");    // Heap
String c = b.intern();             // 상수 풀 참조

System.out.println(a == b); // false
System.out.println(a == c); // true

```

* `a`는 상수 풀
    
* `b`는 Heap 객체
    
* `c`는 `intern()`을 통해 상수 풀 참조 → `a == c` → `true`