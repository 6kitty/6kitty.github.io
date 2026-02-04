---
layout: post
title: "SECCON 2016- Anti-Debugging"
categories: [System Hacking]
tags: [Debugging, CTF, Reverse Engineering]
last_modified_at: 2025-04-12
---

```cpp
// positive sp value has been detected, the output may be wrong!
int __cdecl main(int argc, const char **argv, const char **envp)
{
  FILE *v3; // eax
  HANDLE CurrentProcess; // eax
  int v6; // [esp+C4h] [ebp-A8h]
  DWORD TickCount; // [esp+D4h] [ebp-98h]
  LPCSTR lpFileName; // [esp+D8h] [ebp-94h] BYREF
  BOOL pbDebuggerPresent[6]; // [esp+DCh] [ebp-90h] BYREF
  char Str2[16]; // [esp+F8h] [ebp-74h] BYREF
  int v11; // [esp+108h] [ebp-64h]
  char Str1[68]; // [esp+10Ch] [ebp-60h] BYREF
  CPPEH_RECORD ms_exc; // [esp+154h] [ebp-18h]

  memset(Str1, 0, 64);
  v11 = 1;
  printf("Input password >");
  v3 = (FILE *)sub_40223D();
  fgets(Str1, 64, v3);
  strcpy(Str2, "I have a pen.");
  v11 = strncmp(Str1, Str2, 0xD);
  if ( !v11 )
  {
    puts("Your password is correct.");
    if ( IsDebuggerPresent() )
    {
      puts("But detected debugger!");
      exit(1);
    }
    if ( sub_401120() == 112 )
    {
      puts("But detected NtGlobalFlag!");
      exit(1);
    }
    CurrentProcess = GetCurrentProcess();
    CheckRemoteDebuggerPresent(CurrentProcess, pbDebuggerPresent);
    if ( pbDebuggerPresent[0] )
    {
      printf("But detected remotedebug.\n");
      exit(1);
    }
    TickCount = GetTickCount();
    pbDebuggerPresent[3] = 0;
    pbDebuggerPresent[1] = 1000;
    if ( GetTickCount() - TickCount > 0x3E8 )
    {
      printf("But detected debug.\n");
      exit(1);
    }
    lpFileName = "\\\\.\\Global\\ProcmonDebugLogger";
    if ( CreateFileA("\\\\.\\Global\\ProcmonDebugLogger", 0x80000000, 7u, 0, 3u, 0x80u, 0) != (HANDLE)-1 )
    {
      printf("But detect %s.\n", (const char *)&lpFileName);
      exit(1);
    }
    v6 = sub_401130();
    switch ( v6 )
    {
      case 1:
        printf("But detected Ollydbg.\n");
        exit(1);
      case 2:
        printf("But detected ImmunityDebugger.\n");
        exit(1);
      case 3:
        printf("But detected IDA.\n");
        exit(1);
      case 4:
        printf("But detected WireShark.\n");
        exit(1);
    }
    if ( sub_401240() == 1 )
    {
      printf("But detected VMware.\n");
      exit(1);
    }
    pbDebuggerPresent[2] = 1;
    pbDebuggerPresent[5] = 1;
    pbDebuggerPresent[4] = 1 / 0;
    ms_exc.registration.TryLevel = -2;
    printf("But detected Debugged.\n");
    exit(1);
  }
  printf("password is wrong.\n");
  return 0;
}
```

입력값도 다 있음 겁나게 우회하면 되겠군 생각함  

![스크린샷 2025-04-12 150741](https://blog.kakaocdn.net/dna/rD6TW/btsNi3a0PZC/AAAAAAAAAAAAAAAAAAAAAHUflwpD8fQpXekdXZqSP2nTO4KpTIqiVt2uz_H6u7pr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=J39tUz769%2B%2F73Z6%2FAaxUdq3wpWE%3D)  

프롬프트에 I have a pen. 입력하고  

![스크린샷 2025-04-12 150741](https://blog.kakaocdn.net/dna/ODIeL/btsNgg3osZZ/AAAAAAAAAAAAAAAAAAAAAASqTQQeoHsC_26orFyGGjWoV0eypgsFJW0i0qqn9gkL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rsk5EGQDCxhYq%2FAHNxREMWS0dPM%3D)  

겁나게 우회  

근데 어디선가 디버거를 잡는겨  

![스크린샷 2025-04-12 150741](https://blog.kakaocdn.net/dna/vbG0o/btsNi9vBA3r/AAAAAAAAAAAAAAAAAAAAAIUZcArdsYZ91xEtRt3I_6jlVbKLD0BgrykARukj6Ctq/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7MhVsjW7UZGrrjTZ9mBhNb3bHYI%3D)  

이쯤임  

그래서 ida로 다시 봐봄  

![스크린샷 2025-04-12 150741](https://blog.kakaocdn.net/dna/bh1g6m/btsNjbfRq43/AAAAAAAAAAAAAAAAAAAAAEtKXDTTPdxDNqelykKVTWmjh18cVIhNd3QheRqlcYvr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=DfNhT5xcVGUHgnB6nIvOOjKKKOw%3D)  

수많은 디버거 우회를 하면 이쯤에서 도달하는 거 같음 여기가 아마 flag 번역하는 부분  

그래서 그냥 저 aAjJq7 머시기를 문자열 참조로 찾고 그위쯤 주소를 jmp하기로 함  

![스크린샷 2025-04-12 152536](https://blog.kakaocdn.net/dna/b2CqYf/btsNh2psh2l/AAAAAAAAAAAAAAAAAAAAAAREhVYc9Lt_gvNL5WnclJLSQ4NXBNYmFuskK4RtYbY0/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=z%2Fzc2ujnzwYkXj%2F4cEDMFdPGRc0%3D)  

맨처음 디버거 우회 함수가 IsDebuggerPresent머시기  

여기 아래가 원래 jne 머시기였는데  

je bin.40165D로 바꿔줌  

![스크린샷 2025-04-12 152523](https://blog.kakaocdn.net/dna/cA31aS/btsNirYa57G/AAAAAAAAAAAAAAAAAAAAAKptlIisVwotBLusGzCq3rJ8956YMDO1weXjzSaRZpo4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=nX74XsJfu2EaIkdf1qcNgu6lPYw%3D)  

여기서 계속 루프 돌면서 flag를 만들고 있음  

뭔가 루프가 많이 도는 거 같아서  

![스크린샷 2025-04-12 152518](https://blog.kakaocdn.net/dna/me3A1/btsNja85WT3/AAAAAAAAAAAAAAAAAAAAALBLfjs2epZNYiFaYdCDZNNl6bvfrsSfVSu9ooBEA5tc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=KXGq4s3K6G9jCcNQkg1XTJYl7Pg%3D)  

여기에 BP 걸고 f9  

![스크린샷 2025-04-12 152554](https://blog.kakaocdn.net/dna/JnWNN/btsNkwQBxkz/AAAAAAAAAAAAAAAAAAAAAGShvBQ3gWC697trr5veEoIKqGYDLuqLlB9zZoB1oS0w/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=KEBLk3tF9ct6TMTYH2CMmSS4E1A%3D)  

그리고 메세지박스 호출까지 f8해주면 이렇게 뜬다  

사실 디버거 함수 중간중간에 flag decode가 있었으면 귀찮게 또 우회우회할 텐데 다행히 걍 분기해도 플래그가 나오는 문제였다.