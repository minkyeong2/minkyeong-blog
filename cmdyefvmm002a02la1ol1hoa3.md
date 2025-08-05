---
title: "마이크로미터(Micrometer)와 Spring Boot Actuator로 메트릭 수집하기"
datePublished: Tue Aug 05 2025 10:31:25 GMT+0000 (Coordinated Universal Time)
cuid: cmdyefvmm002a02la1ol1hoa3
slug: micrometer-spring-boot-actuator
tags: metrics, springboot, micrometer

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754389827631/a5c6006c-0eba-413d-a21f-2b336fba0454.png align="center")

**마이크로미터(Micrometer)**는 JVM 기반 애플리케이션의 **메트릭(Metric)**을 수집하고, 다양한 모니터링 시스템으로 전송할 수 있도록 도와주는 애플리케이션 계측 라이브러리입니다.

주로 **Spring Boot**와 함께 사용되며, 애플리케이션의 성능, 상태, 리소스 사용량 등을 실시간으로 관측할 수 있게 해줍니다.

마이크로미터는 흔히 “**애플리케이션 메트릭 파사드**”라고 불립니다.

즉, 애플리케이션에서 수집한 메트릭 데이터를 Micrometer가 정의한 표준 방식으로 추상화하여, **특정 모니터링 툴에 종속되지 않도록** 해줍니다.

이를 통해 구현체만 바꾸면 다양한 외부 모니터링 시스템 (예: **Prometheus**, **Datadog**, **CloudWatch**)으로 연동할 수 있습니다.

Spring Boot Actuator는 Micrometer를 기본 내장하고 있으며, 설정만으로도 Prometheus 등 다양한 백엔드 시스템에 메트릭을 전송할 수 있습니다.

따라서 초기에는 Prometheus를 쓰다가, 추후 Datadog 등 다른 시스템으로 전환하더라도 **구현체만 변경하면 되기 때문에 유연한 모니터링 구성이 가능합니다.**

---

## 메트릭(Metric)이란?

**메트릭**은 시스템의 상태나 성능을 수치로 나타낸 값입니다.

이러한 값들은 **모니터링**, **알림**, **성능 최적화** 등에 사용됩니다.

### 메트릭의 특징

* 정량적 수치로 표현됨
    
* 그래프/차트로 시각화하기 용이 (ex. Grafana)
    
* 시간에 따라 변하므로 시계열 데이터로 저장됨
    

### 메트릭의 예시

| 메트릭 종류 | 설명 | 예시 |
| --- | --- | --- |
| 시스템 메트릭 | 서버 자원 상태 | CPU 사용률, 메모리 사용량, 디스크 공간 |
| 애플리케이션 메트릭 | 비즈니스 로직 처리 상태 | HTTP 요청 수, 응답 시간, 에러 비율 |
| JVM 메트릭 | 자바 애플리케이션 상태 | GC 횟수, 힙 메모리 사용량, 스레드 수 |
| 비즈니스 메트릭 | 비즈니스 도메인에 특화된 수치 | 가입자 수, 결제 성공 수, 주문량 등 |

### 메트릭 확인하기

[localhost:8080/actuator/metrics](http://localhost:8080/actuator/metrics)

metrics 엔드포인트를 사용하면 기본으로 제공되는 메트릭들을 확인할 수 있습니다.

![image.png](attachment:7c12dd52-fbd2-4564-aa4d-a8613c5d7249:image.png align="left")

---

## Spring Boot Actuator

**Spring Boot Actuator**는 스프링 부트 애플리케이션의 운영 및 모니터링에 필요한 다양한 기능을 제공하는 자동화된 관리 도구입니다.

### 기본 의존성 추가

```plaintext

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

### 기본 설정

```yaml

management:
  endpoints:
    web:
      exposure:
        include: "*"  # 또는 필요한 엔드포인트만 노출
```

### 주요 엔드포인트

* `/actuator/health` : 애플리케이션 상태 헬스 체크
    
* `/actuator/metrics` : 메트릭 수집 결과 확인
    
* `/actuator/loggers` : 런타임 중 로그 레벨 변경
    
* `/actuator/beans` : 등록된 Bean 목록 확인
    
* `/actuator/env` : 환경 변수 확인
    
* `/actuator/threaddump` : 스레드 덤프
    
* `/actuator/httptrace` : 최근 HTTP 요청 추적