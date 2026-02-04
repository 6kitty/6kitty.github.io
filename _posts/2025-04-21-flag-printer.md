---
layout: post
title: "flag-printer"
categories: [System Hacking]
tags: [c++, programming, security]
last_modified_at: 2025-04-21
---

```cpp
void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)
{
  __int64 v3; // rax
  char s[72]; // [rsp+10h] [rbp-50h] BYREF
  unsigned __int64 v5; // [rsp+58h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  sub_13A9(a1, a2, a3);
  sub_140E();
  while ( 1 )
  {
    while ( 1 )
    {
      printf("> ");
      fgets(s, 64, stdin);
      v3 = sub_1437(s);
      if ( (_DWORD)v3 != 2 )
        break;
      sub_1749();
    }
    if ( (int)v3 > 2 )
    {
LABEL_10:
      puts("Invalid Command!");
    }
    else if ( (_DWORD)v3 )
    {
      if ( (_DWORD)v3 != 1 )
        goto LABEL_10;
      sub_1711(HIDWORD(v3));
    }
    else
    {
      sub_1673(HIDWORD(v3));
    }
  }
}
```

```cpp
  int v2; // eax
  unsigned int v4; // [rsp+18h] [rbp-38h]
  int i; // [rsp+1Ch] [rbp-34h]
  const char *s1; // [rsp+20h] [rbp-30h]
  char *v7; // [rsp+28h] [rbp-28h]
  char s2[5]; // [rsp+33h] [rbp-1Dh] BYREF
  unsigned __int64 v9; // [rsp+38h] [rbp-18h]

  v9 = __readfsqword(0x28u);
  v4 = 0;
  a1[strcspn(a1, "\n")] = 0;
  s1 = strtok(a1, " ");
  while ( s1 )
  {
    if ( !strcmp(s1, "print") )
      return (unsigned __int64)v4 << 32;
    if ( !strcmp(s1, "id") )
      return ((unsigned __int64)v4 << 32) | 1;
    if ( !strcmp(s1, "help") )
      return ((unsigned __int64)v4 << 32) | 2;
    v7 = strdup(s1);
    for ( i = 0; v7[i]; ++i )
    {
      v2 = i;
      v7[v2] ^= 0x42u;
    }
    strcpy(s2, "&-17");
    if ( strcmp(v7, s2) )
    {
      free(v7);
      return ((unsigned __int64)v4 << 32) | 0xFFFFFFFF;
    }
    v4 = 1;
    s1 = strtok(0, " ");
    free(v7);
  }
  return ((unsigned __int64)v4 << 32) | 0xFFFFFFFF;
}
```

명령어 입력에 따라 token화함 

일단 whileflag가 2가 아니여야 하니까 근데 2보다 커서도 안되니까 

1이나 0 근데 1이여도 goto LABEL이라고 함 

0이되어야 함 

근데 HIDWORD는 뭘까 

HIDWORD는 주로 64비트 값에서 상위 32비트를 추출하는 데 사용되는 매크로입니다. 이 매크로는 주어진 64비트 값에서 상위 32비트를 반환합니다.

값을 받고 xor했을 때 &-17이면 v4=1이다(strcmp는 같으면 0을 반환하기 때문에) 

strtok으로 띄쓰만큼 토큰을 나누고 while문 돌리기 때문에 그 다음은 print 넣어주면 하위 32비트는 0이 된다. 1이었던 부분은 << 연산자에 의해 32비트 올라가기 때문에 HIDWORD에 포함된다 

![Image](https://blog.kakaocdn.net/dna/so3Bp/btsNugOnOQj/AAAAAAAAAAAAAAAAAAAAAOtyayQ4Ai6cMbfk2APA6Vr9qrgejzw9ZASR7SGg8gJL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=OdLX0LK9IdWlbJurd8woIakmxLg%3D)