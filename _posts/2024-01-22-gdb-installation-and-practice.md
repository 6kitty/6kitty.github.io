---
layout: post
title: "gdb설치와 실습"
categories: [System Hacking]
tags: [gdb, pwndbg, 디버깅, 리눅스]
last_modified_at: 2024-01-22
---

Tool: gdb  
gdb: 리눅스의 디버거  
gdb 플러그인중에서 바이너리 분석 용도로 pwngdb 사용  

```shell
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```

위 명령어 한 줄씩 입력  

![pwndbg 설치 화면](https://blog.kakaocdn.net/dna/OGD2D/btsDI7U0HBH/AAAAAAAAAAAAAAAAAAAAAAfRdJH6vEVUT1KF4LqsVVBBjhWvKW7qZ8L-jENobK85/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=4fbr1%2BvVhVw4ix779hZ8Uey2HTE%3D)

wsl에 깔아주려는데 22.04가 자꾸 안되길래 20.04로 했더니 성공적으로 설치되었다  

디버깅 실습 1  

![디버깅 실습 화면](https://blog.kakaocdn.net/dna/cIbjDN/btsDIyE5xvz/AAAAAAAAAAAAAAAAAAAAABhYwPgDasrzLydmZSxN00bWAjPu-3GSVytEPxJ7Uz8F/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=e896ra0RnTPAZiBrE%2B%2FD0ebAI8Y%3D)

![pwndbg에 만들어서 mv로 옮겨](https://blog.kakaocdn.net/dna/bQFPVm/btsDKziGoP8/AAAAAAAAAAAAAAAAAAAAAAIjstt8eSIUrPOXDoH-j-obShmRX--lzaEBrKV-q9pY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vrbCDhztuPhG59aaun%2BXFv7rYkE%3D)

gdb debugee로 디버깅 시작  
1. entry  
리눅스의 실행파일 포맷: ELF  
헤더+여러 섹션으로 구성되어있음  
헤더: 실행에 필요한 여러 정보 담김  
섹션: 컴파일된 기계어 코드, 프로그램 문자열 등 데이터 담김  

ELF 헤더 중에 진입점(Entry Point) 존재  
OS가 ELF 실행 시, 진입점 값부터 프로그램 실행  

![elf의 헤더 정보](https://blog.kakaocdn.net/dna/DHSxI/btsDQ1expYO/AAAAAAAAAAAAAAAAAAAAAFGegJIRk_se9KTY-DijaERmc4LtiEYpjCkgbeCmyEk5/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BlWtUR3PFjEDwLjtfROUK%2BfgSUM%3D)

위 파일의 entry point address는 0x401050  

![rip 값 확인](https://blog.kakaocdn.net/dna/pnTB6/btsDQbuZ3em/AAAAAAAAAAAAAAAAAAAAAH4rPFn9niq-M6BIxz71-3GickrDfuTDCmqHjeletbLg/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=JAfzoTK5xscr4AeeYWbqFuOwfZY%3D)

gdb의 entry 명령어: 진입점부터 분석함  
rip 값을 보면 진입점의 0x401050 주소임을 알 수 있  

2. context  
프로그램 실행 -> 여러 메모리 접근(레지스터 포함)  
pwndbg에서 주요 메모리 상태를 context라 부름  
제공되는 context는 4개  
1. REGISTERS: 레지스터 상태  
2. DISASM: rip부터 디스어셈블된 결과  
3. STACK: rsp부터 스택 값  
4. BACKTRACE: rip에 도달하기까지 중첩,호된 함수들 목록  

3. break&continue/run  

![중단점 설정과 continue](https://blog.kakaocdn.net/dna/sK4v0/btsDQZHN53g/AAAAAAAAAAAAAAAAAAAAAKiyGp9Cqqd4_3tLplpfcwWA8lBi_xOooFB1ZSS5wFJE/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=JqUdHSwQui%2FuBUZfTuphEtJAHuw%3D)

run(r): 프로그램 단순 실행  

4. disassembly  

![disassemble 명령어](https://blog.kakaocdn.net/dna/KAigi/btsDQ4Cldks/AAAAAAAAAAAAAAAAAAAAAKgecKz664nRCXsK7RCODILRxCwwe-5TqQRFFn2Bmim_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CgYa5rpIRn0dJVmnMu8sylK%2BqTQ%3D)

disassemble: gdb의 기본 디셈블 명령어, 함수 이름을 인자로 전달하면 함수 반환까지 디셈블  
u, nearpc, pdisass: pwndbg에서 제공하는 디셈블 명령어  

![디셈블 명령어 예시](https://blog.kakaocdn.net/dna/CxYxH/btsDQekVPGR/AAAAAAAAAAAAAAAAAAAAAEmU2_VGsw-28y5BHjrZNINFDPdhZlMD9Zb2C63t0pom/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=5tIL41loTETnDYkPKJ6F29RKOQs%3D)

5. navigate - ni(next instruction)  
서브루틴으로 들어가지 않는 게 ni  
서브루틴으로 들어가는 게 si(step into)  
내 실습에서는 printf call이 main+61  

![ni 명령어 예시](https://blog.kakaocdn.net/dna/2GEaC/btsDQ7lA21h/AAAAAAAAAAAAAAAAAAAAANdAnCqzZ0xXX-M224sh6Nre2uOc5Dc0Gidbx5xOdYjM/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Uw2grMGACJVcw%2BoXNrvboTNEYPw%3D)

![ni 명령어 결과](https://blog.kakaocdn.net/dna/Bran9/btsDJZbmKrR/AAAAAAAAAAAAAAAAAAAAAH4i2SBtukyxh0Vdx3brQ_-YodRpmeNuE1ZlmGTUlXZD/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=4z%2BPzMccqocSZnAKCJ32sCcBFSM%3D)

ni는 다음 명령인 main+66으로 넘어감  

6. navigate -si(step into)/finish  
위 실습에서 ni가 아닌 si로 이동하면  

![si 명령어 예시](https://blog.kakaocdn.net/dna/bvzFJN/btsDI7ag6HX/AAAAAAAAAAAAAAAAAAAAAHvuQg0r6TVz4kpzlnyaL39JNu7K3t2edrB1UOArc7kh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=gILl45JH7q91%2FKbKSDGbf7p6y24%3D)

printf 안으로 넘어감  

![backtrace 확인](https://blog.kakaocdn.net/dna/bVsryj/btsDLI1eHUb/AAAAAAAAAAAAAAAAAAAAALJQnbbY5-x14mzeEbWOI4wwPCVSo0Y6GE6Rwu3AQil9/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=9VKDAza6Bw84yjpSbN0QhOiPmFI%3D)

backtrace 참조 시, main위에 printf 쌓임  

finish실행하면 si를 내부 함수 끝까지 실행 후 밖으로 나옴  
backtrace를 확인해보면 main만 위로 쌓임  

7. examine  
가상 메모리 임의 주소의 값 출력  
x라는 명령어 제공: 특정 주소에서 원하는 길이만큼 원하는 포맷으로 인코딩하여 출력하는 명령어  
포맷 자세히는 아래 접은글  

<details>
<summary>더보기</summary>
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal), t(binary), f(float), a(address), i(instruction), c(char), s(string) and z(hex, zero padded on the left). Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
</details>

![rsp부터 8바이트 hex형식으로 10번](https://blog.kakaocdn.net/dna/NL1Td/btsDRbauBzt/AAAAAAAAAAAAAAAAAAAAAObnpJFpn98cs4_UnPn5wuxBnF_IxVrFrWNCA70uRZNQ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ZUspySHSPqOgQm4t0vjADpiuOVE%3D)

![rip부터 5줄](https://blog.kakaocdn.net/dna/1uLKp/btsDJDGot83/AAAAAAAAAAAAAAAAAAAAAFsiQeEUwLPm-E7fskgjhqyhi3QyJL8bULGGFq8leh5i/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=jt5mV0C2zVU4Wke%2Fm5sSK720HOU%3D)

![특정 주소 문자열](https://blog.kakaocdn.net/dna/bhpqE7/btsDQ6G0RVj/AAAAAAAAAAAAAAAAAAAAAFh4c9GVVfDdzWnqVucGXb6FIIMocCkBpgXtkQjrcyeZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=12Ty7AK0N9DLte5hDr2HP42lXkA%3D)

8. telescope/vmmap  
telescope: pwndbg가 제공하는 메모리 덤프 기능  
주소의 메모리 값도 보여주고, 메모리가 참조하는 주소 탐색하여 값 열  

![telescope 예시](https://blog.kakaocdn.net/dna/cfDoIr/btsDJaktaPZ/AAAAAAAAAAAAAAAAAAAAAH5T17A0ZDs7HuaVJfQyroouCUHhzIhz0ILAq78AM3Lx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=1VTGlzr%2F5KdrB%2BLDT6FXEDQvABM%3D)

vmmap: 가상 메모리 레이아웃 출력, 매핑 영역이면 파일의 경로도 출력  

![vmmap 예시](https://blog.kakaocdn.net/dna/TsUuZ/btsDQbWdGqP/AAAAAAAAAAAAAAAAAAAAAGdz4_UAuYoe8dSwMsf9OVWM8KtbYbu1r8_DLYbZCzVg/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=IZB4gQCVg4YOFfOuFad9wcCFFOU%3D)

9. gdb/python  
다음 코드의 경우 argv 필요  
이때 파이썬으로 입력값 생성 후 프로그램 입력으로 넘겨주어야 함  

```cpp
// Name: debugee2.c
// Compile: gcc -o debugee2 debugee2.c -no-pie
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
	char name[20];
	if( argc < 2 ) {
		printf("Give me the argv[2]!\n");
		exit(0);
	}
	memset(name, 0, sizeof(name));

	printf("argv[1] %s\n", argv[1]);

	read(0, name, sizeof(name)-1);
	printf("Name: %s\n", name);
	return 0;
}
```

10. gdb/python argv  
r 명령어와 $()를 같이 입력하면 값 전달 가능  

![argv 전달 예시](https://blog.kakaocdn.net/dna/dirLUU/btsDJ02umvm/AAAAAAAAAAAAAAAAAAAAAJxFmXrItw-kTdxkEzpdiy52bfGk9_NAmL_H7qdfZKFi/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Uj2dzegCP3Rm2eGVTOnY%2FdjdOvw%3D)

여기서 $()는 argv  

![read 함수 입력값](https://blog.kakaocdn.net/dna/1Ye0c/btsDKwz7Fb5/AAAAAAAAAAAAAAAAAAAAAJ9QTvBMtG5UgCmkwCLObzAWqZDcIqV0bG6F9-9jmGQJ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=U8hnPmXxYM2spDdR7%2FfF3XHVmCI%3D)

여기서 <<< $()는 read함수가 읽는 입력값  