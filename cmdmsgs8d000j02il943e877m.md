---
title: "CQRS 패턴(Command Query Responsibility Segregation) - 읽기와 쓰기를 분리해야 할 때"
datePublished: Mon Jul 28 2025 07:30:48 GMT+0000 (Coordinated Universal Time)
cuid: cmdmsgs8d000j02il943e877m
slug: cqrs-command-query-responsibility-segregation
tags: backend, cqrs, springboot

---

### 📌  이런 상황, 겪어본 적 있나요?

* 서비스의 조회 요청이 폭주해서 읽기 성능을 높이려고 캐시를 도입했지만,
    
    쓰기 로직과 섞여 있어서 데이터 정합성 관리가 힘들어진다.
    
* 단순 CRUD 구조로 시작했는데, 도메인 로직이 복잡해지면서 서비스 코드가 점점 거대해진다.
    
* 보고용 조회 API (관리자 대시보드, 통계 API) 때문에 엔티티와 DTO가 난잡해지고, 쓰기 로직과 전혀 관계없는 SELECT 쿼리가 섞인다.
    

➡️ 이런 문제를 깔끔하게 정리할 수 있는 아키텍처 패턴이 **CQRS**입니다.

---

### 💡 CQRS란?

CQRS는 *Command Query Responsibility Segregation*의 약자입니다.

**명령(Command)** 과 **조회(Query)** 의 책임을 분리하는 아키텍처 패턴입니다.

* **Command:** 데이터를 변경하는 작업 (Create, Update, Delete)
    
* **Query:** 데이터를 읽는 작업 (Read)
    

➡️ **" 읽기와 쓰기를 같은 모델에서 처리하지 말고, 아예 분리하자."**

---

### 🔥 왜 굳이 분리할까?

* 읽기와 쓰기는 성격이 완전히 다릅니다.
    
    * 쓰기 → 트랜잭션과 데이터 무결성 보장
        
    * 읽기 → 빠른 응답과 다양한 검색 조건이 중요
        
* 한 모델에서 둘 다 처리하면 코드가 점점 복잡해지고, DB 모델링도 제약을 많이 받습니다.
    

➡️  CQRS는 이 둘을 나눠서 각각 최적화할 수 있게 해줍니다.

---

### CQRS 구조  

![CQRS 패턴의 상위 수준 보기](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-data-persistence/images/cqrs.png align="left")

* 쓰기(Command)는 비즈니스 규칙과 트랜잭션에 집중
    
* 읽기(Query)는 응답 속도와 조회 전용 모델에 집중
    

---

### Spring Boot에서의 코드 예시

### CQRS 미적용

```java

@Service
class UserService {
    public void updateUser(User user) { ... }   // Command
    public User getUser(Long id) { ... }        // Query
}

```

### CQRS 적용

```java

@Service
class UserCommandService {
    public void updateUser(UpdateUserCommand cmd) { ... }
}                                                     // 실제 DB 쓰기/이벤트 발행

@Service
class UserQueryService {
    public UserResponse getUser(Long id) { ... }
}                                                     // **캐시나 조회 전용 DB 사용** 

```

이렇게 Command와 Query를 아예 다른 서비스로 분리하면 코드 책임이 명확해짐.

---

### 장점

* **확장성:** 읽기/쓰기 각각 독립적으로 확장 가능 (조회만 스케일 아웃)
    
* **성능 최적화:** 읽기는 조회 성능이 높은 NoSQL , 쓰기는 트랜잭션 지원되는 RDB
    
* **유지보수성:** 코드가 깔끔해지고 책임이 분명해짐
    

---

### 단점

* **구조가 복잡해짐:** 작은 프로젝트에서는 오버엔지니어링
    
* **동기화 문제:** Command DB와 Query DB가 다르면 일관성 관리 필요
    
* **학습 곡선:** 팀 전체가 패턴을 이해하고 합의해야 함
    

---

### 언제 쓰면 좋은가?

* 조회 트래픽이 쓰기보다 훨씬 많을 때 (SNS 피드, 통계 서비스 등)
    
* 도메인 로직이 복잡해서 읽기/쓰기 모델을 별도로 설계해야 할 때
    
* Event Sourcing과 결합해서 감사 로그(Audit Log)를 남겨야 할 때
    

---

### 결론

> CQRS는 단순 CRUD 구조에서 벗어나, 읽기와 쓰기를 분리해 확장성과 성능을 높이는 아키텍처 패턴입니다.
> 
> 특히 대규모 트래픽, 복잡한 도메인 로직, DDD 환경에서 강력한 효과를 발휘합니다.
> 
> 다만 CQRS는 모델이 두 개로 나뉘고, 조회 전용 모델 설계가 필요하기 때문에 작은 프로젝트에서는 오히려 구현 비용이 더 커질 수 있습니다.
> 
> 단일 모델의 복잡성과 CQRS의 구현 복잡성을 비교해 신중하게 도입하는 것이 중요합니다.

출처 : [https://docs.aws.amazon.com/ko\_kr/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html](https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html)