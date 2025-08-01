---
title: "자바 동시성 프로그래밍 :  경쟁상태, 원자성, 가시성"
datePublished: Fri Aug 01 2025 09:27:30 GMT+0000 (Coordinated Universal Time)
cuid: cmdsme95f000f02l85ara03ui
slug: 7j6q67cuioupmeylnoyessdtlitrozzqt7jrnpjrsi0goiag6rk97jb7iob7yoclcdsm5dsnpdshlesioqwgoylnoyesq
tags: java, race-condition

---

멀티스레드 환경에서는 성능을 높일 수 있지만, 동시에 Race Condition 같은 동시성 문제가 발생할 수 있습니다.

특히 원자성(Atomicity) 과 가시성(Visibility)은 이 문제를 이해하는데 중요한 핵심 키워드 입니다.

---

## 동시성 문제란?

동시성 문제는 여러 스레드가 동시에 같은 자원에 접근하면서 실행 순서에 따라 결과가 달라지거나 데이터가 손상되는 상황을 말합니다.

## 대표적인 동시성 문제 (Race Condition)

* 두 개 이상의 스레드가 동시에 공유 자원에 접근
    
* 실행 순서에 따라 결과가 달라지거나 값이 꼬이는 현상이 발생
    
    ex) 두 스레드가 동시에 i++를 실행할 때, 최종 값이 2가 아닌 1이 될 수 있음
    

---

## 원자성(Atomicity)

: 작업이 더 이상 쪼갤 수 없는 단일 연산 처럼 수행되는 성질

## 왜 필요한가?

![](https://github.com/user-attachments/assets/d7c8463f-9a94-4909-ab30-86e9572f8bd2 align="left")

## 단일 스레드 코드 예시

```java

int i = 0;

// i++ 연산을 분해하면 실제로는 이렇게 동작
int temp = i;     //  Read: i 값을 읽음
temp = temp + 1;  //  Modify: 읽은 값에 1을 더함
i = temp;         //  Write: 결과를 i에 다시 씀

```

i++는 하나의 코드이지만 실제 CPU는 3단계로 수행합니다.

1. i 값을 읽음 (Read)
    
2. 1을 더함 (Modify)
    
3. 결과를 메모리에 씀 (Write)
    

→ 두 스레드가 동시에 연산을 수행하면, 중간 상태에서 개입해 잘못된 값이 저장될 수 있다 (Race Condition)

## 멀티스레드 환경에서 Race Condition 예시

```java

public class RaceConditionExample {
    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int j = 0; j < 1000; j++) {
                int temp = count;     // Read
                temp = temp + 1;      // Modify
                count = temp;         // Write
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("최종 count = " + count);
    }
}

```

* 두 스레드가 동시에 실행하면 `count` 값이 **2000이 아닌 더 작은 값**으로 출력될 수 있습니다.
    
* 이유: 한 스레드가 `Read`한 후 `Modify` 하기 전에 다른 스레드가 `Write`를 해버리면, 이전 값이 덮어씌워지기 때문.
    

## 원자성 보장한 코드

```java

public class AtomicExample {
    private static int count = 0;

    public static synchronized void increment() {
        count++; // synchronized 덕분에 Read-Modify-Write가 하나의 원자적 연산이 됨
    }

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            for (int j = 0; j < 1000; j++) {
                increment();
            }
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("최종 count = " + count); // 항상 2000
    }
}

```

---

## 가시성(Visibility)

: 한 스레드에서 변경한 값이 다른 스레드에 즉시 보이는 성질

## 비가시성은 왜 문제가 될까?

![[Java] Java의 동시성 이슈 - 동시성 프로그래밍에서 발생할 수 있는 문제점](https://blog.kakaocdn.net/dna/GHaI4/btrpgyELIEw/AAAAAAAAAAAAAAAAAAAAAFyl-M1dZMyGcJr5hp3slsMkcgv8KCBKUXLPFcPzzXKo/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=JmA%2F5X72TeRpnaQaJY2D1svi7RM%3D align="left")

* CPU는 속도가 매우 빠르지만, RAM(메인 메모리) 접근은 상대적으로 느림
    
* CPU는 성능을 위해 CPU캐시 (고속 메모리) → 메모리 구조를 사용
    
* 한 스레드가 값을 바꿔도 , 메인 메모리에 반영되기 전까지 다른 스레드는 변경 전의 값을 읽을 수 있음
    

ex) 스레드 A가 flag == true 로 바꿨지만, Thread B가 여전히 false 로 보고 무한 루프에 빠짐

## 비가시성 예시 코드 (volatile 없이)

```java

public class VisibilityIssue {
    private static boolean running = true; // 가시성 보장 X

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) {
                // running 값이 false로 바뀌어도 CPU 캐시에서 옛 값(true)을 계속 읽을 수 있음
            }
            System.out.println("작업 종료");
        });

        worker.start();

        Thread.sleep(1000);
        running = false; // 메인 스레드에서 값을 false로 변경
        System.out.println("running 값을 false로 변경");
    }
}

```

### 결과

* 메인 스레드가 `running = false`로 바꿔도, `worker` 스레드는 계속 무한 루프에 빠져 종료되지 않을 수 있습니다.
    
* 이유: `worker` 스레드가 CPU 캐시의 오래된 값을 보고 있기 때문.
    

## 가시성 보장 방법

* volatile 키워드 : 항상 메인 메모리에서 읽고 쓰도록 강제
    
* synchronized / Lock → 블록 진입 시 캐시와 메모리 동기화
    
* 단 volatile은 원자성은 보장하지 않음
    

## 가시성 문제 해결 (volatile 사용)

```java

public class VisibilitySolution {
    private static volatile boolean running = true; // volatile로 가시성 보장

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) {
                // 메인 메모리에서 최신 값을 읽기 때문에 running=false 변경 즉시 감지 가능
            }
            System.out.println("작업 종료");
        });

        worker.start();

        Thread.sleep(1000);
        running = false;
        System.out.println("running 값을 false로 변경");
    }
}

```

### 결과

* `volatile` 덕분에 CPU 캐시가 아닌 **메인 메모리에서 값을 읽고 쓰기 때문에**, `running=false`가 즉시 다른 스레드에 반영되고 루프가 종료됩니다.
    

참고자료 : [https://www.maeil-mail.kr/question/203](https://www.maeil-mail.kr/question/203)  
[https://steady-coding.tistory.com/554](https://steady-coding.tistory.com/554)