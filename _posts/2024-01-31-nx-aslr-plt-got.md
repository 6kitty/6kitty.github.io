---
layout: post
title: "NX & ASLR와 PLT & GOT"
categories: [System Hacking]
tags: [NX, ASLR, PLT, GOT, Security]
last_modified_at: 2024-01-31
---

하 저장을 잘.하자  
써둔 거 날리지 말고...

### Mitigation: NX & ASLR  

#### NX  
**No-eXecute:** 실행 권한과 쓰기 권한 분리하는 보호 기법  
코드 영역 외에 실행 권한 제거  
CPU가 NX 지원 -> 컴파일러 옵션으로 바이너리에 NX 적용 가능  

**gdb의 vmmap 기능으로 NX 비교**  

![NX 적용](https://blog.kakaocdn.net/dna/Jk9Cj/btsEaXYQNgD/AAAAAAAAAAAAAAAAAAAAAD0VhydTLb6cvasxGKNbgUa2qfc79HZYyZqSTwKKc7Vy/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xV1zobCL7cW8zx36fqdIJgX16ZQ%3D)  
*NX 적용*  

![NX 미적용](https://blog.kakaocdn.net/dna/bDg14U/btsEbzDfv6A/AAAAAAAAAAAAAAAAAAAAAAcQXbnzYr_uHThgwmxlUzKUh1zCo_PLR_mcMLP5uudv/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=nysaTyB3GrrmAhkKX7Ryvo1uesM%3D)  
*NX 미적용*  

**checksec을 이용한 NX 확인**  
NX unknown 혹은 NX enabled 혹은 NX disabled  
아래 접은 글은 NX의 다양한 명칭  

NX를 인텔은 **XD(eXecute Disable)**, AMD는 NX, 윈도우는 **DEP(Data Execution Prevention)**, ARM에서는 **XN(eXecute Never)**라고 칭하고 있습니다. 명칭만 다를 뿐 모두 비슷한 보호 기법입니다.

**Return to Shelolcode (NX 적용 후)**  

![Return to Shellcode](https://blog.kakaocdn.net/dna/b4MT24/btsEeHG2YmO/AAAAAAAAAAAAAAAAAAAAACUpj9BvWhh2e5WkMdm68EIZ7WoKn3d5SUuCNUculS9S/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=8VuLsB9Zumz3CxdEf%2BKBXhCnOQk%3D)  

canary에서 했던 r2s에 NX 적용 후 동일한 익스플로잇 코드 실행 -> segmentation fault 발생  
실행 권한이 없기 때문에 쉘코드 실행 X -> 종료  

#### ASLR  
**Address Space Layout Randomization:** 바이너리가 실행될 때마다 스택, 힙, 공유 라이브러리 등을 임의의 주소에 할당하는 보호 기법  
커널에서 지원하는 보호 기법  

**ASLR 확인하는 방법**  
```shell
cat /proc/sys/kernel/randomize_va_space 
2
```

| 값 | 설명 | 비고 |
|----|------|------|
| 0  | No ASLR | ASLR 적용 X |
| 1  | Conservative Randomization | 스택, 힙, 라이브러리, vdso 등.. |
| 2  | Conservative Randomization+ brk | 1의 영역 + brk로 할당한 영역 |

**ASLR 특징**  
메모리 주소를 출력하는 코드  
코드에서 보여주는 메모리  

| 변수 | 설명 |
|------|------|
| buf_stack | 스택 영역 |
| buf_heap | 힙 영역 |
| printf | 라이브러리 함수 |
| main | 코드 영역의 함수 |
| libc_base | 라이브러리 매핑 주소 |

1. 실행할 때마다 main을 제외한 다른 영역 주소 변경됨  
-> 실행 전에는 주소를 유추할 수 없음  
2. libc_base 주소 하위 12비트 값, printf 주소 하위 12비트 값은 변경 X  
-> 리눅스는 ASLR이 적용되면 파일을 페이지(page) 단위로 임의 주소에 매핑, 페이지 크기인 12비트 이하 주소 변경 X  
3. libc_base와 printf 주소 차이 항상 동일  
-> ASLR이 적용되면 라이브러리 매핑 -> 매핑된 주소로부터 라이브러리의 다른 심볼들까지의 거리(오프셋) 항상 동일  

### Background: Library - Static Link vs. Dynamic Link  

#### 링크  
**Link:** 컴파일의 마지막 단계, 라이브러리 함수와 caller 함수를 연결해주는 과정  

```c
// Name: hello-world.c
// Compile: gcc -o hello-world hello-world.c

#include <stdio.h>

int main() {
  puts("Hello, world!");
  return 0;
}
```
실습에 쓸 코드(caller)  

```c
// Path: /usr/include/stdio.h

...
/* Write a string, followed by a newline, to stdout.

   This function is a possible cancellation point and therefore not
   marked with __THROW.  */
extern int puts(const char *__s);
...
```
실습에 쓸 라이브러리(callee)  

**리눅스에서 C 코드의 번역 과정**  
전처리 -> 컴파일 -> 어셈블 -> 오브젝트 파일(ELF포맷)  

![Translation Process](https://blog.kakaocdn.net/dna/tDB93/btsEf7SWMfx/AAAAAAAAAAAAAAAAAAAAAPPvtKdIbql3vMlSRVtvs9KyW-kOcAF901UWxHZsl-VN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=EkBQoFP0lxhFilmGVCuOWTG60iY%3D)  

오브젝트 파일 생성..  
-> ELF를 갖추고 있기 때문에 실행 가능한 형식  
-> 하지만 라이브러리 함수들의 정의를 연결해주지 않아서 실행 불가능  

라이브러리에서 가져올 함수는 puts  
**링크되기 전후 비교**  

![링크 전](https://blog.kakaocdn.net/dna/chHWNo/btsEfpsBBeA/AAAAAAAAAAAAAAAAAAAAAKNVb88uTUNpT7JlpNsIm2XdQ_-LPoTHkvd9aE35AX7t/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=5vC%2BPPKnaXLiyBznIbHxy62lrNQ%3D)  
*링크 전*  

![링크 후](https://blog.kakaocdn.net/dna/drDLmi/btsEaNPv5ec/AAAAAAAAAAAAAAAAAAAAAKdMtqYUZW_qCDHXOpt1vM8CoL2uUSyRzumNDGq9YiNv/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=aSDhN%2FgzO%2B7nHlEC07pt4MBdUOI%3D)  
*링크 후*  

#### 라이브러리 링크와 종류  
**동적 링크**  
동적 링크된 바이너리 실행 -> 동적 라이브러리가 메모리에 매핑  
**정적 링크**  
정적 링크된 바이너리 실행 -> 정적 라이브러리에 필요한 모든 함수 포함, 동적과 다르게 참조하는 것이 아니라 본인의 함수 호출처럼 사용 가능  

#### 동적 링크 vs. 정적 링크  

![정적 컴파일, 동적 컴파일 진행](https://blog.kakaocdn.net/dna/bfbQTi/btsEcqF1Sle/AAAAAAAAAAAAAAAAAAAAAD6TC9bZqk9oNA-DQvyfJCRjdFwiNO3O8AvfHxEpDmqr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=RjcA6WyBtQ33%2FpaAOUx7ey6XCHY%3D)  
*정적 컴파일, 동적 컴파일 진행*  

![static](https://blog.kakaocdn.net/dna/YCd46/btsEf2c2o2z/AAAAAAAAAAAAAAAAAAAAALmwSXZb0zKHLchiMuetr5s8N2ziDzz9DhhqrsuZjCk1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=pQWBoKkx5GJ0RXkRVy4J8Z32Bv4%3D)  
*static*  

![dynamic](https://blog.kakaocdn.net/dna/cmFYcv/btsEbBA4BLV/AAAAAAAAAAAAAAAAAAAAAHZ71xkyWgXwqS01YIGILnTNyKoOJ2b-ZEOwlXqgdt8u/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=OHS%2BYdloshUn6i3oq3k0beHt54g%3D)  
*dynamic*  

두 파일 크기를 비교해보면 정적 파일이 852k, 동적 파일이 17k  
동적 파일이 훨씬 가벼움  

두 어셈블리 코드를 비교해보면.. static에서는 puts 직접 호출, dynamic에서는 plt주소로 puts 호출  
*plt: 사용되는 테이블.. 라이브러리에서 함수 주소 찾기 위함*  

#### PLT & GOT  
두 개념 모두 라이브러리에서 동적 링크된 심볼 주소를 찾을 때 사용하는 테이블  

#### runtime resolve  
바이너리 실행 -> ASLR로 인해 라이브러리 임의 주소에 매핑 -> 함수 호출 -> 함수 이름을 이용해서 심볼들 탐색 -> 정의 발견하면 실행 흐름 옮김  

함수의 정의 매번 탐색 -> 비효율적!  
ELF는 GOT라는 테이블 위치시키고 resolve된 함수 주소를 테이블에 저장  
나중에 필요하면 다시 꺼내서 사용  

#### GOT 실습  
```c
// Name: got.c
// Compile: gcc -o got got.c -no-pie

#include <stdio.h>

int main() {
  puts("Resolving address of 'puts'.");
  puts("Get address from GOT");
}
```
**resolve되기 전**  
entry로 진입 후 got와 plt 명령어 입력  

![GOT Entry](https://blog.kakaocdn.net/dna/TI6Q4/btsD6uCVDoY/AAAAAAAAAAAAAAAAAAAAADBZxu2SJP-dRf5ZM2uNRcMgboOziXSfmCb6_nVtrj2f/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=IXgNPbTexgjLr%2FGknuxfdVJaJLc%3D)  

GOT 엔트리인 0x404018에는 puts 주소를 찾기 전이라 .plt 섹션 어딘가의 주소가 적혀있음  
.plt 주소는 0x401020-0x401040  

![plt address](https://blog.kakaocdn.net/dna/BsFoS/btsD6vhuz2q/AAAAAAAAAAAAAAAAAAAAAI35K4V0GLwewYlJ2_BljCTdyz84HqTuBP7005osAxZw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=PYCSnKqaNyVHqL0LxSuuQWpCv3A%3D)  

puts@plt 호출 지점 breakpoint  
내부로 들어가기(si)  

1. puts의 GOT엔트리에 쓰인 값인 0x401030으로 실행 흐름 옮김  
2. 실행 흐름을 따라가면 _dl_runtime_resolve_fxsave가 호출된다는데 내 실습에서는 _dl_runtime_resolve_xsavec가 호출..  
이유는 접은 글  

XSAVE는 FXSAVE를 확장한 명령어  
FXSAVE에서는 FPU, MMX, SSE 레지스터만 담지만,  
XSAVE에서는 여기에 FPU운영 레지스터와 MXCSR레지스터도 담는다  

3. _dl_runtime_resolve_xsavec 함수 실행 -> 여기서 puts 주소 구해지고 GOT 엔트리에 주소 입력  

![GOT Entry After Resolve](https://blog.kakaocdn.net/dna/cm6kG6/btsD6x7v8E2/AAAAAAAAAAAAAAAAAAAAANw8U_Ikrk3p6ki9PRUagSFZXG4CHCuvAnKj2u0dDIwx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xAbBFDpSawJfJJ212t7Flt5ixZY%3D)  

함수 실행 이후 got 0x7ffff7e50420 저장되어 있음  

해당 주소의 경로 열람  

**resolve된 후**  
puts@plt를 got에 한 번 저장한 이후 재호출  

![GOT After Recalling](https://blog.kakaocdn.net/dna/dzEyhZ/btsEa7AbRfn/AAAAAAAAAAAAAAAAAAAAAEFcM3mnwNENH_z5OgNZSoeOXH-HIqAKEJGEgg1MQeZV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=utalkpLMZ7JUXngV8%2F%2FRDQ40GBM%3D)  

### 시스템 해킹에서의 PLT와 GOT  
PLT에서 GOT를 참조하여 실행 흐름을 옮길 때, GOT 값 검증 X  
-> 보안상의 약점..  
puts의 GOT 엔트리에 저장된 값 조작 가능 -> 공격자가 원하는 코드 실행 가능  

**GOT Overwrite:** GOT 엔트리에 값을 오버라이트하여 실행 흐름 변조하는 공격  