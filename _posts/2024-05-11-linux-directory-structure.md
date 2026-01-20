---
layout: post
title: "리눅스 디렉토리 구조"
categories: [Self-study]
tags: []
last_modified_at: 2024-05-11
---

리눅스 루트 디렉토리는 트리 구조로 구성되어 있다.

**/bin User Binaries**  
이진파일  
기본적인 명령어가 저장되어 있다. mv, cp, ls 등...

**/sbin System Binaries**  
시스템 바이너리, ifconfig, ethtool, halt 같은 시스템 명령어 저장되어 있다.

+ 추가  
- bin: cd, ls 등의 사용자 커맨드 파일이 위치한 디렉토리 (필수적인 파일만 관리)  
- sbin: systemctl 등의 시스템 커맨드 파일이 위치한 디렉토리 (필수적인 파일만 관리)  
- usr/bin: 필요에 의해 설치된 사용자 커맨드 파일이 위치한 디렉토리 (yum 등 패키지 관리자가 관리)  
- usr/sbin: 필요에 의해 설치된 시스템 커맨드 파일이 위치한 디렉토리 (yum 등 패키지 관리자가 관리)  
- usr/local/bin: 기타 사용자 커맨드 파일이 위치한 디렉토리 (사용자 또는 설치 파일이 해당 디렉토리에 파일 설치)  
- usr/local/sbin: 기타 시스템 커맨드 파일이 위치한 디렉토리 (사용자 또는 설치 파일이 해당 디렉토리에 파일 설치)  

**/etc Configuration Files**  
설정 파일을 두는 디렉토리  
네트워크 관련 설정파일, 암호 정보, 보안 파일 등...

**/dev Device Files**  
시스템 장치 파일이 있는 디렉토리  
하드디스크, CD-ROM 등이 존재한다.  
피지컬 장치를 파일화 하여 저장  

**/proc Process Information**  
현재 메모리에 존재하는 작업들을 파일화  
프로세스 정보 등 커널 관련 정보가 저장  

**/var Variable Files**  
로그파일, DB 캐싱 파일, 웹서버 이미지 파일 등...  
동적인 파일들(로그) 저장  

**/tmp Temporary Files**  
임시파일 저장 디렉토리  

**/usr User Programs**  
루트 계정 아닌 일반 사용자들이 사용하는 디렉토리  

**/home Home Directories**  
홈 디렉토리, id마다 한 개씩 존재  

**/boot Boot Loader Files**  
시스템(여기서는 리눅스) 부팅에 필요한 정보 파일이 있다.  

**/lib System Libraries**  
커널이 필요로 하는 라이브러리 파일, 모듈파일 존재  
bin, sbin 실행에 필요한 공유 라이브러리  

**/opt Optional add-on Apps**  
추가 응용프로그램 패키지 설치  

**/mnt Mount Directory**  
아래 media와 비슷하지만 해당 디렉토리는 사용자가 직접 마운트  
OS가 자동으로 마운팅하는 건 media로...  

**/media Removable Devices**  
외부 장치들의 연결 포인트로 사용하는 위치(디렉토리)  

**/srv Service Data**  