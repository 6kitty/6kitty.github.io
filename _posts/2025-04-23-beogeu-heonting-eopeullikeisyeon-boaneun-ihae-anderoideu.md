---
layout: post
title: "버그 헌팅: 어플리케이션 보안의 이해 (안드로이드)"
categories: [System Hacking]
tags: [mobile, app, security, hacking, android]
last_modified_at: 2025-04-23
---

Mobile App Hacking 

1. 모바일 앱 공격 
2. 모바일 앱 서버 공격 

Mobile App Attack 
-> 모바일 앱의 흐름을 조작해서 악용하는 것 
로그인 흐름인 것을 우회하거나 
유료인 것을 무료로 우회하거나 

Mobile App Server Attack 
모바일 앱 서버 공격 -> 앱 서버 DB 데이터 추출 
Web App: web view를 이용하여 만든 앱 
Hybrid App, Native App

1. Mobile App 취약점 분석 환경 
Rooting /  Jailbreak 
단말기에서 root 획득하는 과정 
안드로이드에서.. 
1. Boot loader unlocked 
2. custom recovery 
3. custom ROM install 
*Bootloader: 안드로이드 OS 부팅 초기에 실행, kernel을 RAM에 로드 
-> 사용자가 펌웨어를 수정할 수 없도록 Bootloader locked를 건다 
-> unlocked은 보통 fastboot mode에 들어가서.. fastboot oem unlock 

Recovery & Custom Recovery 
Recovery: android OS 유지 관리 기능 제공 (백업이나 기기 초기화 등..) 
Custom Recovery에서는 제한 기능을 우회하기 위한 AOSP에서 제공하는 Recovery+몇가지 추가 
fastboot mode 이용하여 설치 -> 안되면 Odin 

Android ROM -> Android 기기 펌웨어 
Custom ROM -> setuid 권한을 가진 su binary 추가한 버전 -> custom recovery를 이용하여 설치 

Jailbreak -> iOS에서 root 권한 획득 
해당 버전의 취약점이 있어야 Jailbreak 가능 
checkra1n, Unc0ver, Taurine, Unc0ver 등... 
서치는 "Programs used to jailbreak 14.x cve" 등.. 

jailbreak 종류는 아래와 같음 
1. untethered jailbreak 
2. tethered jailbreak 
3. semi tethered jailbreak  

3utools 이용 
만약 탈옥이 되면 Cydia나 Sileo라는 앱 사용 가능 
tweak? 

Proxy 세팅 
단말기 app과 app 서버 사이 프록시 연결 

일단 이거 관련한 실습은 노션에 자료 올려둠 
같은 와이파이에 연결되어 있어야 함 
https 통신을 하기 때문에 버프 인증서 설치가 필요함 
일단 루팅을 해야함!!!!!!!!!!!!!! 
Nox 앱 플레이어로 실습해볼 수도 있음 

Android App  
Android Runtime으로는 세 가지가 있다 
1) JVM 
Java에서 bytecode로... 
이 bytecode를 JVM에서 실행 
2) Dalvik VM 
bytecode 실행 전 네이티브 코드로 -> JIT 컴파일 
3) ART 
OAT file 실행 
앱 설치 시 완전 네이티브 코드로 변환 -> AOT 컴파일 

DEX -> android 실행 파일 
Smali -> Dalvik code를 위한 어셈블리 언어 
java code를 dalvik을 위해 smali라는 어셈블리로 변환 

Android App 위변조 방지 
코드 서명: 개발자 본인의 코드 서명 인증서로 서명 -> 공격자의 악성 패치 앱 배포를 막기 위해 

APK 구조 

cpu에 따른 lib가 다름 

Activity, Activity 생명 주기 
activity: 사용자와 상호작용/ 화면/ 하나의 클래스 

리버싱할 때 위 그림 기준으로 확인하기 

Broadcast receiver