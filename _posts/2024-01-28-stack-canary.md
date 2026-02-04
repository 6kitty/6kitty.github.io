---
layout: post
title: "Stack Canary"
categories: [System Hacking]
tags: [Stack, Canary, Buffer Overflow, Security]
last_modified_at: 2024-01-28
---

Stack Canary: 스택 BOF로부터 반환 주소를 보호하는 기법

in 함수 프롤로그..

임의의 값 삽입

스택 버퍼 >>>임의의 값(카나리)<<< 반환 주소

in 함수 에필로그..

이 값의 변조를 확인

### 카나리의 작동 원리

#### 카나리의 정적 분석

```cpp
// Name: canary.c
#include <unistd.h>
int main() {
  char buf[8];
  read(0, buf, 32);
  return 0;
}
```

stack BOF에서 배웠던 것처럼 segmentation fault 발생

하지만 canary를 적용하면 stack BOF가 탐지되어 강제 종료됨

canary.asm과 no_canary.asm 비교..
빨간 부분이 canary.asm에 있던 것... no_canary.asm에서는 없다

이 코드들을 동적 분석..

#### 카나리 동적 분석

내 실습에서는 main+12에서 fs:0x28의 데이터 읽고 rax 저장
*fs: 세그먼트 레지스터 일종..
fs:0x28에는 리눅스가 만든 랜덤값이 저장되어 있음

ni로 넘긴 후 rax값 확인.. 첫 바이트가 null바이트인 8바이트 데이터

이후 그 랜덤값(rax)는 rbp-8에 저장

#### 카나리 검사(함수 에필로그)

rbp-8에 저장한 카나리 -> rcx로 옮김
fs레지스터에 있던 값과 xor 수행 -> 0이 되는가 확인(변조 확인)
je로 main 반환되거나 __stack_chk_fail 호출(강제종료)

16개의 H 입력 준 main+54는 아래..

스택을 보면 -008에서 H로 덮어짐

xor의 값이 0이 아니므로 main+74로 점프가 아니고 다음줄 main+69로 넘어감
__stack_chk_fail

### 카나리 생성 과정

카나리 값은 프로세스 시작하면서 TLS에 전역변수로 저장
각 함수의 프롤로그, 에필로그마다 이 값을 참조
fs는 TLS를 가리킴, 떄문에 fs의 값을 알면 TLS의 주소를 알 수 있음
fs는 특정 시스템 콜 사용으로만 조회, 설정 가능
gdb 기능 사용 불가

#### 1. TLS 주소 파악

arch_prctl(int code, unsigned long addr): fs를 설정할 때 호출되는 시스템 콜
위 상태로 호출하면 fs 값은 addr이 됨

catch: 특정 이벤트 발생 시, 프로세스 중지하는 명령어
arch_prctl에 catchpoint 설정

init_tls() 안에서 catchpoint 도달할 때까지 continue

catchpoint 도달했을 때, rdi값이 0x1002
이때 rsi 0x7ffff7fbf540, TLS를 0x7ffff7fbf540에 저장하고 fs레지스터를 이를 가리킴

fs+28은 곧 0x7ffff7fbf540+0x28이 됨
참조해보면 아직 아무값도 없음

#### 2. 카나리 값 설정

watch: 특정 주소에 저장된 값 변경되면 중단하는 명령어

Hardware watchpoint로 중단될 때까지 c

아까 그대로 fs+0x28 값 참조
아까와 달리 값이 채워짐

main함수에서 rax, qword ptr fs:[0x28] 실행시켜볼거임
rax값을 참조해보면 아까 fs+0x28의 값과 같음

### 카나리 우회

#### 1. 무차별 대입(Brute Force)

x64(8바이트), x86(4바이트) 카나리 생성
각각의 카나리에는 널바이트 포함

#### 2. TLS 접근

조건)
실행중에 TLS 주소를 알 수 있음
임의 주소에 대한 읽기, 쓰기 가능
-> TLS 설정 카나리를 읽거나 조작 가능해짐
이후 스택 BOF 이용해서 익스플로잇 가능

#### 3. 스택 카나리 릭

스택에서 카나리를 읽을 수 있는 취약점
함수의 프롤로그에서 스택 카나리 저장
이를 읽어내는 취약점