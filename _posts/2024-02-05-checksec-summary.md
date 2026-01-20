---
layout: post
title: "checksec 정리"
categories: [Self-study]
tags: [checksec, security, analysis, CTF]
last_modified_at: 2024-02-05
---

아무거나 사진 하나 가져옴 checksec으로 확인할 수 있는 분석 기법..  
괄호 숫자는 드림핵에서 배운 순서..  

### 1. Arch (1)  
아키텍처 확인.. 엔디언 처리가 필요하니 꼭 확인해줘야 함  
*아키텍처: 컴퓨터 시스템의 하드웨어 구조, CPU, I/O 등 기계적 구조 설계 방식*  
헷갈렸던 pwntools에서 패킹, 언패킹 개념도 짚고 넘어가겠음  

**Packing & Unpacking**  
리틀 엔디언 처리 혹은 역변환..  
packing: 정수를 인자로 받아 문자열 형태로  
ex) p32 사용.. 0x41424344 -> b'DCBA' (리틀 엔디언 영향)  
unpacking: 문자열을 인자로 받아 정수 형태로  
ex) u32 사용.. 'ABCD' -> 0x44434241 (출력할 때는 hex 한 번 더 체크해줘야 0x 값으로 나옴)  

### 2. RELRO (5)  
1. Partial RELRO -> GOT Overwrite  
2. Full RELRO -> Hooking  

### 3. Stack Canary (2)  
버퍼와 반환 주소 사이 카나리 삽입..  

**카나리 릭의 원리**  
널바이트까지 덮어주면 문자열 출력 함수들은 문자열 끝을 인식하지 못해 카나리값까지 릭  
카나리의 첫 바이트가 \x00  

### 4. NX (3)  
코드 영역 외에 실행 권한을 제거하는 기법  
이러면 스택에 실행 권한이 없어서 덮어써도 소용이 없음  
-> 코드 영역에 반환 주소를 덮어야 함  

PLT에서 GOT 참조할 때 취약점을 이용하여 ROP 공격  

### 5. PIE (4)  
코드 영역에도 ASLR 적용하는 기술  
*ASLR: 실행할 때마다 스택, 힙, 라이브러리 등을 무작위 주소에 할당하는 기법*  

**PIE 우회 방법**  
1. 코드 베이스 구하기  
2. partial overwrite  