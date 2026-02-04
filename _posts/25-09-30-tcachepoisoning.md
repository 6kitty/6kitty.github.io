---
layout: post
title: "[DH] Exploit Tech: Tcache Poisoning"
categories: [System Hacking]
tags:
  - dfb
  - heap
  - pwn
last_modified_at: 2025-09-30
---

오늘은 어제 공부한 DFB 취약점으로 익스플로잇 공부를 해보자~ Tcache Poisoning을 간단하게만 언급하자면 중복으로 연결된 청크 재할당 -> 그러면 이제 그 청크는 해제된 청크이자 할당된 청크인 점을 이용한다. 이 중첩 상태를 이용해서 공격자가 임의 주소에 청크를 할당하고 -> 해당 청크를 이용하여 임의 주소의 데이터를 읽쓰가 가능하다. 

# Dockerfile 

```dockerfile
FROM ubuntu:18.04

ENV PATH="${PATH}:/usr/local/lib/python3.6/dist-packages/bin"
ENV LC_CTYPE=C.UTF-8

RUN apt update
RUN apt install -y \
    gcc \
    git \
    python3 \
    python3-pip \
    ruby \
    sudo \
    tmux \
    vim \
    wget

# install pwndbg
WORKDIR /root
RUN git clone https://github.com/pwndbg/pwndbg
WORKDIR /root/pwndbg
RUN git checkout 2023.03.19
RUN ./setup.sh

# install pwntools
RUN pip3 install --upgrade pip
RUN pip3 install pwntools

# install one_gadget command
RUN gem install one_gadget

WORKDIR /root
```

이거 빌드하고 런 해주자. 드림핵에서 준 bash는 작동을 안 해서 sudo로 넣었다.

```bash
sudo docker build . -t ubuntu1804
sudo docker run -d -t --privileged --name=my_container ubuntu1804
sudo docker exec -u root -it my_container /bin/bash
```

이러면 사전 설정이 끝났다.
...이제 공부 시작 

# Tcache Poisoning 
tcache를 조작하여 임의 주소에 청크를 할당시키는 공격이 이 tcache poisoning이다. 중첩 상태인 청크에 값을 작성하면 그 청크가 해제된 청크에서는 Fd, bk로 인식하기 때문에 ptmalloc2의 free list에 주소 추가가 가능하다. 

Tcache Poisoning으로 할당한 청크에 대해 
1. 출력 
2. 조작 가능하다면 -> 임의 주소 읽기나 쓰기 

```c
// Name: tcache_poison.c
// Compile: gcc -o tcache_poison tcache_poison.c -no-pie -Wl,-z,relro,-z,now

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
  void *chunk = NULL;
  unsigned int size;
  int idx;

  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);

  while (1) {
    printf("1. Allocate\n");
    printf("2. Free\n");
    printf("3. Print\n");
    printf("4. Edit\n");
    scanf("%d", &idx);

    switch (idx) {
      case 1:
        printf("Size: ");
        scanf("%d", &size);
        chunk = malloc(size);
        printf("Content: ");
        read(0, chunk, size - 1);
        break;
      case 2:
        free(chunk);
        break;
      case 3:
        printf("Content: %s", chunk);
        break;
      case 4:
        printf("Edit chunk: ");
        read(0, chunk, size - 1);
        break;
      default:
        break;
    }
  }
  
  return 0;
}
```

1에서 청크 할당하고 2에서 지우기가 가능하다. 근데 이 2번에 아무런 필터링이 없기 때문에 DFB가 생긴다. 
+) 후에 드림핵 풀이에서는 필터링이 아니라 포인터를 정리하는 코드가 없어서라고 한다. 

1. tcache poisoning 
: 청크 할당 -> key 조작(4번 이용하는듯?) -> 해제 

2. libc leak 
: setvbuf로 stdin, stdout을 전달함 -> 이 포인터 변수는 Libc 내부에서 IO_2_1_stdin, IO_2_1_stdout을 가리킨다. 이중 한 변수 값을 읽으면 -> Libc 주소를 계산할 수 있다. 

+) Libc 추가 공부가 필요할듯 

해당 포인터는 전역 변수라서 bss에 위치 -> PIE 적용 X 

3. hook overwrite to get shell 

libc 주소 -> 원 가젯 주소 + __free_hook 주소 
__free_hook 주소에 청크 할당 -> 그 청크에 원 가젯 주소 입력 -> free 호출 시 쉘 획득 

배운 거 그대로 정리해보자면 
변수 값 읽어서 libc 주소 -> 원 가젯이랑 Free 얻기 -> __free_hook 주소에 청크 할당(1번) -> 원 가젯으로 Key 작성(4번) -> 해제(2번)

# 보호 기법 

# 익스플로잇 설계 
## Tcache 연결 리스트 조작 
## 라이브러리 릭 
## 훅 덮어쓰기 