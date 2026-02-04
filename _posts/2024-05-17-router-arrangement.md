---
layout: post
title: "라우터 정리"
categories: [CS & Development]
tags: [라우터, 스위치, 네트워크, IP주소, 동적 경로, 정적 경로]
last_modified_at: 2024-05-17
---

라우터: data 전달하는 경로(route)를 정하는 장치  
이중에서 (가정용) 공유기는 단일 경로만 다루고 있다.  
스위치는 data의 통로를 선별하여 차단 또는 개방하는 문지기 역할을 수행 -> 불필요 data 필터링  

**라우터 기능**  
IP주소 사용하여 전달, 두 프로토콜이 같아야 함  
1. 경로 설정 -> 길을 검사하고 테이블링  
2. 패킷 전달  

스위치, 라우터 차이점...  
스위치: 중간에 있어서 경로를 스위칭 -> 데이터 링크 계층  
라우터: 경로를 찾아주는 -> 네트워크 계층  
라우터는 관리자의 설정으로 라우팅 테이블 생성, 통신 필요  

경로 설정에는 인위적으로 등록하는 정적 경로(Static Routing)과 알고리즘에 의해 판단하는 동적 경로(Dynamic Routing)가 있다.  
1. 정적 경로 설정 (직접 지정)  
2. 동적 경로 설정 (알고리즘)  

**스위칭할 때**  
동일한 그룹 내 정보 교환은 IGP 프로토콜로...  
다른 그룹 사이 정보 교환은 EGP 프로토콜로...  

![라우터 이미지](https://blog.kakaocdn.net/dna/NiNVG/btsHq2JO10W/AAAAAAAAAAAAAAAAAAAAAIzOe176T2c6WWklqaILY4aFLWOp8GOnf7XhOySLGJQq/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=X3UpRgUu2SHptoAuuMUrIf2vgNE%3D)  
target: TP-link M7350 Travel WiFi Router  
휴대용 라우터: 일반 라우터 + 무선 어댑터 역할  
4G 대상인 거 같다  
이더넷 케이블 연결 -> 라우터를 액세스 포인트로 활성화 -> 기기에서 암호로 로그인 -> 라우터 사용  
ISP와 포켓 라우터 연결 -> 무선 신호 브로드캐스팅  

v3의 제품설명서이지만... 일단 작동 방식은...  
IP address는 192.168.0.1이고, 서브넷 마스크는 255.255.255.0 -> 변경 가능  

**Connect to the Internet**  
1. 무선 네트워크 연결  
2. SSID 선택한 다음 연결 -> menu>Device Info에 SSID, PW, Login Name 담겨 있음  

![연결 화면](https://blog.kakaocdn.net/dna/WyCcf/btsHr1DxBO9/AAAAAAAAAAAAAAAAAAAAAGrxYWsEu678dnmAV8Jtj8rau9ftieOUzmTPzgfMhb9b/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vqSYUtw2%2FedHPswNOiw87ELKhuo%3D)  
3. 해당 화면 확인해서 기기에서 PW 입력하면 연결됨  
휴대폰이나 패드는 http://192.168.0.1 입력하고 아이디, 비밀번호 admin으로 로그인하면 좀 더 빠른 설정  

웹 기반 관리 페이지에는 다음과 같은 기능 존재  

![관리 페이지](https://blog.kakaocdn.net/dna/bMqJhe/btsHsIDlu3n/AAAAAAAAAAAAAAAAAAAAALztTqJggdKtmGp5r0xNOasYZaBZKUdgN5X1-Jskc_d8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MIVHWzlSFFZ7zw%2Fq8QWYqWYtr5Q%3D)  
간단하게 살펴 봤는데  
1. Wizard  
2. Status: 읽기 전용이라 딱히..  
3. SMS: 받은, 보낼 편지 등 메세지 담겨 있음  
4. advanced