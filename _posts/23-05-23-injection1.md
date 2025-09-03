---
layout: post
title: "DLL injection"
categories:
  - Pwnable
tags:
  - windows
  - injection
last_modified_at: 2023-05-23
---

새내기 때 작성한 글 백업했다. 

---

# DLL (Dynamic Link Library)

주로 쓰고 기초적인 함수들을 미리 만들어서 모아둔 것을 라이브러리라고 한다. DLL은 Dynamic Link Library를 뜻하는데, 해당 라이브러리를 사용할 때만 파일을 다운하여 기능을 호출한다. 정적 링크는 함수를 복사해서 붙여넣기라고 생각하고 동적 링크는 잠깐 위치를 옮겨서 호출받았다가 원함수로 돌아오는 거라고 이해했다. DLL은 메모리에 로드되면 DllMain을 실행한다.

DllMain()은 진입점 함수이다. 진입점 함수란 프로그램이 처음 시작될 때 호출되는 함수이다. DllMain 함수는 세 개의 인자를 가진다. hModule, fdwReason, IpReserved이다. hModule은 DLL 파일 이미지가 매핑된 가상 메모리 주소값이다.

DLL 인젝션이란 말 그대로 특정 프로세스에 DLL을 삽입하는 것이다. DLL을 프로세스에 삽입하면 DllMain이 실행되면서 이때 목적에 따라 코드를 넣어 프로그램이 진행될 수 있다.

# DLL 인젝션 활용 예시

DLL 인젝션은 다음과 같은 경우에서 활용할 수 있다.

기능 개선 및 버그 패치: 프로그램의 소스 코드가 없는 등 수정할 수 없는 경우에 DLL 인젝션을 이용해서 새로운 기능을 추가하거나 코드, 데이터를 수정할 수 있다.

메시지 후킹: 등록된 후킹 DLL을 OS에서 직접 인젝션시켜준다.(메시지 후킹에 관해서는 아래서 자세히 서술했다.)

API 후킹: API 후킹이란 Win32 API가 호출될 때 중간에서 이를 가로채서 제어권을 얻어내는 것이다. Win32 API는 Windows에서 제공하는 API인데 원래 Windows OS는 프로그램이 시스템 자원에 접근할 수 없다. 하지만 시스템 접근이 필요할 때 이 API를 이용해서 시스템 커널에 접근을 요청한다. 후킹 함수를 DLL 형태로 만들고 인젝션하여 활용할 수 있다.

기타 응용 프로그램: PC 사용자를 감시/관리 하는 프로그램 목적으로도 쓰인다. 실행 차단, 유해사이트 접속 차단, 모니터링 프로그램 등이 있다.

악성코드: 정상적인 프로세스에 들어가서 백도어 포트를 열고 외부에서 접속하거나, 키로깅 기능 등으로 개인 정보를 훔치는 등 사용하고 있다.

# DLL 인젝션 - 원격 스레드 생성 이용

다음은 블로그에서 발췌한 소스코드이다. 먼저 주석까지 그대로 작성했다. 인젝션 과정에 대해서는 코드 아래서 설명했고 그 외 세부적으로 이해가 안 되는 함수 등은 용어 정리 또는 소스 코드 안에 ///를 기입하고 작성하였다.

Myhack.cpp

```C
// Myhack.cpp
#include <Windows.h>
#include <tchar.h>
#include <stdio.h>
BOOL Go_Injection(DWORD dwPID, LPCSTR DllPath) {
    //프로세스 핸들
    HANDLE hProcess = NULL;
    //스레드 핸들
    HANDLE hThread = NULL;
    //모듈 핸들
    HMODULE hMod = NULL;
    //DLL 경로를 기록한 메모리 주소를 넣을 포인터변수
    LPVOID pRemoteBuf = NULL;
    //스레드 시작 루틴 함수주소를 저장할 변수
    LPTHREAD_START_ROUTINE pThreadProc;
    //인젝션 할 프로세스 제어권 얻기
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID);
    printf(" [+] 프로세스의 핸들(hProcess) : %d\n", hProcess);
    //DLL경로의 길이 얻기
    DWORD dwBufSize = (DWORD)(strlen(DllPath) + 1);
    printf(" [+] DLL의 길이 : %d byte\n", dwBufSize);
    //인젝션 할 DLL 경로를 해당 프로세스에 기록
    pRemoteBuf = VirtualAllocEx(hProcess, NULL, dwBufSize, MEM_COMMIT, PAGE_READONLY);
    WriteProcessMemory(hProcess, pRemoteBuf, (LPVOID)DllPath, dwBufSize, NULL);
    printf(" [+] 인젝션할 프로세스 내 할당받은 메모리주소 : 0x%p\n", pRemoteBuf);
    printf(" [+] 할당받은 주소에 작성할 DLL의 경로 : %s\n", DllPath);
    //write한 DLL을 프로세스에서 로드하기 위한 작업
    hMod = GetModuleHandleA("kernel32.dll");
    pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hMod, "LoadLibraryA");
    printf(" [+] 스레드의 시작주소(LoadLibrary) : 0x%p\n", pThreadProc);
    //write한 DLL을 인젝션할 프로세스에 스레드 생성 후 스레드의 시작주소로
    //LoadLibraryA를 지정
    hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, pRemoteBuf, 0, NULL);
    printf(" [+] 인젝션한 프로세스에서 실행한 스레드 식별자 : %d\n", hThread);
    //스레드가 실행될 때까지 무한정 대기
    WaitForSingleObject(hThread, INFINITE);
    CloseHandle(hThread);
    CloseHandle(hProcess);
    return 1;
}

int main(int argc, CHAR *argv[]) {
    if (argc != 3) {
        printf(" [-] Usage : %s [Process_Name] [DLL_Path] \n", argv[0]);
        //argv[0]=인젝션시킬프로그램, argv[1]=인젝션할 프로세스 이름, argv[2]=dll경로
        return 0;
    }
    //PID=-1
    DWORD dwPID = 0xFFFFFFFF;
    HWND hWnd = FindWindowA(NULL, argv[1]);
    printf(" [+] 프로세스의 창 번호(hWnd) : %d\n", dwPID);
    //Go_Injection 함수의 인자로 PID와 DLL의 경로를 넘겨줌
    //Go_Injection함수 정상동작 시 TRUE 반환
    BOOL flag = Go_Injection(dwPID, argv[2]);
    return 0;
}
```

Injection.dll

```C
// Injection.dll (daum.net 창이 열리는 dll파일)
#include "stdafx.h"
#include <Windows.h>
#include <stdio.h>
#include <urlmon.h>
#pragma comment(lib, "urlmon.lib")
#define DEF_DAUM_ADDR "https://daum.net/index.html"
#define DEF_SAVE_PATH "C:\\Users\\0_0\\Desktop\\index.html"

DWORD WINAPI ThreadProc(LPVOID lParam) {
    URLDownloadToFileA(NULL, DEF_DAUM_ADDR, DEF_SAVE_PATH, 0, NULL);
    MessageBoxA(NULL, " [+] Success", "caption", MB_OK);
    return 0;
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    HANDLE hThread = NULL;
    switch (fdwReason) {
        case DLL_PROCESS_ATTACH:
            hThread = CreateThread(NULL, 0, ThreadProc, NULL, 0, NULL);
            //스레드를 생성하고 스레드의 시작주소를 THreadProc으로 지정 후 실행
            CloseHandle(hThread);
            break;
    }
    return TRUE;
}
```

# 인젝션 과정 설명

## OpenProcess를 이용하여 대상 프로세스의 핸들 구하기:
인젝션을 하기 위해 핸들을 얻어야 한다.
hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID);
OpenProcess는 PID를 통해 핸들을 얻어올 때 사용한다. 위 코드를 해석하자면 PROCESS_ALL_ACCESS(접근권한)의 myhack.dll(dwPID) 핸들을 구한다. hProcess를 이용하여 제어할 수 있다.

## VirtualAllocEx로 대상 프로세스의 메모리 공간 확보하기:
DLL 경로를 작성하기 위해 버퍼*를 할당해줘야 한다. 이때 절대경로의 문자열 길이만큼 할당해줘야 한다.
pRemoteBuf = VirtualAllocEx(hProcess, NULL, dwBufSize, MEM_COMMIT, PAGE_READONLY);
dwBufSIze는 경로 문자열 길이(NULL 포함)이다. VirtualAllocEx는 가상 주소 공간 내에서 메모리 상태를 변경, 할당, 해제하는 함수이다. VirtualAllocEx()의 리턴값은 할당된 버퍼의 주소이다.

## WriteProcessMemory로 대상 프로세스의 메모리에 Injection 할 DLL의 경로 입력:
이후 다음을 이용하여 절대경로를 작성해준다.
WriteProcessMemory(hProcess, pRemoteBuf, (LPVOID)DllPath, dwBufSize, NULL);
위 코드를 그대로 해석하면 hProcess 권한을 가지고 pRemoteBuf에 DllPath(DLL 경로 문자열)을 써준다는 뜻이다.

## LoadLibrary API 주소 구하기 (GetModuleHandle, GetProcAddress):
LoadLibaryW() API를 실행시켜 myhack.dll을 호출해야 하는데 LoadLibaryA는 kernel32.dll 안에 존재한다. 때문에 kernel32.dll에서 LoadlibraryA의 api 주소를 얻어낸다.
hMod = GetModuleHandleA("kernel32.dll");
pThreadProc = (LPTHREAD_START_ROUTINE)GetProcAddress(hMod, "LoadLibraryA");

pThreadProc = 프로세스 메모리 내의 LoadLibraryW() 주소

pRemoteBuf = 프로세스 메모리 내의 "C:\\myhack.dll" 주소

## CreateRemoteThread로 대상 프로세스의 Injection된 데이터 실행하기:
LoadLibrary()를 실행하면 DLL이 인젝션할 프로세스 메모리 안으로 삽입된다. 삽입되면 DllMain()이 호출되고 이는 공격자가 원하는 행위를 프로세스 내에서 실행할 수 있다는 뜻이다.
hThread = CreateRemoteThread(hProcess, NULL, 0, pThreadProc, pRemoteBuf, 0, NULL);

pThreadProc = 프로세스 메모리 내의 LibraryA() 주소

pRemoteBuf = 프로세스 메모리 내의 삽입할 DLL 파일 경로 문자열 주소

```
CreateRemoteThread()
HANDLE WINAPI CreateRemoteThread(
__in HANDLE hProcess, //프로세스 핸들
__in LPSECURITY_ATTRIBUTES lpThreadAttributes,
__in SIZE_T dwStackSize,
__in LPTHREAD_START_ROUTINE lpStartAddress, //스레드 함수 시작
__in LPVOID lpParameter, //스레드 파라미터 주소
__in DWORD dwCreationFlags,
__out LPDWORD lpThreadId //)
```

CreateRemoteThread 함수는 가상 주소 공간에 실행되는 스레드를 생성한다. 프로세스 핸들을 가져야 한다. lpThreadAttributes는 하위 프로세스가 반환된 핸들을 상속할 수 있는지에 대한 여부를 나타내는 포인터이다. 본래 소스코드에서 NULL로 입력해줬으므로 스레드는 핸들을 상속할 수 없다. lpThreadId는 스레드 식별자를 수신하는 변수에 대한 포인터이다. 본 코드에서 NULL로 입력해줬으므로 스레드 식별자는 반환되지 않는다.

# DLL injection – 레지스트리 이용

Windows 운영체제에서 제공하는 Applnit_DLLs와 LoadApplnit_DLLs라는 레지스트리* 항목이 있다.
Applnit_DLLs 항목에 인젝션을 원하는 DLL 경로 문자열을 쓰고 LoadApplnit_DLLs 항목의 값을 1로 변경한 후 재부팅하면, 모든 프로세스에 해당 DLL을 삽입한다.

Myhack2.cpp

```C
#include "windows.h"
#include "tchar.h"
#define DEF_CMD L"c:\\Program files\\Internet Explorer\\iexplore.exe"
#define DEF_ADDR L"http://www.naver.com"
#define DEF_DST_PROC L"notepad.exe"

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    TCHAR szCmd[MAX_PATH] = {0,};
    TCHAR szPath[MAX_PATH] = {0,};
    TCHAR *p = NULL;
    STARTUPINFO si = {0,};
    PROCESS_INFORMATION pi = {0,};
    si.cb = sizeof(STARTUPINFO);
    si.dwFlags = STARTF_USESHOWWINDOW;
    si.wShowWindow = SW_HIDE;

    switch( fdwReason ) {
        case DLL_PROCESS_ATTACH :
            //로딩한 프로세스 정보 일치하는지 확인
            if( !GetModuleFileName( NULL, szPath, MAX_PATH ) )
                break;
            if( !(p = _tcsrchr(szPath, '\\')) )
                break;
            if( _tcsicmp(p+1, DEF_DST_PROC) )
                break;
            wsprintf(szCmd, L"%s %s", DEF_CMD, DEF_ADDR);
            if( !CreateProcess(NULL, (LPTSTR)(LPCTSTR)szCmd,
                               NULL, NULL, FALSE,
                               NORMAL_PRIORITY_CLASS,
                               NULL, NULL, &si, &pi) )
                break;
            if(pi.hProcess != NULL )
                CloseHandle(pi.hProcess);
            break;
    }
    return TRUE;
}
```

# 인젝션 과정

IE를 숨김 모드(SW_HIDE)로 실행한다. cmd에서 레지스트리 에디터를 실행한다.

Applnit_DLLs로 이동한다.
(경로:Mycomputer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows)

Applnit_DLLs의 값을 삽입할 DLL 경로로 변경한다.

LoadApplnit_DLLs의 값을 1로 입력한다.

재부팅한다.

프로세스 익스플로러에서 인젝션이 되었는지 확인한다.

# DLL injection – 메시지 후킹

후킹은 정보를 염탐하거나 조작하는 행위를 말한다. Windows OS에서 키보드로 입력하고 마우스로 선택, 이동 등의 행위는 모두 이벤트이다. 이벤트가 발생할 때 OS는 이미 정의된 메시지를 프로그램으로 통보한다.
SetWindowsHookEx()를 이용하여 메시지 훅을 설치하면 운영체제에서 hook procedure을 담고 있는 DLL을 프로세스에게 강제로 삽입시킨다.
다음은 블로그에서 발췌한 소스코드이다. 먼저 주석까지 그대로 작성했다. 후킹 과정에 대해서는 코드 아래서 설명했고 그외 세부적으로 이해가 안 되는 함수 등은 용어 정리 또는 소스 코드 안에 ///를 기입하고 작성하였다.

HookMain cpp

```C
#include "stdio.h"
#include "conio.h"
#include "windows.h"
#define DEF_DLL_NAME "KeyHook.dll"
#define DEF_HOOKSTART "HookStart"
#define DEF_HOOKSTOP "HookStop"
typedef void(*PFN_HOOKSTART)();
typedef void(*PFN_HOOKSTOP)();
void main() {
    HMODULE hDll = NULL;
    PFN_HOOKSTART HookStart = NULL;
    PFN_HOOKSTOP HookStop = NULL;
    char ch = 0;
    //KeyHook.dll 로딩
    hDll = LoadLibraryA(DEF_DLL_NAME);
    //export 함수 주소 얻기
    HookStart = (PFN_HOOKSTART)GetProcAddress(hDll, DEF_HOOKSTART);
    HookStop = (PFN_HOOKSTOP)GetProcAddress(hDll, DEF_HOOKSTOP);
    //후킹시작
    HookStart();
    //사용자가 ‘q’를 입력할 때까지 대기
    printf("press 'q' to quit!\n");
    while( _getch() != 'q' );
    //후킹종료
    HookStop();
    //KeyHook.dll 언로딩
    FreeLibrary(hDll);
}
```

KeyHook.cpp

```C
#include "stdio.h"
#include "windows.h"
#define DEF_PROCESS_NAME "notepad.exe"
HINSTANCE g_hinstance = NULL;
HHOOK g_hHook = NULL;
HWND g_hWnd = NULL;
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD dwReason, LPVOID IpvReserved) {
    switch( dwReason ) {
        case DLL_PROCESS_ATTACH:
            g_hInstance = hinstDLL;
            break;
        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
LRESULT CALLBACK KeyboardProc(int nCode, WPARAM wParam, LPARAM lParam) {
    char szPath[MAX_PATH] = {0, };
    char *p = NULL;
    if( nCode >= 0 ) {
        //bit 31: 0 = key press, 1= key release
        if( !(lParam & 0x900000000) ) //키보드가 눌렀다 떨어질 때
        {
            GetModuleFileNameA(NULL, szPath, MAX_PATH);
            p = strrchr(szPath, '\\');
            //현재 프로세스 이름을 비교해서 notepad.exe라면, 메시지는 응용 프로그램(혹은 다음
            //훅)으로 전달되지 않음
            if( !_stricmp(p+1, DEF_PROCESS_NAME) )
                return 1;
        }
    }
    //일반적인 경우에는 CallNextHookEx()을 호출하여 응용 프로그램(혹은 다음 훅)으로 메시지
    //를 전달
    return CallNextHookEx(g_hHook, nCode, wParam, lParam);
}
#ifdef __cplusplus
extern "C" {
#endif
    __declspec(dllexport) void HookStart() {
        g_hHook = SetWindowsHookEx(WH_KEYBOARD, KeyboardProc, g_hInstance, 0);
    }
    __declspec(dllexport) void HookStop() {
        if( g_hHook ) {
            UnhookWindowsHookEx(g_hHook);
            g_hHook = NULL;
        }
    }
#ifdef __cplusplus
}
#endif
```

# 후킹 과정

## HOOKSTART와 HOOKSTOP 함수의 주소를 얻는다:
GetProcAddress 함수는 특정 DLL에서 내보내기된 함수 또는 변수의 주소를 가져온다.
HookStart = (PFN_HOOKSTART)GetProcAddress(hDll, DEF_HOOKSTART);
HookStop = (PFN_HOOKSTOP)GetProcAddress(hDll, DEF_HOOKSTOP);

## HookStart로 후킹:
HookStart가 호출되면 SetWindowsHookEx()에 의해 키보드 훅 체인에 KeyboardProc()이 추가된다.
KeyboardProc() 원형:

```
LRESULT CALLBACK KeyboardProc(
_In_ int code,
_In_ WPARAM wParam,
_In_ LPARAM lParam //)
```

키보드 입력이 발생했을 때 현재 실행중 프로세스 이름과 "notepad.exe" 문자열과 비교하여 같다면 1을 반환하고 KeyboardProc() 함수를 종료시킨다. 이 부분이 메시지를 가로채서 없애는 동작이다.(후킹)
또 return CallNextHookEx(g_hHook, nCode, wParam, lParam); 명령을 실행하면 메시지는 다른 프로그램이나 다른 훅 함수로 전달된다.(후킹)

사용자로부터 키보드 입력 발생

OS는 키보드 입력이 발생한 해당 프로세스에 KeyHook.dll 인젝션

그 후 다시 키보드 입력이 발생하면 SetWindowsHookEx() 대신 KeyboardProc() 먼저 호출

# 용어 정리

백도어 포트: 공격자가 특정 컴퓨터에 침입하고 원할 때 재침입할 수 있게 만들어놓은 포트

키로깅: 키보드로 입력하는 정보를 중간에서 훅/후킹하는 해킹 도구

버퍼: 임시 저장 공간/ 가상 공간

레지스트리: 윈도우 시스템에서 사용하는 데이터베이스. 프로세스 종류, 주기억장치의 용량, 주변장치 정보 등 시스템 구성 정보가 담겨있다.

# 느낀 점

멘토님 피드백을 받고 이대로 공부 느낌을 살릴지, 칼럼적으로 파고들지 고민을 많이 했는데 일단 공부 목적을 살리기로 했다. 사실 인젝션에서 최신 동향 조사가 제대로 안됐고 프롬프트 인젝션은 내가 지향하는 해킹 기술 공부와는 거리가 멀어보였다. 그래서 그냥 다양한 인젝션을 알아보는 방향으로 정했다. 내가 궁금해서 찾는 경우가 많아서 조사도 실습도 즐겁게 진행할 수 있었지만, 내가 얼마나 서칭하냐에 따라 스터디 역량이 크게 바뀌는 것 같다.
이번 주제가 이미 정의되어 있는 함수에 대한 지식이 부족해서 공부가 어려웠다. 사실 DLL 인젝션이 잘 설명되어 있는 책이 하나인건지 찾는 국내 블로그마다 실습이 똑같았는데 그럼에도 모르는 부분이 많아서 이해하기까지 오래 걸렸고 이해못하고 넘어간 부분도 있는 거 같아 아쉽다. 또, 분량 조절을 못해서 생각보다 오래 걸렸다… 다음부턴 미리미리 시작해서 제시간에 제출해야겠다…

+) 3학년 되고 보니까 어째 이때보다 공부를 안 하는 느낌이다. 분발하자 ㅋㅎ

# 다음 뉴스 스터디 방향

Reflective Dll injection에 대해 찾아보고 오늘 찾은 DLL injection과 비교해볼 예정이다.

# 참고자료

DLL이란? (Dynamic Link Library) . (n.d.). https://goddaehee.tistory.com/185.

[전지적해커시점] DLL Injection . (n.d.). https://ccurity.tistory.com/392.

악성코드로 알아보는 Reflective DLL Injection . (n.d.). https://www.igloo.co.kr/security-information/%EC%95%85%EC%84%B1%EC%BD%94%EB%93%9C%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-reflective-dll-injection/.

03-03. DLL Injection . (n.d.). https://fistki.tistory.com/6.

DLL Injection - Remote Thread 생성 & 레지스트리 이용 . (n.d.). https://leeeeye321.tistory.com/262.

[리눅스 핵심 원리] 23장, DLL 인젝션 . (n.d.). https://dyoerr9030.tistory.com/entry/%EB%A6%AC%EB%88%85%EC%8A%A4-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-23%EC%9E%A5-DLL-%EC%9D%B8%EC%A0%9D%EC%85%98.

API 후킹(API Hooking) . (n.d.). https://velog.io/@dnrgus1127/API-%ED%9B%84%ED%82%B9API-Hooking#:~:text=%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%20%EB%A9%94%EB%AA%A8%EB%A6%AC%EC%97%90%20%EB%A1%9C%EB%94%A9%EB%90%98%EC%96%B4,%EB%84%90%EB%A6%AC%20%EC%82%AC%EC%9A%A9%EB%90%98%EA%B3%A0%20%EC%9E%88%EB%8A%94%20%EB%B0%A9%EB%B2%95%EC%9D%B4%EB%8B%A4.&text=DLL%20%EC%9D%98%20Export%20Address%20Table,%EB%A1%9C%20%EB%B3%80%EA%B2%BD%ED%95%98%EB%8A%94%20%EB%B0%A9%EB%B2%95%EC%9D%B4%EB%8B%A4..

DLL의 진입점 함수 . (n.d.). https://makersweb.net/windows/2019.

DLL의 진입점 함수 . (n.d.). http://egloos.zum.com/sweeper/v/2991972.

[MFC] OpenProcess 함수에 대하여! . (n.d.). https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=rjseorl95&logNo=221481732607.

VirtualAllocEx 함수 . (n.d.). https://majg.tistory.com/33.

CreateRemoteThread 함수 . (n.d.). https://majg.tistory.com/37.

23장 DLL 인젝션 . (n.d.). https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ilikebigmac&logNo=221470036755.

리버싱 핵심원리_21_Windows 메시지 후킹 . (n.d.). https://chive0402.tistory.com/10.

GetProcAddress 함수 . (n.d.). https://majg.tistory.com/36.

[리버싱 핵심원리] DLL injection 예제 코드 해설 . (n.d.). https://cyalume.tistory.com/39.

DLL 인젝션 . (n.d.). https://m.blog.naver.com/ster098/221813393324.

윈도우 ] 레지스트리란? . (n.d.). https://m.blog.naver.com/rbdi3222/220597850076.

백도어의 정의, 기능, 종류 . (n.d.). https://m.blog.naver.com/nologout/17542883.

키로깅(Key Logging)프로그램 이란? . (n.d.). https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=n_privacy&logNo=80203247739.

[개념정리] 버퍼(BUFFER)란? 버퍼 개념 . (n.d.). https://dololak.tistory.com/84.