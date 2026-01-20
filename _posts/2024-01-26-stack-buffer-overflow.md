---
layout: post
title: "Stack Buffer Overflow"
categories: [SWING, Writeup, Self-study]
tags: [Buffer Overflow, Memory Corruption, Exploit]
last_modified_at: 2024-01-26
---

Memory Corruption: Stack Buffer Overflow

버퍼 오버플로우

버퍼: 데이터의 임시 저장소..
처리속도가 다른 장치 사이에서 데이터를 안전하게 전달하기 위해서 임시 저장소 필요
이 역할을 버퍼가 해줌
지금은 의미가 확대되어 데이터가 저장되는 모든 단위를 버퍼라고 함

스택에 있는 지역 변수 -> 스택 버퍼
힙에 할당된 메모리 영역 -> 힙 버퍼

버퍼 오버플로우: 버퍼가 넘친다..
원래 배당된 공간보다 데이터가 넘치는 현상

버퍼 오버플로우 취약점

**1. 중요 데이터 변조**
```cpp
// Name: sbof_auth.c
// Compile: gcc -o sbof_auth sbof_auth.c -fno-stack-protector
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_auth(char *password) {
    int auth = 0;
    char temp[16];
    
    strncpy(temp, password, strlen(password));
    
    if(!strcmp(temp, "SECRET_PASSWORD"))
        auth = 1;
    
    return auth;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: ./sbof_auth ADMIN_PASSWORD\n");
        exit(-1);
    }
    
    if (check_auth(argv[1]))
        printf("Hello Admin!\n");
    else
        printf("Access Denied!\n");
}
```
check_auth에 취약점 존재
strncpy로 temp 버퍼 복사할 때 strlen(temp)가 아닌 strlen(password)만큼 복사
strlen(temp)<strlen(password)라면 버퍼 오버플로우!
temp는 16바이트, 16바이트 넘는 문자열 전달
**모듈 실습..**
스택은 지역 변수를 쌓기 때문에 temp아래에 auth위치..
char 16바이트만큼 아무렇게 넣고 int가 4바이트 이므로 0001입력

**2. 데이터 유출**
표준 문자열 출력 함수 -> null 바이트를 문자열 끝으로 인식
버퍼 오버플로우 발생 -> 다른 버퍼 사이에 있는 널바이트 모두 제거 -> 다른 버퍼도 출력

**3. 실행 흐름 조작**
```cpp
// Name: sbof_ret_overwrite.c
// Compile: gcc -o sbof_ret_overwrite sbof_ret_overwrite.c -fno-stack-protector
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    char buf[8];
    printf("Overwrite return address with 0x4141414141414141: ");
    gets(buf);
    return 0;
}
```
친절하게 안 알려주고 나보고 하라고 함
buf 8바이트 gets로 바로 받음
gets(buf) 호출과 반환을 생각해보좌
스택에 반환주소 쌓고 buf 받겟지?
그러면 buf 8바이트만큼 채우고 0x4141414141414141
해보자
안되는디? 8로 16바이트 다 채워볼
41이 16진수니까 아스키 테이블 참고해서 A로 채워봄 

됐음 buf랑 sfp를 8로 채우고(16바이트) ret가 반환주소니까 A로 채우기 

[함께실습] Exploit Tech: Return Address Overwrite
```cpp
// Name: rao.c
// Compile: gcc -o rao rao.c -fno-stack-protector -no-pie
#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

void get_shell() {
  char *cmd = "/bin/sh";
  char *args[] = {cmd, NULL};
  execve(cmd, args, NULL);
}

int main() {
  char buf[0x28];
  init();
  printf("Input: ");
  scanf("%s", buf);
  return 0;
}
```
28바이트 buf scanf에 %s 포맷 스트링은 받을 길이를 정해주지 않아서 bof에 취약함 
이거 이용해서 buf에 a*0x28+sfp채우고+ret 넣기 

segmentation fault

segmentation fault 에러: 잘못된 메모리 주소 접근
전 코어 파일이 안 생기는데요

Ubuntu 20.04 버전 이상은 기본적으로 /var/lib/apport/coredump 디렉토리에 코어 파일을 생성합니다.

코어 파일 크기 제한 풀어주기 

gdb -c core로 코어 파일 분석

최상단 rsp가 ret일 텐데 
A로 가득 채워짐(겁나 많이 입력했네;;) 
이 ret는 실행가능한 주소가 아니라서 세그먼테이션 폴트 발생 

**익스플로잇**
1. 스택 프레임 구조 파악 
2. get_shell() 주소 파악 / 페이로드 구성 
3. 엔디언 적용 
4. 익스플로잇 실행

아래부터는 익스플로잇 과정 상세히 설명...

익스플로잇 실습 
**1. 스택 프레임 구조 파악**

main의 어셈블리 코드 nearpc로 열람 
scanf에 인자 전달하는 부분이 중요
한데 내 실습에선 안 보여서 아래처럼 해주었음 

내 실습에서는 0x402014가 "%s" 
의사 코드 표현해보자~ 
scanf("%s",[rbp-0x30]);
버퍼는 rbp-0x30에 존재한다는 거겠죠...
rbp에는 sfp 저장 
rbp+0x8에 반환 주소 저장 
sfp가 8바이트? 왜?
아 x86-64면 8바이트
x86면 4바이트라네염
헤헷콩
그럼 rbp+0x8은 반환 주소가 맞음 
그러면 스택 다시 정리

| buf | 0x30 |
| --- | --- |
| sfp | 0x8 |
| ret | get_shell() |

근데 0x28인데 왜 0x30이나 차지하지? 의 답변 접은글
나 같은 사람이 드림핵 질문 남겨놨음.. [https://dreamhack.io/forum/qna/3647/](https://dreamhack.io/forum/qna/3647/)
컴파일러는 효율적인 방법으로 버퍼 확보함.. 꼭 문자열의 크기만큼 버퍼를 할당해줄 필요는 없음 
만약 그렇게 해주려면 setvbuf로 미리 선언해줘야 함 
관련 내용은 stack alignment이라고 함(찾아보진 않았다...) 

암튼 이러한 이유로 코드보다 실제로 확보하는 버퍼의 크기를 확인하는 게 더 도움될 거 같다는 교훈..!

buf부터 sfp까지 0x38을 더미값으로 채움 
근데 그 반환 주소를 shell 취득할 수 있는 함수 주소로 가야함 
본 코드엔 get_shell()로 떠맥여줌 

**2. get_shell() 주소 확인/ 페이로드 작성**
get_shell() 주소 0x4011dd 

**3. 엔디언 적용**
x86-64는 리틀 엔디언이므로 
\xdd\x11\x40\x00\x00\x00\x00\x00

**4. 익스플로잇**
(python3 -c "import sys;sys.stdout.buffer.write(b'A'*0x30 + b'B'*0x8 + b'\xdd\x11\x40\x00\x00\x00\x00\x00')";cat) | ./rao
안되는데 그냥 pwntools로 워게임 flag 구하겠음 

왜 get_shell() 주소가 다르지 했는데 당연히 내가 컴파일 햇으니까............................. 바본가?