---
layout: post
title: "ptrace_block"
categories: [System Hacking]
tags: [CTF, Ptrace, AES, Reverse Engineering]
last_modified_at: 2025-04-20
---

풀면서 rename 해서 함수/변수명이 다를 수 있습니다~

![Image](https://blog.kakaocdn.net/dna/w6NtW/btsNlZf0O1d/AAAAAAAAAAAAAAAAAAAAAJql7QQea4rhcD8OnFkKwFPyw-MqeuyZZ7G5bhdAOFlT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=RF80t618iy7mAO5hz%2FoY0ZAxSN8%3D)

sub_13f1을 봐야겠군 

```python
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  int fd; // [rsp+Ch] [rbp-514h]
  _QWORD input[32]; // [rsp+10h] [rbp-510h] BYREF
  _BYTE buf[1032]; // [rsp+110h] [rbp-410h] BYREF
  unsigned __int64 v7; // [rsp+518h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  memset(input, 0, sizeof(input));
  puts("generate your flag!");
  printf("> ");
  __isoc99_scanf("%255s", input);
  AES((__int64)input, (__int64)buf, 256);
  fd = open("./out.txt", 1);
  write(fd, buf, 0x100u);
  close(fd);
  return 0;
}
```

디컴파일은 위와 같고 걍 어쩌구저쩌구 out.txt에 AES 값을 써줌  
드림핵에서 준 out.txt의 원본 데이터를 구하면 Flag일듯  
sub_13f1 가면 key를 set하고 encrypt하는 함수 존재  
key 만들 때 쓰이는 값은 byte_4010  

근데 ptrace가 안 보이길래 functions에서 ptrace 찾고 xref했더니  

![Image](https://blog.kakaocdn.net/dna/5fKQp/btsNm2vvf7D/AAAAAAAAAAAAAAAAAAAAAFZB3NkOcvcV6S_BT-xxJyMZeJ9gt03b3nSknyR8VnXZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Qw65z9DC2vJLubmPYASD3GUQziM%3D)

이런 부분이 있었음 -> ptrace 값에 따라 key 변형  

해당 코드들  

![Image](https://blog.kakaocdn.net/dna/E3aEd/btsNlMBeVLb/AAAAAAAAAAAAAAAAAAAAABJ1RH8bZiWFFZPO0529jfDFBu60mrulqwuEh-Wcwy-S/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CeCwGOl%2FyMQK6lKgYCU7E0VDr%2FU%3D)

라업에서는 왼쪽만 사용했지만 난 후킹 코드 안 짜고 gdb에서 빼올 거니까 오른쪽처럼 ptrace로 rand말고 ebx,0으로 패치했다  

gdb 돌리고 set key 함수에서 인자값 들여다보면 위와 같이 나온다  
이걸 임의의 값 0x41로 xor해줬으니 다시 0x41로 역연산하고 0부터 255 랜덤으로 브루트 포싱하여  
DH{ 머시기의 printable한 flag를 구하면 된다.  

+) 다른 풀이  
32기분들 풀이 보니까 굳이 패치하지 않고 ida에서 초기 byte_4010 값 빼와서 차피 AES 256가지이니까 0부터 255까지 브루트 포스해도 구해진다고 합니다(저는 파이썬으로 짜다가 오류나서 c로 openssl 코드 짰는데 파이썬으로 짜신 분들도 있네요)  

![Image](https://blog.kakaocdn.net/dna/kADDA/btsNlbs1AAG/AAAAAAAAAAAAAAAAAAAAAGZDHLzkzsHSy408SO-riH3Gxww3A_4mpCKDWC47mbO3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WQ2m1P394zP2udcm0ccL9aU10a8%3D)

이 openssl 라이브러리가 있어야 한다.  

```cpp
#include <openssl/aes.h>
#include <iostream>
#include <string>
using namespace std;

unsigned char initial_key[16] = { 0x00, 0x28, 0xc3, 0x91, 0x34, 0xb0, 0xd3, 0x92, 0xa7, 0xf4, 0x7c, 0xa8, 0x52, 0x42, 0xfb, 0xd5 };
unsigned char key[16] = { 0, };
AES_KEY aes_key[256];
unsigned char iv[16] = { 0, };

unsigned char oridata[64] = {
    0x62, 0xda, 0xb2, 0x6a, 0x74, 0xa3, 0x75, 0xd0,
    0x14, 0x02, 0xad, 0xa3, 0x87, 0x9a, 0x40, 0x31,
    0xd8, 0xa4, 0x85, 0xec, 0x17, 0x73, 0x80, 0xf4,
    0x4c, 0xa4, 0xab, 0x94, 0x41, 0x54, 0x59, 0x99,
    0x00, 0x3a, 0x80, 0x70, 0xe7, 0xea, 0xcd, 0x6c,
    0xe8, 0xbb, 0xf2, 0x80, 0xd5, 0x38, 0x59, 0xe2,
    0x78, 0xe7, 0xbc, 0x0e, 0x69, 0xb3, 0xf6, 0x0d,
    0xe7, 0x49, 0x3b, 0x51, 0x00, 0x88, 0xdc, 0x8b
};

int main() {
    for (int i = 0; i < 256; i++) {
        for (int j = 0; j < 16; j++) {
            initial_key[j] ^= 0x41;
            key[j] = initial_key[j] ^ i;
            iv[j] = 0;
        }
        
        AES_set_encrypt_key(key, 128, aes_key);
        unsigned char dedata[256];
        AES_cbc_encrypt(oridata, dedata, 256, aes_key, iv, 0);

        if (dedata[0] == 'D' && dedata[1] == 'H') {
            cout << dedata;
            break;
        }
    }
    return 0;
}
```

일단 이렇게 짜봄 안됨  
oridata를 잘못 불러오는 거 같아서 file에서 읽어오기 함  

```cpp
#include <stdio.h>
#include <openssl/aes.h>

unsigned char initial_key[16] = { 0x00, 0x28, 0xc3, 0x91, 0x34, 0xb0, 0xd3, 0x92, 0xa7, 0xf4, 0x7c, 0xa8, 0x52, 0x42, 0xfb, 0xd5 };
unsigned char key[16] = { 0, };
unsigned char iv[16] = { 0, };

char* readFile(const char *filename) {
    FILE *file = fopen(filename, "r");
    fseek(file, 0, SEEK_END);
    long fileSize = ftell(file);
    rewind(file);
    char *content = (char*)malloc(fileSize + 1);
    fread(content, 1, fileSize, file);
    content[fileSize] = '\0';
    fclose(file);
    return content;
}

char dedata[256];

int main() {
    AES_KEY aes_key;
    int flag = 0;

    for (int i = 0; i < 16; i++) {
        key[i] = initial_key[i] ^ 0x41;
    }
    char *oridata = readFile("./out.txt");

    for (int i = 0; i < 256; i++) {
        for (int j = 0; j < 16; j++) {
            iv[j] = 0;
        }

        for (int j = 0; j < 16; j++) {
            key[j] = key[j] ^ i;
        }
        AES_set_decrypt_key(key, 128, &aes_key);
        AES_cbc_encrypt(oridata, dedata, 256, &aes_key, iv, AES_DECRYPT);
        if (dedata[0] == 'D' && dedata[1] == 'H') {
            printf("%s", dedata);
            flag = 1;
            break;
        }
    }

    if (flag == 0) {
        printf("NO ANSWER");
    }

    return 0;
}
```

file 읽기 말고 배열로도 꼭 풀고 싶어서 뭐가 틀렸나 다시 한 번 풀어봤다  

![Image](https://blog.kakaocdn.net/dna/p3sRI/btsNsFUpCiu/AAAAAAAAAAAAAAAAAAAAAC3Sam9qxxtV2os6Sm4V8tko58HiI097U5KRd6eAT0M6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=RE5tPT5%2B%2FgmO05fbAyMYKNn%2FZC8%3D)

일단 리눅스 명령어로 읽고  

![Image](https://blog.kakaocdn.net/dna/vgRpI/btsNrypcaPo/AAAAAAAAAAAAAAAAAAAAAFjT0YUDyo8ww7LTVJ3jT-_DRapsVRRSqts3AeYpr-8d/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=GArXgQd8E%2BZC5UlXGKi3yvWbQvM%3D)

이건 지피티 도움  

```cpp
#include <stdio.h>
#include <openssl/aes.h>

unsigned char initial_key[16] = { 0x00, 0x28, 0xc3, 0x91, 0x34, 0xb0, 0xd3, 0x92, 0xa7, 0xf4, 0x7c, 0xa8, 0x52, 0x42, 0xfb, 0xd5 };
unsigned char key[16] = { 0, };

unsigned char iv[16] = { 0, };

unsigned char oridata[] = {
    0xE1, 0x6D, 0xFC, 0xEC, 0xB9, 0x34, 0x22, 0xBD, 0x80, 0xFF, 0x05, 0x45, 0x56, 0xE4, 0x0B, 0x7A,
    0x05, 0xE4, 0x8D, 0x3F, 0xD3, 0x03, 0xFB, 0x01, 0xF0, 0x00, 0xC5, 0x70, 0xF0, 0x52, 0x8F, 0x68,
    0x3A, 0xDD, 0x5A, 0x23, 0xB1, 0x6F, 0x19, 0x3A, 0x57, 0x36, 0xE6, 0xD7, 0xFB, 0xB1, 0xFF, 0x8B,
    0x5F, 0x4A, 0x7B, 0x8D, 0x6D, 0x0A, 0x98, 0x21, 0x8B, 0x04, 0x00, 0xC4, 0x3D, 0xB7, 0x19, 0x82,
    0x58, 0xE1, 0xB1, 0x23, 0xB4, 0xB6, 0x6B, 0xD2, 0x5D, 0x0A, 0xBF, 0x52, 0x0F, 0xB3, 0x08, 0xE6,
    0x35, 0x8B, 0x25, 0xB0, 0x15, 0xA4, 0x87, 0x2C, 0x32, 0x11, 0x33, 0x72, 0xBF, 0x5D, 0xBA, 0xCC,
    0x94, 0xF6, 0xDB, 0x09, 0xD0, 0xBC, 0x31, 0x7F, 0xD1, 0x05, 0x89, 0x7C, 0x06, 0xF5, 0xBF, 0x58,
    0xB9, 0x2D, 0x0E, 0x98, 0x9C, 0x23, 0xC5, 0x1F, 0xD5, 0x8C, 0xB4, 0x73, 0x8E, 0x28, 0x75, 0xD3,
    0x40, 0xDC, 0x68, 0xF6, 0xF3, 0x2E, 0x1E, 0x0F, 0xBC, 0x8B, 0xCF, 0x6A, 0xD9, 0x46, 0x7D, 0x1F,
    0x7D, 0x8E, 0x08, 0x66, 0xB6, 0xCF, 0x43, 0x5A, 0x4B, 0xFC, 0xE0, 0xBA, 0xCB, 0xB5, 0xF9, 0xA9,
    0xF3, 0x96, 0x6C, 0xB3, 0xC6, 0x2C, 0x87, 0xCF, 0xA5, 0xCB, 0x8D, 0xAE, 0xD0, 0x67, 0x5C, 0xF7,
    0xBE, 0x13, 0x2D, 0x67, 0x1B, 0x2D, 0xF8, 0xC6, 0xB1, 0x41, 0xF9, 0x20, 0xD0, 0xDB, 0xF3, 0x16,
    0x60, 0x50, 0x6B, 0x2F, 0x1A, 0x31, 0x9E, 0x19, 0x8F, 0x06, 0x95, 0xC3, 0xBC, 0x17, 0x59, 0xC5,
    0xF7, 0x14, 0xF7, 0x8C, 0x7D, 0x8E, 0x26, 0x53, 0xCA, 0x67, 0xFD, 0x99, 0x19, 0xBF, 0x84, 0x35,
    0xE0, 0x68, 0xD4, 0xA3, 0x2D, 0x17, 0x09, 0x94, 0x87, 0xF4, 0xB7, 0xF7, 0xB3, 0xBC, 0xC4, 0xB3,
    0xE1, 0xE5, 0x74, 0x27, 0xB0, 0x29, 0x75, 0x83, 0xAA, 0x3A, 0x7F, 0xD1, 0xDB, 0xB0, 0xB8, 0x5D
};

char dedata[256];

int main() {
    AES_KEY aes_key;
    int flag = 0;

    for (int i = 0; i < 16; i++) {
        key[i] = initial_key[i] ^ 0x41;
    }

    for (int i = 0; i < 256; i++) {
        for (int j = 0; j < 16; j++) {
            iv[j] = 0;
        }

        for (int j = 0; j < 16; j++) {
            key[j] = key[j] ^ i;
        }
        AES_set_decrypt_key(key, 128, &aes_key);
        AES_cbc_encrypt(oridata, dedata, 256, &aes_key, iv, AES_DECRYPT);
        if (dedata[0] == 'D' && dedata[1] == 'H') {
            printf("%s", dedata);
            flag = 1;
            break;
        }
    }

    if (flag == 0) {
        printf("NO ANSWER");
    }

    return 0;
}
```

lssl lcrypto 넣어서 컴파일 하면  