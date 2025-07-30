---
title: "테스트 더블 (Test Double)"
datePublished: Wed Jul 30 2025 09:48:38 GMT+0000 (Coordinated Universal Time)
cuid: cmdps9qx4000c02jlhyeg7idy
slug: test-double
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1753868910350/a24ca7c1-b7e6-48a1-9eae-9d84b85c6d5a.png
tags: testdouble

---

**테스트 더블(Test Double)** 은 *xUnit Test Patterns*의 저자 **Gerard Meszaros**가 만든 용어로,

**소프트웨어 테스트에서 실제 객체를 대체하는 가짜 객체를 통칭하는 개념**입니다.

실제 DB나 외부 API와 상호작용하는 코드는 테스트하기 무겁고 제어하기 어렵기 때문에,

테스트 더블을 사용해 **테스트를 독립적이고 빠르게** 만들 수 있습니다.

---

## 테스트 더블이 필요한 이유

* **실제 환경 의존성 제거:** DB, 네트워크, 외부 API 없이 테스트 가능
    
* **테스트 속도 향상:** 인메모리 객체 사용으로 I/O 부담 감소
    
* **예측 가능한 테스트:** 외부 서비스 상태와 무관하게 결과 고정
    
* **특정 상황 시뮬레이션:** 에러/타임아웃 등 비정상 케이스 재현 가능
    

---

## 테스트 더블의 종류 (Ge**r**ard Meszaros 분류)

### 1️⃣ Dummy

> 인스턴스는 필요하지만 동작은 전혀 필요 없는 객체

* 파라미터를 채우기 위해서만 사용되고, 실제 로직에서는 쓰이지 않음
    
* 호출되어도 정상 동작 보장하지 않음
    

```java

class NotificationService {
    public void sendNotification(User user) {
        // 테스트에서 user는 사용되지 않음
    }
}

@Test
void dummyTest() {
    NotificationService service = new NotificationService();
    User dummyUser = new User(); // 아무 동작 없는 더미 객체
    service.sendNotification(dummyUser);
}

```

**사용 포인트:** 단순히 객체만 필요할 때

---

### 2️⃣ Fake

> 실제와 유사하게 동작하지만 단순화된 구현

* 동작은 하지만 프로덕션 환경에서는 적합하지 않음
    
* 예: 인메모리 DB, 메모리 기반 Repository
    

```java

interface UserRepository {
    User save(User user);
    Optional<User> findById(Long id);
}

// 인메모리 Fake 구현
class FakeUserRepository implements UserRepository {
    private final Map<Long, User> store = new HashMap<>();
    public User save(User user) { store.put(user.getId(), user); return user; }
    public Optional<User> findById(Long id) { return Optional.ofNullable(store.get(id)); }
}

@Test
void fakeTest() {
    UserRepository repo = new FakeUserRepository();
    repo.save(new User(1L, "홍길동"));
    assertTrue(repo.findById(1L).isPresent());
}

```

**사용 포인트:** DB 없이 빠른 테스트를 하고 싶을 때

---

### 3️⃣ Stub

> 고정된 값을 반환하는 테스트용 구현

* 외부 API나 Repository 호출 결과를 항상 같은 값으로 응답하게 함
    
* 테스트를 **예측 가능하고 안정적**으로 만듦
    

```java

public class StubUserRepository implements UserRepository {
    // ...
    @Override
    public User findById(long id) {
        return new User(id, "Test User");
    }  // 동일한 id 값에 Test User라는 값을 가진 고정된 User 인스턴스 반환받음
}

```

**사용 포인트:** 특정 값으로 테스트를 고정하고 싶을 때

---

### 4️⃣ Mock

> 행위(Behavior) 검증을 위한 객체

* 메서드가 올바르게 호출됐는지(횟수/인자) 검증
    
* `verify()`로 호출 여부를 체크 → 결과값이 아니라 **행위 자체를 테스트**
    

```java

@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock PaymentGateway paymentGateway; // Mock 객체
    @InjectMocks OrderService orderService;

    @Test
    void mockTest() {
        orderService.placeOrder(500);
        // pay() 메서드가 정확히 한 번 호출됐는지 검증
        verify(paymentGateway, times(1)).pay(anyInt());
    }
}

```

**사용 포인트:** 특정 메서드가 기대한 대로 호출됐는지 확인할 때

---

### 5️⃣ Spy

> Stub + 호출 기록을 남기는 객체

* Stub처럼 값도 반환하면서, 메서드 호출 횟수나 인자를 기록
    
* 테스트 종료 후 검증에 활용
    

```java

class SpyEmailService implements EmailService {
    private int sendCount = 0;

    public void send(String to, String message) {
        sendCount++;
    }
    public int getSendCount() { return sendCount; }
}

@Test
void spyTest() {
    SpyEmailService spy = new SpyEmailService();
    UserService service = new UserService(spy);

    service.registerUser("test@test.com");

    assertEquals(1, spy.getSendCount()); // 호출 횟수 검증
}

```

**사용 포인트:** 호출 내역(횟수, 인자)을 검증하고 싶을 때

---

## 요약

| 종류 | 특징 | 사용 예시 |
| --- | --- | --- |
| Dummy | 동작 없음, 인스턴스만 필요 | 파라미터 채우기 |
| Fake | 단순화된 실제 구현 | 인메모리 DB |
| Stub | 고정된 응답 반환 | 외부 API 대체 |
| Mock | 행위 검증 | `verify()`로 호출 여부 체크 |
| Spy | Stub + 호출 기록 | 호출 횟수 검증 |

---

### 한 줄 요약

> 테스트 더블은 실제 객체 대신 사용하는 가짜 객체이며, 테스트의 독립성과 안정성을 높이기 위해 사용됩니다.
> 
> 목적에 따라 Dummy, Fake, Stub, Mock, Spy로 나뉘며, 각기 다른 테스트 시나리오에서 활용됩니다.

출처:[https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/](https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/)