---
layout: post
title: "[심화스터디] 해외 케이스노트 읽어보기"
categories: [Self-study, Writeup]
tags: [Temu, 개인정보, 보안, 소송]
last_modified_at: 2024-08-02
---

[https://www.mk.co.kr/news/world/10963791](https://www.mk.co.kr/news/world/10963791)

![Temu Article](https://blog.kakaocdn.net/dna/bp9Ayr/hyWG1aqxln/AAAAAAAAAAAAAAAAAAAAAK7MJ8urEp_tEefYfiNuxoLLGhfVFd4ojoAMxM0b_1Gw/img.jpg?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Wqu3xYdrC7KjHwVI7kgiSPlfh2E%3D)

테무 회원 가입 시 개인정보가 중국으로 유출될 수 있음  
미국에서는 개인정보 노출 관련 소송 진행중  

소비자 측: 테무가 고객 알림과 메시지 추적 의심  
1. 테무 앱의 코드가 난독화  
2. 다른 앱보다 4배 더 많은 권한 허용 필요  
3. 백그라운드 앱 활성화  

[해당 링크는 2024 6/24의 피고 테무에 대한 complaint(고소/고발)이다.](https://cdn.arstechnica.net/wp-content/uploads/2024/06/Arkansas-v-PDD-Holdings-Temu-Complaint-6-25-2024.pdf)  
case no. 12cv-24-149  

미국의 complaint에는 사건과 사실 관계, relief(주문사항) 등이 존재한다.  
다만 우리 소학회는 기술적으로 더 파보기를 원해서 introduction 내용만 정리 후에 GRIZZLY research팀이 작성한 기술 블로그를 분석하였다. 원고가 주장하는 내용의 진위 자체를 따지거나 재판 자체에 대한 의견은 나누지 않았고 어떤 조사가 이루어졌는지에 집중하여 스터디를 전개하였다. introduction 내용중에서도 스터디 방향과 맞지 않다 생각한 부분은 스킵하였다.  

Plaintiff: state of arkansas  
Defendants: TEMU  

### Introduction  
1. temu는 online shopping platform으로 위장한 malware에 불과하다. temu 앱은 사용자의 핸드폰 내 거의 모든 데이터 액세스를 갖는다.  
2. 사용자의 핸드폰 OS, 카메라, 특정 위치, 연락처, 메세지, 문서, 다른 어플리케이션 등 access 권한을 얻도록 설계되었다. 한 번 설치되면 테무 앱은 스스로 재컴파일하고 properties를 변경할 수 있다. 또한, 개인정보 보호 설정도 수정 가능하다.  
8. Temu와 판둬둬(Temu를 운영하는 회사) 앱에 대해 security and privacy 문제를 제기하였다.  
9. 애플 앱스토어와 구글 플레이 스토어의 잇따른 판둬둬 앱들 제재를 가하였다(정지처분)  
10. 이후에 Temu에 관한 자세한 조사 진행, malware, spyware 실행 도구가 임포트되어있다고 주장하였다(주장, 진위 여부 파악 X)  
11. 이커머스 목적과는 먼 데이터까지 수집 가능하다(GPS, 생체 정보)  
12. 원격 실행, 무단 액세스가 가능하다.  
13. Temu 앱의 코드는 의도적으로 프론트엔드 보안 검사를 우회하고, 사용자의 휴대폰에 다운로드된 후에는 자체 코드를 변경할 수 있도록 설계되었다.  