---
layout: post
title: "함수 호출 규약"
categories: [System Hacking]
tags: [Calling Convention, cdecl, SYSV, x86, x86-64]
last_modified_at: 2024-01-25
---

### Background: Calling Convention

함수 호출 규약: 함수의 호출 및 반환에 대한 약속

한 함수에서 다른 함수 호출 -> 실행 흐름 옮기기 -> 호출한 함수 반환 -> 실행 흐름 옮기기

이 과정에서 Caller의 stack frame과 return address 저장 필요 <- 이래야 반환 후 실행 흐름을 옮길 수 있음

호출 시 Callee가 요구하는 인자 전달 필요, 반환 시 Caller에게 반환값 전달 필요

컴파일러가 함수 호출 규약 적용

이 호출 규약은 다양함. 프로그래머가 따로 명시하지 않는 이상, CPU 아키텍처에 따른 호출 규약 적용

1. x86(32bit): 레지스터 적음 -> 스택으로 인자 전달
2. x86-64: 레지스터 많음 -> 레지스터로 인자 전달이 주

![함수 호출 규약 종류들...](https://blog.kakaocdn.net/dna/eDgo8P/btsDRlyYEfB/AAAAAAAAAAAAAAAAAAAAAH3qr0rtquHNxbDqmn-AOnxWmALz6uUmWIXhqaX2ypAM/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MgFjlaHca91EW9G9cNYGuzbRx1I%3D)

if 컴파일러 도움 없이 어셈블리 코드 작성 혹은 읽으려면 호출 규약을 알아야 함

1. cdecl: x86
   - 스택을 통해 인자 전달
   - 인자 전달을 위해 caller가 스택 정리
   - 스택이 아래로 자라므로 마지막 인자부터 거꾸로 push

#### cdecl 함수 호출 규약 실습

```c
// Name: cdecl.c
// Compile: gcc -fno-asynchronous-unwind-tables -nostdlib -masm=intel \
//          -fomit-frame-pointer -S cdecl.c -w -m32 -fno-pic -O0

void __attribute__((cdecl)) callee(int a1, int a2){ // cdecl로 호출
}

void caller(){
   callee(1, 2);
}
```

```assembly
; Name: cdecl.s
.file "cdecl.c"
.intel_syntax noprefix
.text
.globl callee
.type callee, @function
callee:
nop
ret ; 스택을 정리하지 않고 리턴합니다.
.size callee, .-callee
.globl caller
.type caller, @function
caller:
push 2 ; 2를 스택에 저장하여 callee의 인자로 전달합니다.
push 1 ; 1를 스택에 저장하여 callee의 인자로 전달합니다.
call callee
add esp, 8 ; 스택을 정리합니다. (push를 2번하였기 때문에 8byte만큼 esp가 증가되어 있습니다.)
nop
ret
.size caller, .-caller
.ident "GCC: (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0"
.section .note.GNU-stack,"",@progbits
```

### 2. SYSV: x86-64

1. 6개의 인자를 RDI, RSI, RDX, RCX, R8, R9에 순서대로 저장 후 전달, 이후 인자들은 스택 사용
2. Caller에서 인자 전달에 사용된 스택을 정리
3. 반환값은 RAX로 전달

**sysv 함수 호출 규약 실습**

```c
#define ull unsigned long long

ull callee(ull a1, int a2, int a3, int a4, int a5, int a6, int a7) {
  ull ret = a1 + a2 + a3 + a4 + a5 + a6 + a7;
  return ret;
}

void caller() {
  callee(123456789123456789, 2, 3, 4, 5, 6, 7);
}

int main() {
  caller();
}
```

```bash
$ gcc -fno-asynchronous-unwind-tables -masm=intel -fno-omit-frame-pointer -o sysv sysv.c -fno-pic -O0
```

분석 코드와 컴파일 방법

![함수 호출 규약 분석](https://blog.kakaocdn.net/dna/bEgL0O/btsDWTgXljs/AAAAAAAAAAAAAAAAAAAAAOpyrRIXRmS4Yz1haTIDLA6zoO5cUdfLfWZXrQhhOKoW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ZPL1eVPPOcvXzo%2FyEy2t89Il1Tc%3D)

caller에 중단점 설정 후 run

![중단점 설정](https://blog.kakaocdn.net/dna/BGwLb/btsDW9w57PZ/AAAAAAAAAAAAAAAAAAAAANpp-Ug5m_VqiEstWWq5eM2BMgGY5uDSycxD3HFh57pf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=LQIszd%2FG668RELqJBQK0a7Kv5bU%3D)

caller+10부터 caller+37까지 인자 설정, r9d부터 rdi순으로 넣음

근데 그전에 7을 먼저 push함 -> 이건 스택으로 보냄

최대 6개 인자까지 레지스터에 넣고 그 이상은 stack으로 보내나봄

**인자 전달 거꾸로 하는 이유(레지스터는 거꾸로 넣을 필요 없지 않나..?)**

![스택 구조](https://blog.kakaocdn.net/dna/bUj59X/btsDUcVVeKR/AAAAAAAAAAAAAAAAAAAAACfbCtomRWVLenvRdWCGv4QXCnUnRdYNUVGB8vlwOxGq/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Mj%2FKKLeK1NfZgU3iH10IKXnVxh8%3D)

![callee 호출 부분](https://blog.kakaocdn.net/dna/bN5XV5/btsDXzoONHG/AAAAAAAAAAAAAAAAAAAAAJsWO-mgogKWHw64JbZAH3vgUYZcOKbtIRAS5YrvYGdh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=4eBa%2BolW4VOnUYgzAsHZer5KtFQ%3D)

callee의 인자들 확인

RDI부터 R9까지 그리고 7은 스택의 RSP(사용중인 스택 위치)

![스택 상태](https://blog.kakaocdn.net/dna/kwzcb/btsDUfE7dME/AAAAAAAAAAAAAAAAAAAAALUQ0pHu1lkzxnhOa8eDnYNfuOqZz_qzlFwAm6_ff2NN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WZsWjUqomdw8DzRr5F56Du%2F38Pg%3D)

si 이후 스택 rsp를 보면 반환 주소가 저장되어 있음

![callee 도입부](https://blog.kakaocdn.net/dna/D608p/btsDUiaNowB/AAAAAAAAAAAAAAAAAAAAAJqxJBYzlzNRv6ahFaMKbr-fAJhF-m-P6NOVyF9R_k3y/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vsWAfDnYysKX%2FW%2F7whs1BkATzhk%3D)

$rip부터 9줄 열람하면 push rbp로 반환 주소를 저장함

rbp는 SFP(Stack Frame Pointer)이라고도 함

![push rbp 이후 스택](https://blog.kakaocdn.net/dna/6IHZB/btsDWJyFUTv/AAAAAAAAAAAAAAAAAAAAAItFKOoVv16Cf2AgpJBzn53u-zvbwM_xS2ZEZLRWI4OD/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=8Ht3xRtqTCVAoFQvIS65RnYvf1k%3D)

그림 a의 RSP값에 0x7fffffffe130이 저장됨(rbp의 주소)

![rsp 상태](https://blog.kakaocdn.net/dna/nr06u/btsDWRjbWUc/AAAAAAAAAAAAAAAAAAAAAJO2GpiwUK6wLNEoR5sWAcDl2TCxdNSsQhmNykuw0B1N/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ydLJ0CzgK3vuzOfwmrr1JUmMgE4%3D)

그림 b랑 비교해보고 어떻게 정렬되는지 확인해보기

![mov rbp, rsp 진행 후](https://blog.kakaocdn.net/dna/cO9mIm/btsDRbKg9HE/AAAAAAAAAAAAAAAAAAAAAFGfPuq5-GPc8u0WTSYyPruuCWpsQaZaKwVZhYlzSps3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=912FJHK72kt3CysktdbInLgNlHI%3D)

mov rbp, rsp 때문에 rbp와 rsp는 같은 값을 가르키게 됨

이후 지역 변수 필요 시 -> rsp의 값을 빼서 아래로 공간 할당

(이 실습에서는 지역 변수가 없기 때문에 공간 할당을 하지 않음)

![ret가 반환](https://blog.kakaocdn.net/dna/bfmieM/btsDRLEvTX0/AAAAAAAAAAAAAAAAAAAAALaADDrG6J70M04GQMxJ7DBuIoqHnAJaEDzYhCtg3kDw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=bjJXjw1DF%2FdegHqiCq4YIOfSAlA%3D)

ret가 반환, callee+91에서 반환하는 걸 알 수 있음

![callee+91에서의 rax값](https://blog.kakaocdn.net/dna/HiLqK/btsDWqF8wyn/AAAAAAAAAAAAAAAAAAAAANL32P77K9pR4r9e_Te9jIfSx9RrB_xPcoGlBezz5PhE/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ECNEda419%2FJhV9p8e%2F8NC1byEAI%3D)

![지우고 90에 중단점 설정](https://blog.kakaocdn.net/dna/vrjTA/btsDXABgU3A/AAAAAAAAAAAAAAAAAAAAAAKSLPGDWCzXEGQ4MqSXCckjZmAtlQ_gCL1pnmfnUU-H/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7xNykjqadd%2BCYpShea0WJu0VZcU%3D)

![pop rbp로 스택 프레임 꺼냄](https://blog.kakaocdn.net/dna/dpkQO5/btsDUVl5Gjo/AAAAAAAAAAAAAAAAAAAAAHSE0flkpro-E5Ma8hI7-CHmq0cyS7Uu6-XPt2AzCZmA/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=iLJiJ3gZbU%2Fl9T7VmUnQdhvS3BI%3D)

원래는 leave 사용, 이 실습에서는 따로 할당해준 게 없기 때문에 pop으로도 가능

### Quiz: Calling Convention

4번 자꾸 틀려서 찬찬히 봐야할듯

```c
// Name: callconv_quiz.c
// Compile: gcc -o callconv_quiz callconv_quiz.c -m32
int __attribute__((cdecl)) sum(int a1, int a2, int a3){
	return a1 + a2 + a3;
}
void main(){
	int total = 0;
	total = sum(1, 2, 3);
}
```

main:   
   0x080483ed <+0>:	push   ebp  
   0x080483ee <+1>:	mov    ebp,esp  
   0x080483f0 <+3>:	sub    esp,0x10  
   0x080483f3 <+6>:	mov    DWORD PTR [ebp-0x4],0x0  
   0x080483fa <+13>:	(a)  
   0x080483fc <+15>:	(b)  
   0x080483fe <+17>:	(c)  
   0x08048400 <+19>:	call   0x80483db <sum>  
   0x08048405 <+24>:	(d)  
   0x08048408 <+27>:	mov    DWORD PTR [ebp-0x4],eax  
   0x0804840b <+30>:	nop  
   0x0804840c <+31>:	leave  
   0x0804840d <+32>:	ret  

cdecl호출 문제였음.. 하하~ 어쩐쥐~