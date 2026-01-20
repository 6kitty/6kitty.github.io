---
layout: post
title: "Binary & Analysis"
categories: [SWING, Writeup]
tags: [Reversing, Static Analysis, Dynamic Analysis, Malware Analysis, Compiler, Disassembly]
last_modified_at: 2024-01-14
---

리버싱의 쓰임
- 악성코드 분석
- 보안성 검사
- 프로그램 보안 패치

### Background: Binary
**기계어 -> 어셈블리어 -> 고급 언어**  
프로그램: 연산 장치가 수행하는 일련의 동작들을 정리한 문서  
programmable, non-programmable  

#### 컴파일러와 인터프리터
소스 코드를 기계어 형식으로 번역 : 컴파일  
ex) GCC, Clang  
Python, javascript 등은 컴파일 필요 x -> 인터프리팅  

#### 컴파일 과정
C언어: 전처리 -> 컴파일 -> 어셈블 -> 링크  
초반 C언어 코드와 헤더 파일:
```c
//name: add.c

#include "add.h"
#define HI 3

int add(int a, int b) { return a+b+HI; } //return a+b


//name: add.h

int add(int a, int b);
```

**전처리**  
1. 주석 제거  
2. 매크로 치환  
#define으로 정의한 매크로: 자주 쓰이는 코드나 상숫값 정의  
전처리 과정에서는 이걸 값으로 치환함  
3. 파일 병합  
일부 경우에서는 전처리 단계에서 소스와 헤더 파일 합함  
```c
//name: add.i

1 "add.c"
1 "<built-in>"
1 "<command-line>"
31 "<command-line>"
1 " /usr/include/stdc-predef.h" 1 3 4
32 "<command-line>" 2
1 "add.c"
1 "add.h" 1
int add(int a, int b);
2 "add.c" 2

int add(int a, int b) {return a+b+3;}
```

**컴파일:** C로 작성된 소스 코드 -> 어셈블리어  
gcc에서 지원하는 옵션: -0 -00 -01 -02 -03 -0s -0fast -0g  
반복문 컴파일: x가 가질 값 직접 계산 -> 이를 대입  
ex)  
```c
//name: opt.c
//compile: gcc -o opt opt.c -02

#include <stdio.h>

int main() {
	int x = 0;
    for(int i=0;i<100;i++) x +=1; //x에 0부터 99까지의 값 더하기
    printf("%d",x);
}
```
주소 빼고 어셈블리어만 작성함  
```assembly
lea rsi,[rip+0x1bd]    ; 0x724
sub rsp,0x8
mov edx,0x1356    ; hex((0+99)*50) = '0x1356' = sum(0,1, ... ,99)
mov edi,0x1
xor eax,eax
call 0x540 <__printf_chk@plt>
xor eax,eax
add rsp,0x8
ret
```

-S 옵션을 이용하면 어셈블리 코드로 컴파일  

**어셈블:** 어셈블리어 코드 -> ELF 형식의 목적 파일(Obj)  
ELF: 리눅스의 실행파일 형식  
윈도우의 경우 PE형식으로...  
gcc -c add.S -o add.o 명령어 입력  

**링크:** 여러 목적 파일(obj) 연결 -> 바이너리 제작  
```c
//name: hello-world.c
//compile: gcc -o hello-world hello-world.c

#include <stdio.h>

int main() { printf("hello, world!"); }
```
printf 함수의 정의는 hello-world.c에 존재 X, libc 라이브러리에 존재  
링커가 libc 안 printf를 실행할 수 있도록 연결해줌  
gcc add.o -o add -Xlinker add.o를 링크하는 명령어  
다만, 소스 코드에 main함수 정의가 없으므로 아래 옵션을 추가함  
--unresolved-symbols  

#### 디스어셈블과 디컴파일
디스어셈블: 바이너리 -> 어셈블러  
objdump -d ./add -M intel  
디컴파일: 어셈블리 -> 고급 언어  
일대일 대응이 어려움  

### Background: Static Analysis vs. Dynamic Analysis  
#### 정적분석
프로그램 실행 X  

장점  
1. 전체구조 파악  
2. 분석 환경의 제약을 비교적 덜 받음  
3. 악성 프로그램 감염 X  

단점  
1. obfuscation 적용 -> 분석 어려워짐  
2. 동적 요소 고려 어려움  

#### 동적분석
프로그램 실행 O  

장점  
프로그램의 개략적인 동작 파악 가능  

단점  
분석 환경 구축  
안티 디버깅 -> 동적 분석 어려워짐  