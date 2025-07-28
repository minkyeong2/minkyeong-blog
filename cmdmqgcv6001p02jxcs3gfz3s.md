---
title: "Spring Boot에서 Graceful Shutdown이 필요한 이유와 기본 설정이 꺼져 있는 이유"
datePublished: Mon Jul 28 2025 06:34:29 GMT+0000 (Coordinated Universal Time)
cuid: cmdmqgcv6001p02jxcs3gfz3s
slug: spring-boot-graceful-shutdown-1
tags: spring-boot-cj3rk5tin007imsk8uz3eg4kz, graceful-shutdown

---

## “Graceful Shutdown” 이란?

서비스를 운영하다 보면 서버를 재시작하거나 배포할 때 요청이 끊기는 문제가 생길 수 있습니다.

Graceful Shutdown은 서버가 종료될 때 **처리 중인 요청을 안전하게 마무리하고, 리소스를 정리한 후 종료하는 방식**을 말합니다.

일반적인 Immediate Shutdown(즉시 종료)과 달리, 현재 진행 중인 트랜잭션과 연결을 끝까지 처리해 **데이터 정합성을** 유지할 수 있습니다.

---

## 왜 필요한가?

1️⃣ **처리 중인 요청 손실 방지**

서버가 중간에 꺼지면 응답을 못 받은 클라이언트는 실패로 인식하지만, 서버 쪽에서는 이미 데이터가 반영될 수 있습니다. → **중복 처리, 데이터 불일치 발생**

2️⃣ **트랜잭션 안정성 확보**

DB 트랜잭션이 열린 상태에서 강제 종료되면 커밋/롤백이 정상적으로 이루어지지 않아 정합성 문제가 생깁니다.

3️⃣ **리소스 정리**

DB 커넥션, 메시지 큐 연결, 파일 핸들러 등을 정상적으로 닫아야 리소스 누수와 락(Lock)을 방지할 수 있습니다.

4️⃣ **클라우드 환경에서 필수**

Kubernetes, AWS Auto Scaling 등에서 인스턴스 교체 시 Graceful Shutdown이 없으면 Rolling Update 중 요청이 끊기고 서비스 장애가 발생할 수 있습니다.

---

## Spring Boot에서 기본 동작

* **기본값:** Immediate Shutdown (즉시 종료)
    
* 서버가 SIGTERM 신호를 받으면 새로운 요청을 차단하고, **현재 처리 중인 요청과 관계없이 바로 프로세스를 종료**합니다.|
    
    `SIGTERM` 수신 시
    
    1. 더 이상 요청을 받지 않음 (서버 소켓 닫힘)
        
    2. **현재 처리 중인 요청 무시하고 즉시 프로세스 종료**
        
    3. 커넥션 풀, 스레드 풀 강제 종료
        

➡️ 이 상태라면 긴 처리 요청 중간에 서버가 내려가면 **데이터 손실, 트랜잭션 중단, 요청 실패**가 발생할 수 있습니다.

## SIGTERM vs SIGKILL 차이

* **SIGKILL**: 프로세스를 강제로 종료, 핸들링 불가
    
* **SIGTERM**: 프로세스가 시그널을 받아 종료 절차를 안전하게 수행 가능
    
* Graceful Shutdown은 **SIGTERM**을 활용하고, **SIGKILL**은 Graceful Shutdown이 불가능하다.
    

---

## Spring Boot에서 Graceful Shutdown 활성화하기

Spring Boot 2.3 이상부터는 설정 한 줄로 Graceful Shutdown을 켤 수 있습니다.

```yaml

server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # 요청 완료 대기 최대 시간 (기본값 30s)
```

➡️ 새로운 요청은 차단하고, 현재 처리 중인 요청이 완료될 때까지 최대 30초 대기 후 종료.

---

## 그럼 왜 기본값이 "꺼져" 있을까?

라는 의문이 들 텐데요,

> "Graceful Shutdown이 이렇게 중요한데, 왜 Spring Boot는 기본으로 안 켜놨을까?"

### 1\. 빠른 종료 우선

초기 Spring Boot는 WAS(Tomcat)의 기본 정책을 따라 즉시 종료를 기본값으로 유지했습니다. 테스트나 간단한 앱에서는 빠르게 종료되는 것이 더 중요했기 때문입니다.

### 2\. 항상 안전한 것은 아님

Graceful Shutdown은 요청 완료를 기다리는 동안 서버가 종료되지 않고 남아 있을 수 있습니다. 요청이 오래 걸리는 서비스라면 **오히려 종료가 지연**될 수 있어 환경에 따라 부작용이 발생할 수 있습니다.

### 3\. 환경별 요구사항 다름

Kubernetes, ECS, 온프레미스 등 배포 환경마다 종료 시그널 처리 방식이 다르기 때문에, Spring은 **개발자가 의도적으로 설정을 켜는 것**을 권장합니다.

---

## 결론

* **Graceful Shutdown**은 안정적인 서비스 종료를 위해 꼭 필요한 기능
    
* Spring Boot는 2.3 이상에서 옵션으로 지원하며, **명시적으로 켜야 함**
    
* 클라우드 환경, 트랜잭션 무결성이 중요한 서비스에서는 필수 설정
    

---

## 한 줄 요약

> Graceful Shutdown은 요청 손실과 데이터 불일치를 막는 안전장치이며, Spring Boot에서는 기본 꺼져 있으니 server.shutdown=graceful 로 명시적으로 켜는 것이 좋다.