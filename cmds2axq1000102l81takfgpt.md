---
title: "Dns란 무엇인가요?"
datePublished: Fri Aug 01 2025 00:05:02 GMT+0000 (Coordinated Universal Time)
cuid: cmds2axq1000102l81takfgpt
slug: dns
tags: dns, network

---

![Difference between DNS and DHCP - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/20250423151312675710/DNS-SERVER.webp align="center")

* *DNS(Domain Name System)*\*는 인터넷에서 **도메인 이름을 IP 주소로 변환해주는 시스템**입니다.
    

우리가 자주 접하는 `www.naver.com`, `www.google.com` 같은 주소가 모두 DNS를 통해 IP로 변환되어 접속됩니다.

---

## 왜 DNS가 필요할까?

* 컴퓨터는 통신할 때 `192.168.1.1` 같은 **IP 주소**를 사용
    
* 하지만 사람은 숫자 IP를 기억하기 어렵기 때문에 `www.example.com` 같은 **도메인 이름**을 사용
    
* **DNS는 이 도메인과 IP 주소를 매핑해주는 인터넷의 전화번호부 역할**을 합니다.
    

---

## DNS의 동작 과정

1️⃣ 브라우저에 `www.example.com` 입력

2️⃣ 로컬 DNS 캐시 확인 → 없으면 DNS 서버로 질의

3️⃣ DNS 서버가 해당 도메인의 IP 주소 반환

4️⃣ 브라우저가 IP를 통해 웹 서버와 통신

---

## DNS의 계층 구조

DNS는 전 세계적으로 분산된 계층 구조를 가지고 있으며, 각 단계가 협력하여 도메인을 IP로 변환합니다.

### 1\. Root DNS (루트 DNS 서버)

* **DNS 계층 구조의 최상위 레벨**
    
* 전 세계적으로 13개의 루트 서버 클러스터 존재 (`.`, A–M)
    
* 직접 IP를 반환하지 않고, **“어떤 TLD 서버에 물어봐야 하는지” 안내**
    

💡 예시:

`www.example.com` → 루트 DNS: “`.com` TLD 서버에 물어봐!”

---

### 2\. TLD DNS (Top Level Domain 서버)

* `.com`, `.net`, `.org`, 국가 코드 `.kr`, `.jp` 같은 TLD 관리
    
* 루트 서버로부터 질의를 받아 해당 도메인의 **권한 DNS 서버 정보**를 반환
    

💡 예시:

`.com` TLD → “`example.com`의 권한 DNS는 [ns1.exampledns.com](http://ns1.exampledns.com)”

---

### 3\. Authoritative DNS (권한 DNS 서버)

* **도메인의 최종 IP 주소를 보관하는 서버 (정답을 가진 서버)**
    
* 도메인 등록 시 지정하는 네임서버(NS)가 여기 해당
    
* 요청한 도메인의 **실제 IP 주소**를 반환
    

💡 예시:

Authoritative DNS → `www.example.com` → `93.184.216.34`

---

### 4\. Recursive DNS (재귀 DNS 서버)

* 사용자의 요청을 받아 루트 → TLD → Authoritative 순으로 **대신 탐색 (조회해주는 서버)**
    
* ISP(통신사)나 회사 네트워크에서 제공
    
* 결과를 일정 시간(Cache TTL) 동안 저장해 다음 요청을 빠르게 처리
    

💡 예시:

브라우저 → OS DNS → ISP Recursive DNS → 루트 → TLD → Authoritative → IP 반환

---

## DNS의 장점

* 사람이 읽기 쉬운 도메인으로 접근 가능
    
* IP가 바뀌어도 도메인만 유지하면 서비스 연결 가능
    
* 전 세계적으로 분산돼 있어 빠르고 안정적인 구조