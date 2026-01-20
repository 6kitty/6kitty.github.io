---
layout: post
title: "Computer Science"
categories: [Self-study]
tags: [Computer Architecture, x86-64, Windows Memory Layout]
last_modified_at: 2024-01-15
---

Background: Computer Architecture
===============================
명령어 집합구조(아키텍처): CPU가 사용하는 명령어와 관련된 설계  
ex) x86-64 아키텍처  

컴퓨터 구조와 명령어 집합 구조
------------------------------
**컴퓨터 구조**  
컴퓨터 기능에 대한 설계(폰 노이만 구조, 하버드 구조, 수정된 하버드 구조)+(CPU가 처리해야 하는)명령어 집합 구조(ARM, MIPS, x86, x86-64)+ 마이크로 아키텍처(CPU의 하드웨어적 설계)+기타 하드웨어 및 컴퓨팅 방법 설계(직접 메모리 접근)  

**폰 노이만 구조: 기능 설계**  
연산+제어+저장  
연산과 제어는 CPU: 산술논리장치+제어장치+레지스터  
저장은 기억장치(memory)가: 주기억장치(RAM) 혹은 보조기억장치(하드 드라이브)  
데이터 및 신호 교환은 버스(bus): 데이터 버스, 주소 버스, 제어 버스(읽쓰 제어)  

![Computer Architecture](https://blog.kakaocdn.net/dna/bumJMP/btsDsZofglA/AAAAAAAAAAAAAAAAAAAAAOKCOt1fZlSI8AQtkOJCmmoMgde4ZGw24CriP6zBSJMK/img.jpg?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=phPtXMBlk0KBOu9p5Zk5fSPohco%3D)

**명령어 집합 구조**  
CPU가 해석하는 명령어의 집합, 아키텍처라고도 함(ISA)  
임베디드의 경우 ARM, MIPS, AVR  

x86-64 아키텍처
-----------------
32비트 아키텍처 -> 4기가 바이트 가상 메모리 크기  
64비트 아키텍처 -> 16엑사 바이트 가상 메모리 크기  

범용 레지스터, 세그먼트 레지스터, 명령어 포인터 레지스터, 플래그 레지스터 존재  

**1. 범용 레지스터**  
8바이트까지 저장, unsigned 정수 2^64-1  

| 이름               |                       |
|------------------|-----------------------|
| rax (accumulator) | 함수의 반환값         |
| rbx (base)       |                       |
| rcx (counter)    | 반복문의 반복 횟수, 연산 시행 횟수 |
| rdx (data)       |                       |
| rsi (source index)| 데이터 옮길 때 원본   |
| rdi (destination index)| 데이터 옮길 때 목적지 |
| rsp (stack pointer)| 사용중인 스택 위치    |
| rbp (stack base pointer)| 스택의 바닥      |

**2. 세그먼트 레지스터**  
cs, ss, ds, es, fs, gs 각각 16바이트 크기  
cs(코드), ds(데이터), ss(스택)  

**3. 명령어 포인터 레지스터**  
CPU가 실행할 부분을 가리키는 레지스터: rip  
8바이트 크기  

**4. 플래그 레지스터**  
프로세서 현재 상태 저장하는 레지스터  
x64 아키텍처에서 RFLAGS라는 64비트 레지스터 존재  

| 플래그 |                       |
|--------|-----------------------|
| CF (Carry) | 부호 없는 수의 연산 결과 > 비트 범위 |
| ZF (Zero)  | 연산 결과 0          |
| SF (Sign)  | 연산 결과 음수        |
| OF (Overflow) | 부호 있는 수 연산 결과 > 비트 범위 |

레지스터 호환 가능  

Background: Windows Memory Layout
==================================
메모리 레이아웃: 가상 메모리의 구성  

섹션
------
데이터가 모여있는 영역  
윈도우의 PE 파일: PE 헤더 + 1개 이상의 섹션  

PE 헤더에 담긴 섹션 데이터: 이름, 크기, 오프셋, 속성 및 권한  
이 헤더 정보를 참조하여 가상 메모리의 적절한 세그먼트에 매핑  

**1. .text**  
실행 가능한 기계 코드  
읽기, 실행 권한 부여, 쓰기 권한은 거의 대부분 제거  

```cpp
int main() { return 31337; }
```

**2. .data**  
컴파일 시점 값이 정해진 전역 변수 위치  
읽기, 쓰기 권한  

```cpp
int data_num = 31337;
char data_rwstr[] = "writable_data";    //data
```

**3. .rdata**  
컴파일 시점 값이 정해진 전역 상수와 참조할 DLL 및 외부 함수들 정보  
읽기 권한  

```cpp
const char data_rostr[] = "readonly_data";
char *str_ptr = "readonly"; //str_ptr은 .data, 문자열("readonly")은 .rdata
```

섹션 외 메모리
----------------
**1. 스택**  
각 스레드 본인만의 스택 존재  
지역 변수, 함수의 리턴 주소 저장  
읽기, 쓰기 권한  
아래로 확장: 낮은 주소로 업데이트  

**2. 힙**  
모든 종류 데이터 저장 가능  
비교적 스택보다 큰 데이터 저장 가능  
전역적 접근 가능, 동적 할당  

```cpp
int main() {
    int *heap_data_ptr = malloc(sizeof(*heap_data_ptr));   //동적 할당한 힙 영역 주소를 가리킴 
    *heap_data_ptr = 31337;    //힙 영역에 값 작성 
    return 0;
}
```
heap_data_ptr은 스택, malloc으로 할당받은 힙 영역을 가리킴  