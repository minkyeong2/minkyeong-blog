---
title: "낙관적 락(Optimistic Lock)과 비관적 락(Pessimistic Lock)"
datePublished: Fri Aug 22 2025 03:15:51 GMT+0000 (Coordinated Universal Time)
cuid: cmem9d79r000d02js1rb6hghu
slug: optimistic-lock-pessimistic-lock
tags: databases, concurrency, lock

---

## 락(Lock)의 필요성

현대 애플리케이션은 대부분 동시성(Concurrency) 문제에 직면합니다.

여러 사용자나 프로세스가 동시에 같은 데이터를 수정하거나, 은행 계좌 이체와 같은 중요한 트랙잭션을 동시에 실행하는 경우가 있습니다.

이때 동시 접근을 적절히 제어하지 않으면, 데이터 불일치, 중복 처리, 시스템 오류까지 발생할 수 있습니다.

이를 방지하기 위해 락(Lock) 개념이 도입되었습니다.

락은 여러 프로세스나 스레드가 동시에 동일한 자원에 접근할 때, 한 번에 한쪽만 접근하도록 제한하여 데이터 정합성을(Consistency) 보장합니다.

락에는 다양한 전략이 존재하는데, 그 중에서도 가장 비교되는 낙관적 락(Optimistic Lock)과 비관적 락(Pessimistic Lock) 을 알아보겠습니다.

---

## 낙관적 락 (Optimistic Lock)

낙관적 락은 충돌이 발생하지 않을 것이라고 가정하고, 트랜잭션이 데이터를 수정하고 할 때만 충돌을 처리합니다.

즉, 트랜잭션이 데이터에 락을 미리 걸지 않고, 데이터를 업데이트 하는 순간 다른 트랜잭션에 의해 변경되지 않았는지 확인합니다.

낙관적 락에서는 데이터를 수정할 때, 충돌이 발생하지 않았는지 검증하는 과정이 필요합니다.

일반적으로 버전(version)필드 또는 타임스탬프를 사용해 충돌을 감지합니다. 트랜잭션이 데이터를 수정할 때, 해당 데이터의 버전을 확인하고, 수정 시점에서 변경되지 않았을 때만 업데이트를 적용합니다.

### 장점

* 락을 걸지 않으므로 성능 우수
    
* 읽기 위주 시스템에 유리
    
* 데드락 문제 발생하지 않음 (트랜잭션간의 교착상태 발생 X)
    

### 단점

* 충돌이 자주 발생하는 경우, 트랜잭션 실패 가능성 높음 (재시도를 해야하므로 성능 떨어질 수 있음)
    
* 충돌을 감지한 후, 트랜잭션 재시도 및 오류 처리 등 추가 작업 필요
    

### 예시(JPA @Version )

```java
@Entity
public class Product {
    @Id
    private Long id;

    @Version
    private Integer version;

    private int stock;
}
```

예를 들어 id=1 인 상품을 조회해서 stock을 100→90으로 줄인다고 가정해보겠습니다.

```sql

-- 업데이트 시 WHERE 절에 version 조건이 포함됨
UPDATE product
SET stock = 90, version = version + 1
WHERE id = 1
  AND version = 1;
```

* version =1 인 상태에서 조회했다면, 업데이트 시에도 where절에 version =1 조건이 붙음
    
* 동시에 다른 트랜잭션이 업데이트해서 version이 2로 변경되었다면, 위 쿼리는 영향을 주는 행(row)가 없어짐
    
* JPA는 OptimisticLockException 발생시킴
    

---

## 비관적 락 (Pessimistic Lock)

비관적 락은 데이터의 충돌이 자주 발생할 것으로 가정하고, 트랙잭션이 데이터를 읽거나 수정할 때 즉시 락을 걸어 다른 트랜잭션이 해당 데이터에 접근하지 못하게 하는 방식입니다.

주로 데이터베이스의 ‘SELECT … FOR UPDATE’, 같은 기능으로 구현됩니다.

비관적 락은 두 유형으로 나뉩니다.

* 공유 락(Shared Lock)
    
    * 트랜잭션이 데이터를 읽을 때, 다른 트랜잭션이 데이터를 수정하지 못하도록 읽기 전용 락 설정
        
    * 동시에 여러 트랜잭션이 데이터를 읽을 수 있지만, 해당 데이터를 수정하려는 트랜잭션은 락이 해제될 때까지 대기
        
* 배타 락(Exclusive Lock)
    
    * 특정 트랜잭션이 데이터를 점유하는 동안, 다른 트랜잭션은 읽기/ 쓰기 불가능
        
    * 트랜잭션 완료 후에 락이 해제될 때까지 대기
        

### 장점

* 충돌이 자주 발생될 것으로 예상되는 환경에서 데이터 무결성 강력히 보장
    
* 데이터 수정 중, 다른 트랜잭션이 데이터에 접근할 수 없기 때문에 데이터가 다른 트랜잭션에 의해 변경되지 않도록 안정성 보장
    

### 단점

* 트랜잭션이 락을 해제할 때까지 다른 트랜잭션이 대기해야 하기 때문에 성능 저하 발생 가능
    
* 두 트랜잭션이 서로의 락을 기다리는 상황에서 데드락 발생 가능
    

### **예시 (JPA 비관적 락)**

```java

Product product = em.find(Product.class, productId, LockModeType.PESSIMISTIC_WRITE);
```

## 결론

낙관적 락과 비관적 락은 모두 데이터 정합성을 지키기 위한 방법이지만, 트랜잭션 충돌 가능성과 시스템 특성에 따라 적절히 선택해야 합니다. 읽기 위주의 시스템이라면 낙관적 락이, 쓰기 충돌이 빈번한 시스템이라면 비관적 락이 더 적합합니다.

참고 자료 : [https://hulrud.tistory.com/113](https://hulrud.tistory.com/113)