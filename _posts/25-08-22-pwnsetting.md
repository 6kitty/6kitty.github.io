---
layout: post
title: "mac m3 포너블 환경 설정"
categories:
  - Pwnable
  - +
tags:
  - mac
  - pwn
last_modified_at: 2025-08-22
---

포스트라고 하기엔 대충이지만 
1. UTM 설치 
2. ISO 다운 (필요한 아키텍처로) 
3. UTM에서 가상환경 생성 후 **에뮬레이터** 로 설치

이러면 이제 엄청나게 느리지만 amd 우분투를 하나 만들 수 있다.
와 잠만 진짜 충격적으로 느리네.  
VScode로 ssh 연결되나 함 보자. 

vm에 sudo apt-get install openssh-server <br>
응 실패다 dkpg 오류 뜨는데.. 너무 느려서 트러블슈팅 하는 것보다 server로 다시 깔았다. 아예 ssh로만 써야겠다. 

[참고한 글 : M1M2-환경에서-Pwnable-환경-구성하기](https://rasser.tistory.com/entry/M1M2-환경에서-Pwnable-환경-구성하기)

1. emulator로 ubuntu server 설치 
2. 초기 config 설정 
3. Openssh server 설치 
4. 설치 다 되면 재부팅을 해야함 -> **이때 iso 파일 꺼내기** 
5. Vscode에 remote ssh extension 설치 
![익스텐션 설치](../../../assets/images/250822_02.png)
6. 연결~

하면 되는데 우분투 서버 설치가 너무 오래 걸린다.. 
다 설치하면 마저 쓰는 걸로 
투비컨티뉴드..