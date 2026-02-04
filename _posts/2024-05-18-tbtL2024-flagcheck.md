---
layout: post
title: "[TBTL2024] Flagcheck"
categories: [System Hacking]
tags: [CTF, Flagcheck]
last_modified_at: 2024-05-18
---

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v3; // ebx
  int seed; // [rsp+4h] [rbp-6Ch]
  int i; // [rsp+8h] [rbp-68h]
  int j; // [rsp+Ch] [rbp-64h]
  char s[72]; // [rsp+10h] [rbp-60h] BYREF
  unsigned __int64 v9; // [rsp+58h] [rbp-18h]

  v9 = __readfsqword(0x28u);
  printf("Let me check your flag: ");
  __isoc99_scanf("%s", s);
  if ( strlen(s) != 63 )
    no();
  seed = 1;
  for ( i = 0; i < strlen(s); ++i )
    seed *= s[i];
  srand(seed);
  for ( j = 0; j < strlen(s); ++j )
  {
    v3 = s[j];
    if ( ((rand() % 256) ^ v3) != target[j] )
      no();
  }
  puts("Correct!");
  return 0;
}
```

s 길이 63  
seed 값 계산 가능..? -> random에서 쓰는 문자열(ascii) seed는 0~256  

v3는 s[j]  
(rand()%256)^v3값이 target[j]의 값과 같아야 함  
이때 rand는 seed가 있으므로 규칙적으로 정해짐  

```cpp
33h, 84h, 3Dh, 3Fh, 2Ah, 93h, 7Bh, 82h, 1Ah, 0ACh, 8Eh //11
0F4h, 0B1h, 0CBh, 8Dh, 21h, 0Eh, 0B7h, 67h, 96h, 2Ch  //10
81h, 0D3h, 0BCh, 29h, 6Ch, 4Bh, 0Dh, 0EDh, 0Fh, 0Dh  //10
0EEh, 56h, 40h, 52h, 0D5h, 5, 6Dh, 90h, 3Eh, 7Ah, 1Bh //11 
69h, 23h, 1Fh, 0B6h, 1Dh, 0BCh, 98h, 0D1h, 0A6h, 83h  //10 
0E9h, 0EBh, 13h, 21h, 3Dh, 0F8h, 2Bh, 79h, 53h, 4Fh   //10 
0A1h  //1
```

이게 target...  

```python
#include <iostream>
#include <random>

using namespace std;

int main() {
    
    unsigned char target[63] = { 0x33, 0x84, 0x3d, 0x3f, 0x2a, 0x93, 0x7b, 0x82, 0x1a, 0xac, 0x8e,
                      0xf4, 0xb1, 0xcb, 0x8d, 0x21, 0x0e, 0xb7, 0x67, 0x96, 0x2c,
                      0x81, 0xd3, 0xbc, 0x29, 0x6c, 0x4b, 0x0d, 0x00, 0xed, 0xfd,
                      0xee, 0x56, 0x40, 0x52, 0xd5, 0x05, 0x6d, 0x90, 0x3e, 0x7a, 0x1b,
                      0x69, 0x23, 0x1f, 0xb6, 0x1d, 0xbc, 0x98, 0xd1, 0xa6, 0x83,
                      0xe9, 0xeb, 0x13, 0x21, 0x3d, 0xf8, 0x2b, 0x79, 0x53, 0x4f, 0xa1 };

    unsigned long long seedst = 0;
    unsigned long long seeden = 256;
    bool found = true;
    unsigned long long seed;
    char s[64] = { 0, };

    for (unsigned long long i = seedst; i <= seeden; ++i) {
        seed = i;
        srand(i);

        for (int j = 0; j < 63; ++j) {
            int randd = rand() % 256;
            s[j] = randd ^ target[j];

            if (s[j] < 32 || s[j] > 126) {
                found = false;
                break;
            }
        }
    }
    if (found) {
        cout << "seed:" << seed << ", string:" << s;
    }
    else
        cout << "NOT FOUND";

    return 0;
}
```

...안된다... 뭔가를 잘못했나 싶어서 라이트업도 참고 해봤는데 라이트업 코드도 실행이 안된다 왜일까...?