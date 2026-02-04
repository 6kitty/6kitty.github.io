---
layout: post
title: "버그 헌팅: 향상된 타겟 정보 수집"
categories: [Web Hacking]
tags: [bug hunting, reconnaissance, security, tools, techniques]
last_modified_at: 2025-04-23
---

1. 수동탐색 : 직접 써보기  
MTSM..  

2. 구글해킹 :  
site  
inurl  
intitle  
link  
filetype  
Wildcard(*) 해킹 * 배우는 법이라고 하면 해킹과 배우는 법 사이 모든 문자가 옴  
큰따옴표 공백 포함해서 검색하고 싶을 때  

admin site:target.com 이런 식으로 검색  
inurl로 알지 못했던 도메인도 발견 가능  
intitle 디렉토링 리스팅 취약점  
link 특정 링크를 포함한..  

실제 버그헌팅 할 때) targetSystem.com을 헌팅할 때  
site:*.targetSystem.com ->  
site:*.targetSystem.com inurl: admin  
site:s3.amazonaws.com  
site:*.targetSystem.com ext:txt password  
inurl:/etc/passwd root:x:0:0:root:/root:/bin/bash -> LFI 취약점 검색  
intitle: index of  

GHDB 사이트 활용  

3. 스파이더링  
ZAP (OWASP Zed Attack Proxy)  
버프 스위트랑 비슷한 툴  
Web Spidering/Web Crawling -> 사이트에서 확인할 수 있는 모든 페이지들을 식별하는 것  
api 다 보여줌  

![우리 학교 사이트 넣어봄](https://blog.kakaocdn.net/dna/WsxrP/btsNwRgcQeR/AAAAAAAAAAAAAAAAAAAAAGj0bltSHEYkYnxD0_hF3mhkSjVVChViBzXBwdLT9J9H/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=M4gTJYcUNmRDCG5ZBQokBBHGJag%3D)  

4. S3 bucket  
숨겨진 서버, 로그, 크리덴셜, 사용자 정보, 소스코드  
DB 구축 대신 쓰거나 백업용서버로 사용  
site: s3.amazonaws.com [Target]  
site:amazonaws.com [Target]  

GrayhatWarfare이란 툴 -> 웹으로 실행 가능  
Bucket Stream -> CT로 도메인 이름 찾아냄 (CT: 인증서를 감시/관리하는 시스템)  

5. github recon  
타겟 회사 이름, 프로젝트 이름, 개발자 이름(닉네임) 조사 후 검사  
Issues / Code commits 현황 조사  
Code History 및 Blame 내역 활용  
-> API key, 암호화 key, DB 비밀번호 등 찾을 수 있음  
키워드는 key, secret, password, cred 등등...  

Gitrob  
TruffieHog  

6. Domain Enumeration  
인증서의 SAN(subject Alternative name) 조사  
SAN: 1개의 인증서에 여러 도메인 등록  
리눅스에 amass 툴 사용으로 도메인 enumeration 가능  