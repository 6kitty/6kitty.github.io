---
layout: post
title: "PIE & RELRO"
categories: [SWING, Writeup]
tags: [ASLR, PIE, RELRO, ROP, CTF]
last_modified_at: 2024-02-03
---

### Background: PIE
ASLR 적용 -> 스택, 힙, 공유 라이브러리가 실행할 때마다 무작위로 매핑됨..  
but 코드 영역의 main 함수 주소는 매번 같음..  
때문에 코드 가젯 활용해서 ROP..  

PIE는 코드 영역에도 ASLR이 적용되도록 하는 기술..  

#### PIC
리눅스 ELF는 실행 파일 혹은 공유 오브젝트 두 종류 존재..  
addr 같은 실행 파일..  
libc.so같은 공유 오브젝트..  

공유 오브젝트는 relocation 가능  
relocation이 가능하다 -> 메모리 어느 주소에 있어도 코드 훼손 X  
이러한 성질을 Position-Independent Code(PIC)  

```shell
$ file addr
addr: ELF 64-bit LSB executable
$ file /lib/x86_64-linux-gnu/libc.so.6
/lib/x86_64-linux-gnu/libc.so.6: ELF 64-bit LSB shared object
```

#### pic 코드 분석
"%p" 문자열을 전달하는 방식이 다르댔음  
no_pic은 절대주소로 참조.. pic은 rip+0xa2로 참조..  
pic는 상대 참조를 하기 때문에 바이너리가 무작위 주소에 매핑되어도 제대로 실행 가능  

![pic 코드 분석](https://blog.kakaocdn.net/dna/bzOY35/btsEpmWP07P/AAAAAAAAAAAAAAAAAAAAAFuvWl5fbhNDYZnV6Xu5IwueCWJZg6S6MQ5_fk8eAFYE/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ugoK0Nweszt4F7i%2F5TExpqudoec%3D)

#### PIE
Position-Independent Executable(PIE)는 무작위 주소에 매핑되어도 실행 가능한 실행 파일을 뜻함  
code와 executable 차이.. pic과 pie의 차이.. 기억하기  
공유 오브젝트를 실행 파일로 사용..  
ex) 리눅스의 기본 실행 파일 /bin/ls 파일 헤더.. Type이 공유 오브젝트(DYN)으로 설정되어 있음  

```shell
$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6ab0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          136224 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

#### PIE on ASLR
PIE의 재배치 가능한 성질.. ASLR이 적용된 시스템에서 실행 파일도 무작위 주소에 적재..  
ASLR 적용되지 않으면 PIE를 가진 바이너리여도 무작위 주소 적재 X..  

![PIE on ASLR](https://blog.kakaocdn.net/dna/bbABdZ/btsEkSDeW8S/AAAAAAAAAAAAAAAAAAAAABz09DP_q6JFnkcGz7rT_9jtsja1acjTMZseVX7tov0W/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=VzWBCKyxuQc4aElBpUlEnID0eB4%3D)  
mitigation:ASLR&NX에서 썼던 주소를 보여주는 코드 재활용..  
이번엔 main함수의 주소가 실행할 때마다 바뀌고 있음  

#### PIE 우회
**1. 코드 베이스 구하기**  
코드 영역 가젯을 사용하거나 데이터 영역 접근하려면 바이너리가 적재된 주소를 알아야 함!!  
-> 이 주소를 PIE 베이스, 코드 베이스라고 함  
코드 베이스를 구하려면 libc_base처럼 코드 영역 임의 주소를 읽고 그 오프셋을 빼야함  
ROP에서 했던 libc_base 내용과 비슷..  

**2. Partial Overwrite**  
코드 베이스를 구하기 어려울 때 ret 일부분만 덮는 공격 고려..  
함수의 ret는 Caller의 내부를 겨냥.. 어느정도 예측할 수 있음  
ASLR 특성 상, 코드 영역의 주소도 하위 12비트 값은 항상 동일  
if 사용하려는 코드 가젯의 주소와 ret와 하위 한 바이트만 다르다면, 일부분만 덮어서 코드 실행 가능  
if 두 바이트 이상 차이.. 브루트 포싱  

### Background: RELRO
ELF는 GOT를 활용하여 반복되는 함수 호출 비용 줄임..  
Background:Library에서 Lazy Binding 소개..  
이 Lazy Binding은 GOT에 쓰기 권한 부여.. -> 바이너리를 취약하게 함  

ELF의 데이터 세그먼트에 .init_arrary, .fini_array 등이 조작되면 프로세스 실행 흐름에 혼란..  

이를 방지하고나 데이터 세그먼트를 보호하는 RELocation Read-Only(RELRO) 개발  
RELRO는 쓰기 권한이 불필요한 데이터 세그먼트에 쓰기 권한 제거  

#### RELRO의 종류
1. Partial RELRO: 부분적 적용  
2. Full RELRO: 가장 넓은 영역에 적용  

#### Partial RELRO
```cpp
// Name: relro.c
// Compile: gcc -o prelro relro.c -no-pie -fno-PIE
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
  FILE *fp;
  char ch;
  fp = fopen("/proc/self/maps", "r");
  while (1) {
    ch = fgetc(fp);
    if (ch == EOF) break;
    putchar(ch);
  }
  return 0;
}
```
실습을 위한 코드..  

![Partial RELRO](https://blog.kakaocdn.net/dna/cRtBjQ/btsEmU77Xb0/AAAAAAAAAAAAAAAAAAAAAJEBTVi1GgK8AD3RSP3WmtzMEYu551TDi1wR4Qq8IP57/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=HorqvMdi%2FUjN%2FDjovERmnyU9MWw%3D)  
partial RELRO 확인  

#### Partial RELRO 권한
![Partial RELRO 권한](https://blog.kakaocdn.net/dna/bj5isT/btsEljmQSU8/AAAAAAAAAAAAAAAAAAAAAHaNyFunp8iVWnCUiMpd-usOp6esl6WNUlx_TXdOxdyc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=m8En6%2BfNvdzpes%2BUOgJB3L59WBQ%3D)  
쓰기 권한 확인 0x404000~0x405000  
위 주소에 어떤 부분이 할당되어 있냐면..  
objdump -h ./prelro 명령어 입력  

![objdump 결과](https://blog.kakaocdn.net/dna/eowjWe/btsEj7AtZCg/AAAAAAAAAAAAAAAAAAAAAOoRgJzzbJnO_L8VkldfOLIFqjqWY3aDyPkFHWLN6YE_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=cCHgFx%2BMSLC17ot6jdnGukVLYNg%3D)  
23부터 25까지.. 해당 주소에 포함되어 있는 걸 확인  
반면 .init_array와 .fini_array가 포함된 주소를 확인해보면 w권한 없는 걸 알 수 있음  
.got와 .got.plt 관련한 글은 접은 글  

실행 중에 바인딩되는 변수는 .got.plt에 위치..  
lazy binding을 실행해야 하므로 쓰기 권한 부여  

바이너리가 실행될 때 이미 바인딩이 끝나있는 변수는 .got에 위치(전역 변수)  
바인딩이 이미 완료되었으므로 쓰기 권한 부여 X  

#### Full RELRO
![Full RELRO](https://blog.kakaocdn.net/dna/nDNz3/btsEppMNSrZ/AAAAAAAAAAAAAAAAAAAAAEhIcybC3QTCLg4b9vdDd8WXyl7Trt4QFAp-SU6tMno0/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2F2tHBZCWyzvQWh5FZxGw65jg7Ps%3D)  
![Full RELRO](https://blog.kakaocdn.net/dna/coyxXe/btsEoM87r0s/AAAAAAAAAAAAAAAAAAAAAGmCgic_hV_zmsyFLXj06IRwVepGwi7k167BWfwz9-iB/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Yyr3l9cmKpHcpDEsdwWDmH20ZPs%3D)  
PIE 때문에 frelro가 0x55eb72639000에 매핑됨  
쓰기 권한이 있는 곳은 0x55eb7263d000~0x55eb7374d000  
frelro 코드 베이스값만큼 빼주면 오프셋 범위가 0x4000부터 시작..  

![frelro 오프셋](https://blog.kakaocdn.net/dna/dnc1Cb/btsEnXbZ7kv/AAAAAAAAAAAAAAAAAAAAAD1-VZmVbluN9VSVSHZIiOd5Yc6Z512l4yEV2BN2kAjc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=KvENV%2Biis8%2FTtY9qEdBAdCtawyA%3D)  
data의 오프셋이 0x4000이므로 쓰기 권한 안에 포함됨  
bss 섹션도 포함..  

Full RELRO가 적용되면 라이브러리 함수들의 주소가 바이너리 로딩 시점에 모두 바인딩됨..  
-> GOT에 쓰기 권한 필요 없음  

#### RELRO 우회
Partial RELRO) GOT Overwrite  
Full RELRO) Hook Overwrite  
Hook에 대한 자세한 내용은 접은 글..  

Partial RELRO와 다르게 .got에도 쓰기 권한이 없기 때문에 덮어쓸 수 있는 다른 함수 포인터 탐색..  
-> 라이브러리에 위치한 Hook!  

대표적인 hook으로 malloc hook과 free hook이 존재  
malloc의 경우..  
함수의 시작 부분에서 __malloc_hook이 존재하는지 검사 -> 존재하면 호출  
__malloc_hook은 libc.so에서 쓰기 가능한 영역..  
공격자는 libc 매핑된 주소를 알면.. malloc 호출 -> 실행 흐름 조작  
자세한 건 다음 포스트..  