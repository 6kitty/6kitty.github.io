---
layout: post
title: "WEB & HTTP/HTTPS"
categories: [Self-study]
tags: [HTTP, HTTPS, Web, Protocol]
last_modified_at: 2024-12-26
---

웹: HTTP라는 프로토콜을 이용한 서비스  
제공 주체 -> 서버  
받는 주체 -> 클라이언트  
HTTP를 이용하여 서버와 클라이언트가 통신  

웹 리소스  
클라이언트의 요청 받는 부분 -> 프론트엔드  
요청을 처리하는 부분 -> 백엔드  
프론트엔드는 웹 리소스로 구성되어 있다.  

[www.6kitt-hack.tistory.com/post](http://www.6kitt-hack.tistory.com/manage)  
라는 주소가 있다. [www.6kitt-hack.tistory.com에서](http://www.6kitt-hack.tistory.com에서)서 /post의 리소스를 요청한다.  

리소스 구성은 아래와 같다. 아래 요소에 대한 자세한 내용은 URI 첨부(비밀번호:p4p4)  
- HTML  
- CSS  
- JS  

웹 클라이언트와 서버의 통신  
통신 과정 그림이다.  
1. 클라이언트가 브라우저를 이용하여 서버에 접속한다.  
2. 브라우저가 요청을 해석하여 HTTP 형식으로 서버에 리소스를 요청한다.  
3. 서버가 HTTP 형식으로 받은 요청을 해석한다.  
4. 서버가 요청을 해석한대로 처리한다.  
5. 서버가 HTTP 형식으로 응답을 클라이언트에 전달한다.  
6. 응답 받은 HTTP 형식을 웹 리소스를 이용하여 시각화한다.  

HTTP  
http에 대해 알아보기 전 프로토콜에 대해 간략히..  
프로토콜: 통신을 하기 위해 약속한 것들..  

HTTP: hyper text transfer protocol, 데이터 교환을 요청과 응답 형식으로 정의한 프로토콜  
클라이언트가 요청 -> 서버가 응답  
웹 서버는 HTTP 서버를 HTTP 서비스 포트(보통 TCP/80 또는 TCP/8080)에 대기시킴  

### HTTP 메시지  
크게 헤더와 바디로 나누어진다.  

### HTTP 요청  
위 메시지 포맷을 기준으로 요청은 어떠한 구성을 갖고 있는지 살펴보자.  
#### in 시작 줄..  
요청은 시작 줄에 메소드, 요청 대상, HTTP 버전을 삽입한다.  
메소드: 서버가 처리하길 원하는 동작  
-> GET : 리소스 요청 메소드  
-> POST : 데이터 전송 메소드, 데이터는 바디에  

### HTTP 응답  
in 시작 줄..  
응답은 시작 줄에 HTTP 버전, 상태 코드, 처리 사유 삽입한다.  
상태코드는 아래 표로 정리  

| 상태 코드 | 설명 |
|------------|------|
| 1xx        | 요청 받고, 처리 진행중 |
| 2xx        | 요청 처리됨 (200 OK) |
| 3xx        | 클라이언트의 추가 동작 요청 (302 Found: 다른 URL 이동) |
| 4xx        | 요청을 잘못 받아서 처리 실패 (400 Bad Request, 404 Not Found) |
| 5xx        | 요청은 받았지만 처리 실패 (500 Internal Server Error, 503 Service Unavailable) |

### HTTPS  
HTTP는 응답 요청이 평문으로 전달 -> 보안성이 취약함  
HTTPS는 HTTP에 TLS 프로토콜을 넣어 보안성 향상시킴  
TLS가 HTTP 메시지를 암호화  

Quiz: web  
1. JS로 웹리소스는 서버에서 실행 -> X, 프론트엔드는 클라이언트 측  
2. 프로트엔드의 동작 구현 -> JS  
3. 웹 리소스 스타일 지정 -> CSS  
4. URI 웹 리소스 식별에 사용  
5. 브라우저는 서버에 HTTP형식으로 전달  
6. 프론트엔드는 웹 리소스로 구성  
7. 태그와 속성 등으로 문서 뼈대 구출 -> HTML  

Quiz: HTTP/HTTPS  
1. 서버에 추가 정보를 전달하는 데이터 부분 -> HTTP 헤더  
2. HTTPS 포트 번호 -> 443  
3. HTTP 포트 번호 -> 80  