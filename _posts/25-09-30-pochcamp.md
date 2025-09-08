---
layout: post
title: "31st Hacking Camp"
categories:
  - +
tags:
  - review
last_modified_at: 2025-08-30
---

해킹 캠프 참가했다. 스윙도 31기인데 해캠도 31회 참여라 걍 신기했다. 
메모로 써서 글이 두서 없음 

# 앗, 야생의 아키텍쳐가 나타났다! (TriCore, PPC 리버스 엔지니어링)
...사실 전날 밤을 통으로 새고 간 날이라 많이 졸았다.... 열심히 조는 와중에 들은 건 IR에 기계어로 파싱하고 번역하는 과정을 전반적으로 다뤘는데, 이때 아키텍처가 TriCore나 PPC?라는 처음 보는 친구들이었다. 이부분을 다루면서 calling convention 규약도 다루는 부분이 좀 흥미로웠다(사실 졸다가 여기서부터 들음)
Binary Ninja를 소개해주셨는데 흠 나도 시간 날 떄 함 써볼까? 캡스톤 엔진? 이건 처음 들어봐서 스타 넣어두고 이따 좀 살펴봐야함 

# 왜 우리는 LLM을 해야하는가
정신 차리고 들었다. symbolic execution은 내 연구에도 중요한데.. 이걸 무작위 계산을 하면 컴퓨팅 파워가 매우 많이 소모된다고 한다. 난 이거 어떻게 해결하쥥.. 아무튼 이를 파생으로 여러 취약점 분석 방법론이 나왔다. 
전반적으로 트렌드를 읊어주고 현재는 그것이 AI라는 것. 

: 취약점 점검 코드의 가용성 침해 여부 분석, PoC 코드 작성 및 eXPLOIT 제작
: 2025년 LLM이 0-day 취약점을 찾아냄 

데이터가 없어서 학습하지 못한 것 -> ??? 
우리의 문제를 찾아 해결하는 것 -> 

# 워게임도 모르는 우리가 버그바운티를 성공한 이야기
정찰, hackerone scope 
asset -> weakness -> 심각도(CVSS 참고하고 판단)
제목 : 발견 위치 + 공격 유형 (+공격 영향)
보고서 작성 -> 악용 시나리오(파급력) + PoC

# DBMS 차분 테스팅을 통한 버그 자동 탐지
DBMS에 대한 차분 테스팅 발표였다. commit status 압도적이닫ㄷ.. 
프로그램 -> 입력 -> 버그(취약점) 
퍼징에 대해서는 
입력 -> 프로그램 -> 버그(취약점)
차분 테스팅의 경우 
가운데 프로그램이 여러개임 -> 각각의 결과값(버그가 되겠지?)을 비교하는 방법이다. 
IR-Baed mutations(다람쥐 논문 가져왔다는데 내가 참고해봐도 좋을듯, CCS20) + LLM-Based Mutation 
LLM이 들어갔네? 아키텍처는 논문 쓰는중이라 하셔서 캡처는 안 했다. 버그 짤 수 있는 SQL문에 대해서는 시드 짜기가 어렵다(처음에 이걸 시도 했다가 버린듯)
SQL이 DBMS마다 문법이 다르니까 이거에 맞게 옵티마이저를 해줘야한다. 표준이 존재했기에 가능하다(솔리디티 눈 감아..) 다른 결과값에 대해 레포트를 작성한다. 
기존 퍼징(다람쥐)의 싱글 코어를 멀티 코어 구조로 개선하였다. -> 근데 성능이 더 떨어짐(엥) -> 이부분에서 컴퓨터 성능을 생각하여 퍼징을 조절 + Orchestration으로 리소스 제어 

# Windows Kernel & Namped Pipe BugHunting
커널 주제라 초집중할 생각으로 들었다. namped가 대체 뭐지 했는데 named였다(오타인듯) 팀원 소개에 '시간이 많은 사람들'이 있었는데 팀원 중 한 명을 아니까 반성하는 시간이 되었다. 

학습 커리는 아래 

```
WDM 리버싱 익스 실습 
HackSysExtreme vulnerable Driver 
1-day 취약점 분석 
``` 

# AI, 알고 쓰고 많이 쓰자
요즘의 AI 트렌드를 읊어주는 발표 같다. 챗봇부터 시작해서 RAG와 MCP까지 다룬다. 
AI Development Over the Years 그래프를 보면서 하나하나의 태스크가 무엇인지 훑었다. attention is all you need가 간만이다. 트랜스포머에 대해 알아보기 위해 윙뽀스 플젝 벼락치기할 떄 봤었는데 그떈 이해가 안갔다 ㅎ 생성형 AI의 역할은 RAG? 
오 트랜스포머의 파트 설명을 해줌. Encoder, Decoder 

이미지 -> 수치 -> 이미지의 과정 
보안을 잘하기 위해서는 보안에 대한 수치화? 어떤 걸 어떤 식으로 어떤 구조로 만들건지를 생각해보면 재밌을듯. 

여기서 수치 -> 이미지로 가는 과정이 궁금했던 것 -> 이게 생성형의 시초 
그리고 이것을 잘 활용하는 것이 RAG?

LSTM, transformer를 언제 배웠더라 생각해봤는데 학기중 선행 연구 살필 떄 했었다. 까먹고 있었음. 
E-D 구조 
trend: Test time scaling -> reasoning에 관한 

합성곱 신경망과 이미지 처리 시리즈 + 밑바닥부터 시작하는 딥러닝 5 + 대규모 언어 모델 

학습을 하지 않고 지식을 업데이트 하는 -> RAG 모델 

vector에 대해 확실한 이해가 됐던 건 king, queen 예제 페이지 

api 호출이 되니까 MCP로 PTE 안되나 -> YURA scanner 
-> MCP malicious scanning tool 

server말고 local로 
project naptime -> big sleep 

온톨로지, 제로데이분석, 익스플로잇 자동화, low latency defen, cyptocurrency tracking, OSINT

# 블록체인 생태계 DeFi 하루벌어 하루먹고살기
block의 nVersion -> 블록체인에서는 업데이트가 여러번 되기 떄문에 이런 버전 기록이 중요 
트랜잭션 -> 컨트랙트 상태를 변경하기 위한 증거물
: 검증을 위한 private key, wallet으로 키 관리하고 트랜잭션 만들고.. 이 지갑으로 public key로 생성 가능 
state상에 컨트랙트가 올라오기 state은 full node에 의해 항상 관찰됨
-> merkle tree로 관리됨 -> hash의 tree 형태 -> 빠르게 확실 fast proof 

node
1. full node : current state+ block 
2. archive node : full node의 작업 + parent state 
3. light node : proof request -> 
4. validate node 

block 생성 방법 
1. PoW : mining, 
2. PoS : SLASH로 부정행위 방지 -> 일부를 태워버림 

흠 내가 연구하는 거 ERC 기준 화이트리스트인데 
다른 에코시스템에 어떻게 엮지 

오 괜찮은 설명이 
일정 조건을 만족하면 automatic하게 실행
이 내용을 먼저 듣고 입문했더라면.. 

withdrawAll의 msg.sender에 대한 검증 
receivers.ping에 receivers[i]에 gas 부족할 정도로 DoS 

스테이블 코인의 준비금 불투명성, 디페깅 가능성 
-> 

staking -> 토큰을 컨트랙트에 예치하여 네트워크 보안에 기여하는 방식 
취약점 유형 : slashing 로직 오류, Re-entrancy 

liquid staking -> 발행받은 토큰으로 금융 활동 또 가능 
-> flashloan : aave, dYdX에서 구현 가능 
arbitrage할 때 flashloan 기반 공격(PMA, O)
-> arbitrage, liquidation, MEV 

bridge hacking 궁금하네 

rug pull, 
novitex hacking 재밌어보인다 
Coinbase 
Cetus -> DEX&liuqidity 

EVM -> solidity, foundry 
Solana -> Rust 

damnvulnerabledefi.xyz 

onlypwner.xyz 
paradigm, remedy 

대표적인 DeFi Primitive protocol
Docs analysis -> mechanism, token economy 
Owner/Admin/Timelock 확인 
각 contract의 역할 파악 (Factory, Router, TOken)

PoC는 Foundry의 Fork

solodit.cyfrin.io 

audit -> bug bounty 

