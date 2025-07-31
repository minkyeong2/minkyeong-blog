---
title: "리플렉션 (Reflection) 이란?"
datePublished: Thu Jul 31 2025 08:15:59 GMT+0000 (Coordinated Universal Time)
cuid: cmdr4efpc000502jpexpo20m3
slug: reflection
tags: java, reflection

---

**리플렉션(Reflection)** 은 **런타임에 클래스, 메서드, 필드 등의 메타데이터를 조회하거나 조작할 수 있는 기능**을 말합니다.

자바, C#, 파이썬 등 다양한 언어에서 제공하며, 특히 **자바 리플렉션 API**는 스프링, 하이버네이트 같은 프레임워크의 핵심 기술로 사용됩니다.

---

## 리플렉션의 특징

* **컴파일 타임이 아닌 런타임에 동작**
    
* 클래스의 구조(필드, 메서드, 생성자)에 접근 가능
    
* private 멤버도 강제로 접근 가능 (`setAccessible(true)`)
    

---

## 리플렉션 예시

### 1\. 클래스 정보 조회

```java

Class<?> clazz = Class.forName("com.example.User");

System.out.println(clazz.getName());  // 클래스 이름 출력

for (Method method : clazz.getDeclaredMethods()) {
    System.out.println(method.getName()); // 메서드 목록 출력
}

```

### 2 . 객체 생성 및 메서드 호출

```java
java
복사편집
Class<?> clazz = Class.forName("com.example.User");

// 생성자 호출로 인스턴스 생성
Constructor<?> constructor = clazz.getConstructor(String.class);
Object user = constructor.newInstance("홍길동");

// setName 메서드를 런타임에 호출
Method setName = clazz.getMethod("setName", String.class);
setName.invoke(user, "김철수");

```

### 3\. private 필드 접근

```java

Field field = clazz.getDeclaredField("name");
field.setAccessible(true);  // private 접근 허용
field.set(user, "이몽룡");

```

---

## `getMethod()` vs `getDeclaredMethod()`

리플렉션에서 메서드를 조회할 때 두 가지 메서드가 자주 혼동됩니다.

* `getMethod()`
    
    * `public` 메서드만 조회 가능
        
    * 부모 클래스에서 상속받은 public 메서드도 포함
        
    * private/protected/default 메서드는 조회 불가
        
* `getDeclaredMethod()`
    
    * 해당 클래스에 *선언된 모든 메서드* 조회 (접근 제어자 무관)
        
    * 상속받은 메서드는 포함되지 않음
        

### 예시 코드

```java

class Parent {
    public void hello() {}
}

class Child extends Parent {
    private void secret() {}
}

Class<?> clazz = Child.class;

// getMethod()
Method m1 = clazz.getMethod("hello");         // 부모의 public 메서드도 가져옴
// clazz.getMethod("secret");                 // private이라 에러

// getDeclaredMethod()
Method m3 = clazz.getDeclaredMethod("secret"); // Child에 선언된 private 메서드 조회 가능
// clazz.getDeclaredMethod("hello");          // Child에 직접 선언돼야만 가능

```

---

## 리플렉션의 장점

* **유연성:** 코드 작성 시점에 타입을 몰라도 런타임에 동적으로 처리 가능
    
* **프레임워크의 기반:** 스프링 DI, AOP, 하이버네이트 ORM 매핑 등 핵심 기술에 활용
    

## 리플렉션의 단점

* **성능 저하:** 런타임 탐색/호출은 일반 메서드 호출보다 느림
    
* **안전성 낮음:** 컴파일 시점 타입 체크 불가 → 런타임 에러 가능성 ↑
    
* **캡슐화 침해:** private 멤버 접근 가능 → 잘못 사용하면 유지보수성 저하
    

---

## 스프링에서 리플렉션의 활용

* `@Autowired` 의존성 주입
    
* `@Transactional` AOP 프록시 생성
    
* Bean 생성 시 클래스 메타데이터 스캔 및 동적 객체 생성
    

👉 스프링의 많은 기능이 리플렉션 기반으로 동작합니다. 런타임에 클래스 정보를 읽고 객체를 조작할 수 있는 이 기능 덕분에, 애너테이션 기반 DI와 AOP 같은 유연한 구조가 가능해집니다.

---

## 정리

* *리플렉션은 런타임에 클래스 구조를 조회하고 객체를 조작할 수 있는 기능*
    
* *강력한 유연성을 제공하지만 성능 저하와 캡슐화 침해를 주의해야 함*
    
* *스프링, 하이버네이트 등 다양한 프레임워크의 기반 기술*