---
layout: post
title: "pico- WinAntiDBG0x100"
categories: [System Hacking]
tags: [CTF, Debugging, Reverse Engineering]
last_modified_at: 2025-04-06
---

```cpp
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char Block; // [esp+0h] [ebp-8h]
  char Blocka; // [esp+0h] [ebp-8h]
  WCHAR *lpOutputString; // [esp+4h] [ebp-4h]

  if ((unsigned __int8)sub_401130())
  {
    OutputDebugStringW("\n");
    OutputDebugStringW("\n");
    sub_4011B0();
    if (sub_401200())
    {
      OutputDebugStringW(
        L"### Level 1: Why did the clever programmer become a gardener? Because they discovered their talent for growing a"
        " 'patch' of roses!\n");
      sub_401440(7);
      if (IsDebuggerPresent())
      {
        OutputDebugStringW(L"### Oops! The debugger was detected. Try to bypass this check to get the flag!\n");
      }
      else
      {
        sub_401440(11);
        sub_401530(dword_405404);
        lpOutputString = (WCHAR *)sub_4013B0(dword_405408);
        if (lpOutputString)
        {
          OutputDebugStringW(L"### Good job! Here's your flag:\n");
          OutputDebugStringW(L"### ~~~ ");
          OutputDebugStringW(lpOutputString);
          OutputDebugStringW(L"\n");
          OutputDebugStringW(L"### (Note: The flag could become corrupted if the process state is tampered with in any way.)\n\n");
          j_j_free(lpOutputString);
        }
        else
        {
          OutputDebugStringW(L"### Something went wrong...\n");
        }
      }
    }
    else
    {
      OutputDebugStringW(L"### Error reading the 'config.bin' file... Challenge aborted.\n");
    }
    free(::Block);
  }
  else
  {
    sub_401060((char *)lpMultiByteStr, Block);
    sub_401060("### To start the challenge, you'll need to first launch this program using a debugger!\n", Blocka);
  }
  OutputDebugStringW(L"\n");
  OutputDebugStringW(L"\n");
  return 0;
}
```

ida 디컴파일 전체 코드 

뜯어서 보자 

```cpp
if (IsDebuggerPresent())
{
  OutputDebugStringW(L"### Oops! The debugger was detected. Try to bypass this check to get the flag!\n");
}
else
{
  sub_401440(11);
  sub_401530(dword_405404);
  lpOutputString = (WCHAR *)sub_4013B0(dword_405408);
  if (lpOutputString)
  {
    OutputDebugStringW(L"### Good job! Here's your flag:\n");
    OutputDebugStringW(L"### ~~~ ");
    OutputDebugStringW(lpOutputString);
    OutputDebugStringW(L"\n");
    OutputDebugStringW(L"### (Note: The flag could become corrupted if the process state is tampered with in any way.)\n\n");
    j_j_free(lpOutputString);
  }
}
```

여기 IsDebuggerPresent만 우회하면 될듯?

![스크린샷 2025-04-06 125839.png](https://blog.kakaocdn.net/dna/bc4TMQ/btsM9xqKF1i/AAAAAAAAAAAAAAAAAAAAAA5qPoEzQdRnHSjfZXHjhvLKa5-AkTs2qTNB1wFa9ioO/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=gpwsnXl7tnVi4IgYCU%2BHTiMddWY%3D)

xdbg 열어주고 winantidbg0x100.exe 모듈에서 문자열 참조하면 

![스크린샷 2025-04-06 131932.png](https://blog.kakaocdn.net/dna/bsRNQh/btsNaL8YTvX/AAAAAAAAAAAAAAAAAAAAAF89alRjZPi1545y8q3WWbOiMrX_dBZ1-7u3z20D1eH2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=09TxxqdWtSHIVOD35YALHeiRisQ%3D)

위 문자열이 보임

대충 Good job! 눌러서 해당 위치로 움직이고 스크롤 좀만 올려보면 

![스크린샷 2025-04-06 125941.png](https://blog.kakaocdn.net/dna/bGtDuQ/btsM90MNTfW/AAAAAAAAAAAAAAAAAAAAAIuAnNr5BXkSLc3EPjbj73mEX-w34U7G-VJxDx_64y4G/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7hzLumrUgBpBKSafDjVUax93U1g%3D)

IsDebuggerPresent call 위치 발견 

BP 걸어주자 

![스크린샷 2025-04-06 132036.png](https://blog.kakaocdn.net/dna/blIKGI/btsNba8lNQC/AAAAAAAAAAAAAAAAAAAAAFdfdNoLTcYlQ2wTeGuK-GbHnRiEJmALc3cmGQec1CfV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=TbLcCeUSkCvVVJN5A8Gn2Ufx7KU%3D)

f9로 call까지 와주고 f8 누르면 EAX값이 1일거임 

아까 if문에서도 봤듯이 이걸 0으로 바꿔줘야 함 

eax 값 더블클릭해서 0으로 바꿔줌 

![스크린샷 2025-04-06 132122.png](https://blog.kakaocdn.net/dna/1tSHA/btsNbAeC6fp/AAAAAAAAAAAAAAAAAAAAADOwgE26z8qRlYmmcMZpcOtnwifwzsGCMIimyWgzPFTj/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=jiYeshQVs8SMgSHzwtJXHav6mEc%3D)

F8 계속 누르다보면 위처럼 picoCTF 플래그가 보일거임 

![스크린샷 2025-04-06 132218.png](https://blog.kakaocdn.net/dna/k2QrT/btsNbz7QsLT/AAAAAAAAAAAAAAAAAAAAAGitqZpfhJtMT0_HuyfS6VMK7FpgDsXjMTq9msB0DUAb/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vChf%2F348braUlNf7tcVoySQ1ppY%3D)

stack에서 보면 위와 같음