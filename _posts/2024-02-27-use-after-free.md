---
layout: post
title: "Use After Free"
categories: [SWING, Writeup]
tags: [Memory Corruption, Use After Free, Dangling Pointer, ptmalloc2]
last_modified_at: 2024-02-27
---

### Background: ptmalloc2
**Memory Allocator:** 메모리를 동적으로 할당하고 해제하는..  
위를 구현하기 위한 알고리즘에 따라 여러 종류 존재..  
리눅스는 ptmalloc2 사용  
메모리 해제 -> 그 메모리 특징 기억 -> 유사 메모리 요청 -> 기억해둔 특징으로 빠르게 반환  
GLibc 2.26부터 도입된 tcache와 관련된 공격 기법  

...일단 실습 환경 구축이 안된다  
문서화 하고 나중에 2회독하면서 다시.................%........다시 해보는 걸로...  

**ptmalloc:** gLibc에 구현됨  
컴퓨터의 전체 메모리 한정적..  

#### ptmalloc의 목적
1. 메모리 낭비 방지  
2. 빠른 메모리 재사용  
3. 메모리 단편화 방지  

**1. 메모리 낭비 방지**  
메모리 할당 요청 -> 해제된 것들중 재사용 탐색  
요청된 크기와 같은 크기가 있다면 재사용  

**2. 빠른 메모리 재사용**  
가상 메모리 공간 매우 넓음!  
특정 메모리 공간을 해제한 이후에 이를 빠르게 재사용하려면  
해제된 메모리 공간 주소 기억해야함 -> tcache 혹은 bin의 연결 리스트에 정보 저장..  

**3. 메모리 단편화 방지**  
내부 단편화: 할당한 메모리 공간 안 실제 데이터가 너무나도 적음 -> 공간이 너무 남음  
외부 단편화: 할당한 메모리 공간 간의 거리(틈)이 너무 큼  
이를 해결하기 위해 정렬(Alignment), 병합(Coalescence), 분할(Split) 사용  
64비트 환경에서 메모리 공간 16바이트 단위..  
16바이트로 정렬..  

메모리를 재사용하기 위해서 병합, 분할도 존재  

#### ptmalloc의 객체
**1. 청크**  
ptmalloc이 할당한 메모리 공간을 말함.. 헤더와 데이터로 구성  
헤더: 필요한 정보  
데이터: 사용자가 입력한 데이터  

헤더중에는 청크의 상태를 나타내므로 in-use와 freed(해제) 헤더 구조가 다름  

| 이름       | 크기 | 의미                                                                 |
|------------|------|----------------------------------------------------------------------|
| prev_size  | 8    | 인접한 직접 청크의 크기<br/>//청크 병합 시 사용                     |
| size       | 8    | 현재 청크의 크기<br/>//64비트 환경에서, 사용중인 청크 헤더 크기 16바이트<br/>//사용자가 요청한 크기 + 16바이트 |
| flags      | 3비트<br/>(나머진 다 바이트) | size의 하위 4비트 의미 없음 (마지막은 0이겟디)<br/>//왜인가 찾아봤더니 8과 16배수 2진수 변경하면 하위 3bit 사용 X<br/>3bit는 A, M, P로 사용<br/><br/>이때 P는 prev-in-use로 병합이 필요한지 판단 |
| fd         | 8    | 연결 리스트에서 다음 청크                                          |
| bk         | 8    | 연결 리스트에서 이전 청크                                          |

**2. bin**  
사용이 끝난 청크들이 저장되는 객체  
ptmalloc 128개의 bin 정의..  
종류는 smallbin(62), largebin(63), unsortedbin(1), 남은 2개는 사용 X  

**1) small bin**  
32바이트 이상 ~ 1024 바이트 미만..  
하나의 small bin에는 같은 크기 청크만 보관..  
index가 1 늘어나면 청크 크기는 16바이트 컺ㅁ  
smallbin[0]가 32바이트라면 smallbin[1]은 48바이트  

small bin은 원형 이중 연결 리스트(circular doubly-linked list)  
먼저 해제된 청크 -> 먼저 재할당... FIFO  
청크를 추가하거나 꺼낼 때 unlink 과정 필요  
smallbin 청크들은 병합 대상.. -> consolidation  

**2) fast bin**  
작은 청크들이 빈번하게 할당, 해제됨..  
32바이트 이상 ~ 176 바이트 이하 크기의 청크  
16바이트 단위로 10개의 fast bin 존재  
단일 연결 리스트.. -> unlink 과정 필요 없음  
LIFO 방법으로 후입선출  
fast bin은 서로 병합되지 않음  

**3) large bin**  
1024 바이트 이상의 크기 청크들 보관  
총 63개  
largebin은 크기가 정해져 있지 않고 범위로 따짐  
ex) largebin[0]은 1024~1088(미만), largebin[32]는 3072~3584(미만)  
재할당 요청 시 크기가 가장 비슷한 청크 할당  
-> 이 과정을 빠르게 하기 위해 largebin 안의 청크를 내림차순으로 정렬  
이중 연결 리스트이므로 unlink  
병합의 대상 //unlink들은 병합의 대상이 되는건가..? 맞다고 함 unlink에서 fd, bk를 교체를 거침  

**4) unsorted bin**  
하나만 존재 -> fastbin에 들어가지 않는 모든 청크 들어감 -> 이후에 small, large 분류  
원형 이중 연결 리스트, 내부 정렬 필요 없음 //걍 약간 아무거나 넣어놓는 맨 아래 서랍 같은 느낌..인데 사진은 맨 위인데 고정적인건가?  

smallbin 크기 요청: fastbin, smallbin 탐색 -> unsortedbin 탐색  
largebin 크기 요청: unsortedbin 탐색 -> largebin 탐색  

**3. arena**  
fastbin, smallbin, largebin 등의 정보를 모두 담고 있는 객체  
멀티 쓰레드 환경에서 레이스 컨디션을 막기 위해 arena에 락  
*레이스 컨디션: 어떤 공유 자원을 여러 쓰레드 혹은 프로세스에서 접근할 때  

최대 64개의 arena 생성 가능  
-> 그렇지만 과도한 멀티 쓰레드 환경에서는 병목 현상 발생..  

한 쓰레드에서 공유 자원에 락 걸면.. 다른 쓰레드는 락이 풀릴 때까지 대기..  
락은 쓰레드를 무제한으로 대기..  
근데 이게 쓰레드가 과다하게 많아지면 병목 현상  
대표적으로 데드락..  

**4. tcache**  
각 쓰레드에 독립적으로 할당되는 캐시 저장소 지칭  
glibc 2.26에서 도입.. 멀티 쓰레드 환경 더욱 최적화된 메모리 관리 메커니즘  

각 쓰레드는 64개의 tcache  
단일 연결 리스트... 하나의 tcache는 같은 크기의 청크만 보관  

tcache에 32~1040바이트 이하의 크기 청크 보관  
이 범위 안 청크는 할당/해제 시 tcache 먼저 조회..  
tcache가 가득 차면 적절한 bin으로 분류  

### Memory Corruption: Use After Free
메모리 참조에 사용한 포인터를 메모리 해제 후에 적절히 초기화하지 않음 -> 해제한 메모리를 초기화하지 않고 다음 청크에 재할당  
그대로 옮겨 적었는데 배우면서 다시 정리해야될듯..  
-> 브라우저 및 커널에서 자주 발견됨.. 익스플로잇 성공률도 높은 편 //ㅎㅎ 잘 배워서 리얼월드에서 취약점 찾고 싶다  

#### Dangling Pointer
Dangling Pointer: 유효하지 않은 메모리 영역 가리키는 포인터  

#### malloc의 동작
malloc 함수의 리턴값: 할당한 메모리 주소  
포인터 선언 후 malloc의 리턴값을 포인터에 저장, 이후 포인터 참조로 메모리 접근  

#### free의 동작
위 malloc과 다르게 청크를 ptmalloc에 반환만 하고 포인터 초기화 X  
때문에 덩그러니 남은 포인터는 Dangling Pointer가 됨  

dangling pointer가 무조건적으로 취약한 것은 아니지만 그렇게 사용될 수 있음  
아래 코드는 Dangling Pointer의 위험성을 보여줌  
```cpp
// Name: dangling_ptr.c
// Compile: gcc -o dangling_ptr dangling_ptr.c
#include <stdio.h>
#include <stdlib.h>

int main() {
  char *ptr = NULL;
  int idx;

  while (1) {
    printf("> ");
    scanf("%d", &idx);
    switch (idx) {
      case 1:
        if (ptr) {
          printf("Already allocated\n");
          break;
        }
        ptr = malloc(256);
        break;
      case 2:
        if (!ptr) {
          printf("Empty\n");
        }
        free(ptr);
        break;
      default:
        break;
    }
  }
}
```
free(ptr)에서 포인터 ptr 초기화 X  

### Use After Free
해제된 메모리에 접근할 수 있을 때 발생하는 취약점  
malloc과 free는 메모리의 데이터를 초기화하지 않는다  
-> 새로 할당한 청크 프로그래머가 초기화하지 않으면, 메모리에 남아있던 데이터 유출 가능  
```cpp
// Name: uaf.c
// Compile: gcc -o uaf uaf.c -no-pie
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct NameTag {
  char team_name[16];
  char name[32];
  void (*func)();
};

struct Secret {
  char secret_name[16];
  char secret_info[32];
  long code;
};

int main() {
  int idx;

  struct NameTag *nametag;
  struct Secret *secret;

  secret = malloc(sizeof(struct Secret));

  strcpy(secret->secret_name, "ADMIN PASSWORD");
  strcpy(secret->secret_info, "P@ssw0rd!@#");
  secret->code = 0x1337;

  free(secret);
  secret = NULL;

  nametag = malloc(sizeof(struct NameTag));

  strcpy(nametag->team_name, "security team");
  memcpy(nametag->name, "S", 1);

  printf("Team Name: %s\n", nametag->team_name);
  printf("Name: %s\n", nametag->name);

  if (nametag->func) {
    printf("Nametag function: %p\n", nametag->func);
    nametag->func();
  }
}
```
위 코드는 UAF 취약점이 있는 코드이다. 구조체 NameTag과 Secret 존재  
**1. Secret은 유출하면 안되는 구조체.**  
main함수를 보면 malloc을 통해 Secret 포인터 할당..  
이후 name, info, code 입력하고 free로 해제..  
**2. nametag도 malloc으로 메모리 할당**  
nametag 요소도 입력하고 출력해줌  
nametag->func이 존재하면 해당 func의 주소를 출력하고 호출  
**3. 위 코드를 실행해보면 다음과 같은 결과가 나옴**  

Name으로 secret_info의 문자열 출력..  
값을 입력한 적 없는 함수 포인터가 0x1337 가리킴  

#### uaf 동적 분석
ptmalloc2는 할당 요청 -> 요청된 크기와 비슷한 청크 bin 혹은 tcache에 있는지 확인  
if 있다면, 재사용  
Nametag와 Secret은 같은 크기 구조체..  
secret 해제 후 nametag 할당 -> 같은 메모리 영역 사용!  
free가 메모리 안 데이터까지 초기화 해주지 않으므로, nametag에 secret 일부가 남아있다  

동적 분석에서 해야할 것  
gdb를 이용하여 secret 해제 직후 secret이 사용하던 메모리 영역 데이터 관찰  

위 부분이 secret free하는 부분..  
Free chunk를 보면, 0x405290이 secret에 해당하는 청크..  
현재 tcache에 들어가 있음(해제되었기 때문에)  

0x405000은 tcache와 관련된 공간..  
tcache_perthread_struct 구조체에 해당.. libc 단에서 힙 영역 초기화할 때 할당하는 청크  

여기까지 하고 그냥 잤다  
이후 실습이 해제된 메모리 영역 출력하는 건데 아래와 같이 청크 주소가 바뀌었다(참고바람)  

0x405290이 해제된 청크  
0x52b0에 뭔가 똥값이 있는데 뭔지 함 보자  

secret_name은 rd, bk 값으로 초기화되었지만,  
secret_info 부분은 그대로 존재.. 정보 유출 주의..  

다음 print문으로 중단점 걸기  
nametag->team_name에 security team 입력됨  
nametag->name에는 secret_info 일부분 존재  
nametag->func위치에 secret->code에 대입했던 값 남아있음  
이값이 0이 아니라서 segmentation fault 발생  