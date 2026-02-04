---
layout: post
title: "[4주차] Tryhackme: Windows Forensics 1 일부"
categories: [Digital Forensics]
tags: [Windows, Forensics, Registry]
last_modified_at: 2024-04-30
---

### Task 2. Windows Registry and Forensics 
레지스트리:  
windows 레지스트리 = key(폴더) + value(키에 저장된 데이터)  

윈도우 시스템 레지스트리는 아래 5개 root key가 존재  
1. HKEY_CURRENT_USER : HKCU라고 함. 사용자의 폴더, 화면 색상, 제어판 설정 등...  
2. HKEY_USERS : HKU라고 함. 로드된 모든 사용자 프로필 정보, 위 current_user가 해당 키의 하위 키  
3. HKEY_LOCAL_MACHINE : HKLM이라 함. 컴퓨터 관련 구성 정보  
4. HKEY_CLASSES_ROOT : HKCR이라 함. HKEY_LOCAL_MACHINE\Software의 하위 키.  
5. HEKY_CURRENT_CONFIG : 시스템 시작 시 로컬 컴퓨터에서 사용되는 하드웨어 관련 정보  

![Windows Registry](https://blog.kakaocdn.net/dna/JdzsH/btsG5VoYi0l/AAAAAAAAAAAAAAAAAAAAAOtJuLWMfvYIWSQF7nUskTlYjjc8epbOsx51f3oIHyYu/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=mFcbLkEOPvp8cGlIUFekfydVBvg%3D)

### Task 3. Accessing registry hives offline  
regedit.exe 말고 디스크에서 레지스트리에 접근하는 방법 -> C:\Windows\System32\Config  

1. DEFAULT -> HKEY_USERS\DEFAULT  
2. SAM -> HKEY_LOCAL_MACHINE\SAM  
3. SECURITY -> HKEY_LOCAL_MACHINE\Security  
4. SOFTWARE -> HKEY_LOCAL_MACHINE\Software  
5. SYSTEM -> HKEY_LOCAL_MACHINE\System  

이외에 사용자 정보가 포함된 hive..  
C:\Users\<username>\ 에서  

1. AppData\Local\Microsoft\Windows로 가면, USRCLASS.DAT -> HKEY_CURRENT_USER\Software\CLASSES  
2. NTUSER.DAT -> HKEY_CURRENT_USER  

C:\Windows\AppCompat\Programs\Amcache.hve : 최근에 실행된 프로그램 관한 정보  

![Registry Hives](https://blog.kakaocdn.net/dna/2M3Nr/btsG5vxr42Z/AAAAAAAAAAAAAAAAAAAAABneeVvB8TYoh6Yps9gHNXyFfRLU1fsJqd1c8XtudiE2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=K2V2LYKMfNisBpNr%2B9Vw7Qycir0%3D)