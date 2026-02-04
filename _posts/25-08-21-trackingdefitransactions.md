---
layout: post
title: "논문 리뷰: Risk Management in DeFi: Analyses of the Innovative Tools and Platforms for Tracking DeFi Transactions"
categories: [Cryptography & Blockchain]
tags:
  - blockchain
  - defi
last_modified_at: 2025-08-21
---

위 글 읽어볼거임 

https://www.mdpi.com/1911-8074/18/1/38
내가 이해할 수 있게 의역하였고, 내용도 많이 축약했다. 

# abstract

DeFi는 탈중앙화된 금융 시스템을 말한다. Web3에서 유동되는 가상자산이기에 이 DeFi에서의 Risk management 필요성이 대두되었다. 해당 논문에서는 주요 추적 플랫폼을 평가하기 위한 유틸리티 기반 프레임워크를 제안한다. 설문 조사 및 인터뷰를 결합하여 플랫폼 기능을 식별하고, found significant differences across these platforms with respect to compliance features, advanced analytics, and user experience. 여기서 플랫폼 간의 상당한 차이점이 있었으며, 우리는 유틸리티 기반 모델을 사용하여 priority, risk management needs를 조정할 수 있다. 

# Introduction

DeFi는 cryptocurrency sectors에서 가장 빠르게 성장하는 분야이다. 디파이 플랫폼은 사용자가 digital assets을 빌리고, 빌려주고, 거래하고, 스테이킹할 수 있게 해준다. 디파이는 많은 이점이 있지만, 몇가지 위협도 존재한다. smart contract vulnerable areas, price swings, market manipulation, liquidity issues, and management uncertainty 등이 존재한다. flash load attack이나 smart contract vulnerabilities 등 많은 손실이 존재 했기에 DeFi transactions 추적과 possible threats를 찾는 risk management tools and systems가 필요하다. 게다가 이 decentralized struture은 규제 감독을 복잡하게 하고, AML, KYC compliance를 복잡하게 한다. 

그동안 DeFi에서 리스크 관리를 위해 numerous DeFi tracking systems and plaforms이 등장하였다. 이것은 디파이 환경에서 트랜잭션을 트래킹하고, 포트폴리오를 관리하는 적절한 Tool을 선택함으로써 potential risks and possibilities를 이해시킬 수 있다. 

이 DeFi risk와 관련해서 다양한 연구가 진행되고 있는데, smart contract audit, liquidity management 등이 급히 논의되고 있다. 근데 이제 논문에서는 DeFi 프로토콜에서 transactional tracking과 anomaly detection에 대한 연구가 거의 없다고 한다. 때문에 해당 논문이 제시한 기여점은: 

1. utility-based evaluation model 제안 : 정확성과 응답성을 결합한 새 프레임워크를 제안한다. 이때 정확성과 응답성은 각각 의심스러운 활동을 얼마나 오탐 없이 잡아내는지, 얼마나 real-time으로 경고가 나오는지이다. 
2. DeFi leading analytic platforms에 대한 비교 분석 수행 : Chainalysis, Elliptic, Nansen, Dune Analytics, DeBank, Etherscan에 대해 compliance tools, advanced analytics, user interface 측면에서 평가하였다. 
3. 설문조사와 인터뷰를 합하여 사용자 선호도 조사 

이전 연구에 대한 언급은 내가 공부하려는 AML, KYC 등에 대해서만 짚었다. 

해당 논문의 핵심 과제는 다음 질문과도 같다. How can risks in DeFi transactions be effectively managed(거래를 효과적으로 관리하는 방법), and what role do existing platforms play in mitigating these risks to ensure the stability, security, and adoption of decentralized financial systems(기존 플랫폼이 리스크 완화..등 다양한 방면에서 어떤 역할을 하는지)?

AML, KYC는 블록체인의 거래가 본질적으로 가명이기에 준수하기 어렵다. 이외에도 스마트 컨트랙트에서, DeFi 환경에서 여러 취약점이나 위협들이 논의되고 있다. 실제로도 최근 BadgerDAO 등 DeFi에서 일어난 공격들을 선제적으로 관리할 수 없었다. 

DeFi에서 이런 포괄적 리스크 관리 프레임워크가 필요한 이유는 : 
1. DeFi 프로토콜은 인터렉션이 촘촘하기에 생태계 안정성이 중요하다. 
2. 사용자 자금이 보장되어야 하기 때문이다(번역 이게 맞나)
3. DeFi에서는 사용자 신뢰가 매우 중요하다. 

그동안의 리스크 관리는 매우 단편적이다. 전체 범위의 DeFi risk를 관리할 수 있는 프레임워크를 만드는 것이 관건이다. 

# Overview of DeFi 

해당 단락은 내가 필요한 부분만 발췌해서 작성하였다. 

DeFi 생태계에서 lending, borrowing, trading, staking, liquidity provision에 대한 서비스를 제공한다.[^1] 그리고 DeFi 프로토콜에서는 중개인 없이 앞서 나열한 서비스를 제공할 수 있도록 lending and borrowing arrangements를 구성한다. 시장 수요와 공급이 interest rates를 만든다. Lending protocol인 Aave, Compound, MakerDAO 등은 차용자(borrower)가 담보를 사용하여 암호화폐로 대출 받고 이자를 받을 수 있게 한다. 

DeFi는 보통 분산형 거래소(이하 DEX) 간의 거래를 말하며 Uniswap, CurveDAO, PancakeSwap 등이 이에 속한다. 여기서 사용하는 contract는 기존 거래소보다 여러 이점이 있다. 

staking은 네트워크 운영하기 위해 암호화폐를 특정 지갑에 Lock up하는 DeFi 기능이다. 자세히는 staking이 주로 PoS 방식 블록체인 네트워크에서 트랜잭션 검증 및 블록 생성을 위한 기능으로 사용된다. 

[^1]: <https://link.springer.com/article/10.1057/s41267-024-00705-7>

(작성중)투비컨티뉴드..