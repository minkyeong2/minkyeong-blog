---
title: "Keep-Alive (HTTP Keep-Alive와 TCP Keep-Alive)"
datePublished: Tue Aug 12 2025 10:21:54 GMT+0000 (Coordinated Universal Time)
cuid: cme8e6l5t003h02jr0qlq0n77
slug: keep-alive-http-keep-alive-tcp-keep-alive
tags: network, http, networking, tcp

---

Keep-Alive는 연결(Connection)을 계속 유지하기 위한 메커니즘입니다.

네트워크에서 어떤 연결을 만들고, 요청/응답을 주고받은 후 바로 끊지 않고 일정 시간 동안 유지하면, 그 시간 안에 들어오는 추가 요청은 이미 열린 연결을 재사용할 수 있습니다.

이를 통해 매번 연결을 새로 맺는데 필요한 오버헤드(3-Way Handshake)를 줄일 수 있고, 불필요한 지연(latency)를 감소시킬 수 있습니다.

하지만 연결을 너무 오래 유지하면 사용하지 않는 연결이 쌓여 리소스가 낭비되고, 너무 짧게 유지하면 매 요청마다 연결을 새로 생성해야해서 비효율적입니다.

따라서 적절한 유지시간이 중요합니다.

---

### HTTP Keep-Alive 와 TCP Keep-Alive를 구분해야 하는 이유

Keep-Alive라고 해서 모두 같은 방식으로 동작하는 것은 아닙니다.

네트워크는 여러 계층으로 구성되어 있고, HTTP Keep-Alive는 애플리케이션 계층에서, TCP Keep-Alive는 전송 계층에서 동작합니다.

둘 다 연결을 유지하는 기능이지만, 동작 계층이 다르고 , 목적이 다르고, 관리 주체가 다릅니다.

## 요약

**HTTP Keep-Alive**

* 애플리케이션 계층(HTTP)
    
* 하나의 TCP 연결로 여러 HTTP 요청/응답 처리
    
* 동일 TCP 연결 재사용 (3-Way Handshake 재실행 방지)
    
* 웹 서버/ 애플리케이션이 관리
    
* 설정된 Keep-Alive 시간 동안 요청이 없으면 서버가 종료
    

**TCP Keep-Alive**

* 전송 계층(TCP)
    
* 유휴(Idle)TCP 연결이 살아있는지 확인이 목적
    
* 주기적으로 작은 패킷을 전송해서 연결 상태 확인
    
* OS가 관리
    
* 설정된 횟수/ 시간 동안 응답이 없으면 OS가 종료
    

---

1. **HTTP Keep-Alive**
    
    HTTP Keep-Alive는 동일한 TCP 연결을 재사용해 여러 HTTP 요청/응답을 처리하는 기능입니다.
    
    매 요청마다 TCP 연결을 새로 만들지 않으므로 지연시간이 줄고 서버 부하가 감소합니다.
    
    HTTP 응답 헤더 예시
    
    ```makefile
    HTTP/1.1 200 OK
    Connection: Keep-Alive 
    Keep-Alive: timeout=10, max=500
    ```
    
    timeout : 마지막 요청 이후 연결을 유지할 시간(초) →10초
    
    max : 연결 종료 전 처리할 수 있는 최대 요청 수 →500개
    
    Connection : Close가 응답에 포함되면, Keep-Alive를 사용하지 않고 요청 처리 후 연결을 바로 종료
    
    ---
    
2. **TCP Keep-Alive**
    
    TCP Keep-Alive는 유휴 상태의 TCP 연결이 유효한지 확인하는 기능입니다.
    
    운영체제가 주기적으로 작은 Keep-Alive 패킷을 보내 응답 여부를 확인하며 응답이 없으면 연결을 끊습니다.
    
    Linux 설정 예시
    
    ```makefile
    
    net.ipv4.tcp_keepalive_time = 40    # 최초 Keep-Alive 전송까지 대기 시간(초)
    net.ipv4.tcp_keepalive_intvl = 5    # 재전송 간격(초)
    net.ipv4.tcp_keepalive_probes = 3   # 응답이 없을 때 재전송 횟수
    ```
    
    위 설정이라면 40초 동안 응답이 없는 경우 5초 간격으로 3번의 Keep-Alive를 주고 받습니다.
    
    만약 정상적으로 Keep-Alive를 주고 받지 못했다면(55초) Connection을 종료합니다.
    

참고 자료:[https://sabarada.tistory.com/262](https://sabarada.tistory.com/262)