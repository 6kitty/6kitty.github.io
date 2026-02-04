---
layout: post
title: "Tools: IDA + helloworld 분석"
categories: [System Hacking]
tags: [IDA, Helloworld, 분석, CTF]
last_modified_at: 2024-01-16
---

### IDA 구조
fuctions window  
IDA에서 분석한 함수 나열, ctrl+f로 원하는 함수 찾기 가능  
graph overview: 함수의 흐름 파악  
output window: 분석 과정 출력  
view: 디컴파일 결과, hex-view, 구조체 목록  

### IDA 기능
임의 주소 및 레이블 이동(jump to address): 단축키 G  
함수 및 변수 이름 재설정(rename address): 단축키 N  
Cross reference(Xref): 단축키 X  
함수 및 변수 타입 변경(enter the type declaration): 단축키 Y  
Strings 문자열 조회: 단축키 Shift+F12  
Decompile: 어셈블리를 C언어 형태로 변환  

### Exercise: Helloworld
#### 1. 정적 분석  
**아이다로 파일 열기**  

![IDA 파일 열기](https://blog.kakaocdn.net/dna/H6fvR/btsDxJy0NSv/AAAAAAAAAAAAAAAAAAAAALjllOgBK_8wwOuDDo5OjY_XqjoFwXrFqgvPkyUknhKI/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=DQKiK%2BVz%2FB2dLH8YpuJphefLbvs%3D)  

정적 분석은 main함수를 먼저 찾음  
바이너리에서 함수 찾는 방법  
1. 엔트리 포인트부터 분석하여 원하는 함수찾을 때까지 탐색  
2. 함수의 특성이나 정보를 이용하여 탐색  

**문자열 검색**  
1. shift+f12 누름 -> strings창  
2. hello, world! 찾아서 더블 클릭  

![문자열 검색](https://blog.kakaocdn.net/dna/ygJgE/btsDyCzDqbS/AAAAAAAAAAAAAAAAAAAAAKBj8zqxW5S9iE3IOt43ullQGf-WBAv_HWzAuH44kbCD/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=UJsbC6MIF8O3KXYmUf44%2Fq2Bp%2Fs%3D)  
![문자열 검색 결과](https://blog.kakaocdn.net/dna/1mpsM/btsDrGjdt7c/AAAAAAAAAAAAAAAAAAAAAOrYwGhGOfXH2t8fzQabq-HRu0cy0UJR67-amlAtsrjT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=XVXfITn7ecnzlgiuBaWPdiJQQ4M%3D)  

**상호 참조**  
수상한 값 또는 함수를 참조하는 함수 분석  
변수 클릭 후 단축키 x -> xrefs  

![상호 참조](https://blog.kakaocdn.net/dna/nBWvW/btsDuSQW4Xz/AAAAAAAAAAAAAAAAAAAAAPy2y61IxLeBnWxNIuqbn2twuPW6zUiYjJPSjy-wcsEU/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FRoIbd7EtHtRucO2jpbtcZCS7C0%3D)  
![main 함수의 모습](https://blog.kakaocdn.net/dna/bY3tiM/btsDrhDUFSv/AAAAAAAAAAAAAAAAAAAAAOYHnbY_tIV4bIMvG7qy639a2BqtVrzsLNtyr-luPDLV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=GxpeEW2%2BUAUf2%2BMkk7U0oHES6TA%3D)  

**main 함수 분석**  
f5 -> 디컴파일  

![디컴파일 결과](https://blog.kakaocdn.net/dna/IeOGF/btsDxniG7S6/AAAAAAAAAAAAAAAAAAAAAC1vIX3-aPmq5Q-__v6w6rchg_sXDuuhfpQuvMUtRFAx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FIVzA%2Beh5jB87w7SDE%2FePXEtZWQ%3D)  

인자 분석: 아이다는 argc, argv, envp 3개 인자 설정  

동작  
1. 0x3e8u만큼 sleep -> 1초 대기  
2. qword_14001dbe0에 문자열 넣기 "Hello, world!\n"  
3. sub_140001060 함수에 "Hello, world!\n"를 인자로 전달하여 호출  
4. return 0  

**sub_140001060 분석**  

![sub_140001060 분석](https://blog.kakaocdn.net/dna/tL1iS/btsDzweT3ix/AAAAAAAAAAAAAAAAAAAAAESu5pO3yi23qJrDtbRtIWAVHCaJF6nLz0NnlfiH0pDa/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=dAsrKZRoEtSf3EtJ%2FpS%2BoEMW82I%3D)  

va_start함수로 가변 인자 처리  
__acrt_iob_func 함수: 스트림을 가져올 때 사용하는 함수, 인자 1이 들어가면 stdout 의미  
문자열 인자를 받고 stdout 스트림 사용 -> printf  

#### 동적 분석  
f2 -> 중단점 설정  
f9 -> 실행  
f8 -> 한 단계 실행  
f7 -> 함수 내부 진행  
appendix -> 실행중인 프로세스 조작  