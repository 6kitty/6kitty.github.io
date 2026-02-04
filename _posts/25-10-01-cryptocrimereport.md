---
layout: post
title: "자료 정리 1"
categories: [Cryptography & Blockchain]
  - 'Self-study'
tags:
  - stolen funds
last_modified_at: 2025-10-01
---

오늘 뽀갤 자료 The Chainalysis 2025 Crypto Crime Report에서 stolen funds만 발췌독함 

그리고 slowmist, TRM labs 자료까지 보고 싶긴 ㅎㅐ.. 틈틈이 기록 

# 2024년 크립토 플랫폼에서 2.2B 달러 가량이 탈취됨

![연도별 자금 탈취 총액과 해킹 건수](../../../assets/images/251001_01.png)

crypto 가격 상승에 따라 금액도 같이 증가하고 있는 추세(2022,2023 무슨 일이지 이거)

Billion 단위로 stolen된 해는 2018, 2021, 2022, 2023, 2024. crypto hacking이 persisten threat라고 보고 있다. 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_02.png)

cumulative한 그래프이다보니 계속 증가하는 게 맞는데 2024년의 경우 7월 이후 매우 둔화된 양상을 보이고 있다. 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_03.png)

2021 - 2023 동안 DeFi가 주요 타겟이었다. 2024 2분기부터는 중앙화 서비스 비중이 커졌는데 예시로는 DMM bitcoin(24.05)이랑 WazirX(24.07)가 있다. 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_04.png)

private key 탈취가 그 이유라고 보는 듯하다. 2024년에 crypto stolen중 가장 큰 비중이 private key 유출. 이 개인키가 access control를 하기 때문에 중요한 요소이다. 

contract vulnerability가 그다음 
Unknown은 왜지

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_05.png)

이거 좀 흥미롭네. 
해킹 vector에 따라 laundering을 어떻게 했는지 나오는 예시이다. 

첫번째 그래프의 경우 bridge, DEX가 압도적인데(최근 지표로 볼 때) private key compromise의 경우, 여러 방법이 혼용되는 거 같다(왼쪽보다 브릿지, 믹싱이 더 많이 활용)

# 2024 북한 해커 사상 최대 규모 자금 탈취 

2023 20건 사건에서 약 6억 6,050만 달러 탈취 
-> 2024 47건 사건에서 13억 4,000만 달러로 증가 

DPRK가 뭐지 했는데 북한 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_06.png)

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_07.png)

DPRK 비중이 점점 더 커지고, 앞으로도 커질 거라 전망한다. 

사용한 TTPs는 마냥 기술적인 것보다는 허위 신분, 제3자 hiring, manipulating remote work opportunities 등등이 있다. 

mitigate 위해서는.. prioritize thorough employment due diligence? 고용 실사라고 번역해주는데 걍 identity verification라고 인식하자.. 그리고 대부분의 해킹이 연초에 발생함 

# In late June 2024, mutual defense pact 

So far this year, their growing alliance has been
marked by Russia releasing millions of dollars in North Korean assets previously frozen in compliance with
UNSC sanctions.

그리고 북한은 러시아에 병력 파견 + 미

이후 줄어든 loss avg에 대해 조금 더 내용 보충할 필요가 있음 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_08.png)

아 뭐야 
but it is
nevertheless important to note that this decline is not necessarily associated with Putin’s visit to
Pyongyang.
이걸 지금 말해주면 어떡해요? 

# Case study : The DPRK's DMM Bitcoin

the attackers targeted vulnerabilities in DMM's infrastructure, leading to unauthorized withdrawals. 

vulnerabilities에 대한 언급은 없고 laundering에 대해서는 

1. 여러 intermediate address로 흩어져 보내고 
2. 결론적으로 Bitcoin CoinJoin Mixing service에 도달 
3. 믹서 돌린 후 a number of bridging services를 거쳐 
4. 최종적으로 Huione Guarantee(an online marketplace tied to the Cambodian conflomerate, Huione Group)로 빠짐 

여기서 이 휴이온 그룹은 significant player in facilitating cybercrimes로 보고된 바 있다고 한다. 

# predictive models를 활용한 해킹 방어 

걍 홍보 코너 같은데.....
hexagate system은 detect and mitigate threats including cyber exploits, hacks, and governance and financial risks.

암튼 해당 시스템에 쓸 사례로 UwU Lend 해킹 사례를 가져왔다. 

![이건 monthly index에 따른 amount of funds stolen](../../../assets/images/251001_09.png)

1. 두 개의 중간 주소 
2. tornado cash로 유입 






### 메모 

