---
layout: post
title: "Web Browser"
categories: [SWING]
tags: [Web, Browser, HTTP, DNS, Rendering, Devtools]
last_modified_at: 2024-12-28
---

웹 브라우저: 서버와 HTTP 통신을 대신 해줌 그리고 시각화해서 클라이언트에게 보여줌  

주소를 입력했을 때 웹 브라우저가 하는 역할  
1. URL 분석  
2. DNS 요청  
3. HTTP포맷 요청 전송  
4. HTTP포맷 응답 수신  
5. 리소스 다운 및 렌더링  

![웹 브라우저 과정](https://blog.kakaocdn.net/dna/bwHmIb/btsLx7mnbev/AAAAAAAAAAAAAAAAAAAAABrQUJZGW50NDw3oEeNa1G9MWFxJ2f_jprJBW3jXdEGo/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uqAxcpdAewCatnAk6ZX%2BYrPM2kI%3D)

## URL  
웹에 있는 리소스 위치 표현  
브라우저는 URL을 이용하여 서버에 웹 리소스 접근  

![URL 구조](https://blog.kakaocdn.net/dna/Sjr8v/btsLAvMS4ol/AAAAAAAAAAAAAAAAAAAAAJ-5KJNOhlLje5vBX-JCSh4HN7dKMquVjAHx6Q_pqQpY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vOt5Cg4yY6jpA0R5O0IdHiKpKDs%3D)

위 그림은 URL의 구조를 표현한다. scheme, authority, path, query, fragment의 자세한 역할은 아래 표로 정리해두었다.

| 요소      | 설명                                      |
|-----------|-----------------------------------------|
| scheme    | 통신하는 프로토콜 명시                     |
| authority | 접속할 웹 서버의 호스트 주소와 포트 번호     |
| query     | 서버에 전달하는 파라미터, '?'로 구분        |
| fragment  | 메인 리소스 내 서브 리소스 접근 시 사용, '#'로 구분 |

## Domain name  
Host는 도메인 이름 또는 ip 주소의 값이 들어간다. 이때 도메인은 ip 주소 대신 사용할 수 있는 별명이라고 생각하면 편하다. 다만 도메인 사용 시에는 서버 접속할 때 DNS(Domain Name Server)를 거쳐서 ip 주소를 응답받고 해당 주소로 접속하는 절차이다. 정리하면 아래와 같다.  
1. Host에 도메인 이름이 적힌 URL 입력  
2. DNS에 도메인 주소 전달, ip 주소 받기  
3. 받은 주소로 서버 접속  

## 웹 렌더링  
받은 리소스를 시각화 하기 위해서는 웹 렌더링이 필요하다.  
브라우저별로 웹 렌더링을 위한 렌더링 엔진이 다르다.  

| 브라우저   | 사용 엔진   |
|------------|-------------|
| 사파리     | 웹킷(Webkit) |
| 크롬       | 블링크      |
| 파이어폭스 | 개코(Gecko)  |

## Browser Devtools  
개발자 도구 -> 브라우저를 띄워둔 상태에서 단축키 f12  

![개발자 도구](https://blog.kakaocdn.net/dna/clfRnV/btsLAfxyw6T/AAAAAAAAAAAAAAAAAAAAAI4v5zr7j9cth2uJMYRmK0Dq9n1nrRh-PgUJ-QPw1-T-/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=79%2FCM2dyCAafNR%2BTvMDLcLmqMNI%3D)

위 카테고리는 기능을 선택하는 패널을 뜻한다. 중요 기능들만 짚어보자  

### Elements  
현재 페이지 HTML 코드를 읽을 수 있다. HTML은 코드..?보다는 일종의 문서라고 이해하면 편한 거 같다.  
코드를 선택하고 F2키 또는 더블클릭하면 코드를 수정할 수 있다.  

### Console  
콘솔창에서는 JS 코드에서 발생한 메시지를 출력하고, 입력한 JS 코드도 실행해준다.  
console 오브젝트에 개발자 도구 콘솔에 접근할 수 있는 함수가 포함되어 있다.  

![콘솔](https://blog.kakaocdn.net/dna/TDNC8/btsLz9KH11U/AAAAAAAAAAAAAAAAAAAAAFHRlw_g5kctkCNSmYkFLUiKFoSDmgd9KWZM0fjzAFwC/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=YvECiglNqtRYtGbRp2Eh2qk9F6s%3D)

위와 같이 입력해주면 hello 문자열을 담은 창이 나타난다.  

### Sources  
웹 리소스들을 확인할 수 있는 창이다. 3분할로 되어있다.  

| 설명                                      |
|-----------------------------------------|
| 현재 페이지의 리소스 파일 트리, 파일 시스템 |
| 왼쪽에서 선택한 리소스 상세 보기             |
| 디버깅 정보                               |

오른쪽 디버깅 정보에 있는 요소도 짚고 넘어가자  

| 요소         | 설명                                      |
|--------------|-----------------------------------------|
| Watch        | JS 식 입력 -> 식의 값 변화 확인             |
| Call Stack   | 함수들 호출 순서 -> 스택 형태로 정리        |
| Scope        | 정의된 변수들 값 확인                       |
| Breakpoints  | 중단점 확인, on/off 설정                   |

### Sources- Debug  
드림핵 실습 페이지에서 source 탭 실습을 진행해보자.  

![소스 디버그](https://blog.kakaocdn.net/dna/DNp8E/btsLzohkMDy/AAAAAAAAAAAAAAAAAAAAANMt50MDB9Z1mEe4KfYdt7JMjH5SiLlmLSOk1wRC7y6L/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2Fb48DqSG7V%2Fd7sIoWQvv%2BmGHy1Q%3D)

source창에서 코드 줄에 중단점을 걸면 Breakpoints에 위 같이 나온다.  

![중단점](https://blog.kakaocdn.net/dna/bx91VT/btsLAVlaX3D/AAAAAAAAAAAAAAAAAAAAALDdJwLpu42d5KosnjNFBp3l9-j-MfYenn3T1vV65TS8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=pjuqqxcbGyiQYX%2FW6Yzt7dcD%2FGk%3D)

name 검사 부분에 중단점을 걸고 click 버튼을 누르면 위와 같이 디버깅을 할 수 있다. name에 hi를 입력해준 것이 debug창 scope에도 출력된다.  

### Network  
서버와 오가는 데이터를 확인할 수 있다.  

![네트워크](https://blog.kakaocdn.net/dna/b7vv8M/btsLzoInrRK/AAAAAAAAAAAAAAAAAAAAAB_pXNuQFWg_6_SPowm_Oa7DobHKIPrWT6B1RywqmuNf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=59peaIQOTiG6LBEv2%2Fh20wJXEJo%3D)

### Network- Copy  
원하는 데이터를 우클릭>copy>copy as fetch 후에  
console 패널에 붙여넣기하면 동일한 요청을 서버에 재전송할 수 있다.  

![복사](https://blog.kakaocdn.net/dna/V4bWu/btsLAXceC44/AAAAAAAAAAAAAAAAAAAAAKwMlnstBPqbBx_oVoeM41XZXTAiWtKiR6ALmUGW_CoY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=r6Z2zyaYsC%2BkHHwQv7RQOtWXc%2Bc%3D)

아래 콘솔창에 요청을 붙여넣기 한 화면이다.  

### Application  
웹 어플리케이션과 관련된 리소스를 열람할 수 있다.  

![어플리케이션](https://blog.kakaocdn.net/dna/Yfk5n/btsLBKJZrlH/AAAAAAAAAAAAAAAAAAAAAEd_XXkLwM_nnC4MMp1YJI4RbS8pu-2Kx3J1n2jwTUj4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=KZWETg%2BhDHnD%2BFC5UkAeOVV9ECo%3D)

### 페이지 소스 보기  
windows 기준 ctrl+u를 누르면, 새 창으로 페이지 소스 코드를 볼 수 있다.  

![페이지 소스](https://blog.kakaocdn.net/dna/eMn6jF/btsLAhhVeXr/AAAAAAAAAAAAAAAAAAAAAL6dXfQurgBdzcW9c0imAz79MJEhbdSJwPCcpprpu1w3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=oKN%2BXaf5X7hyPXYwGnQxSl%2F%2BkdI%3D)

### 시크릿모드  
시크릿모드로 생성한 브라우저 종료 시 저장되지 않는 목록은 다음과 같다.  
1. 방문 기록  
2. 쿠키 및 사이트 데이터  
3. 양식에 입력한 정보 -> 쿠키가 없기 때문인가?  
4. 웹사이트에 부여된 권한  