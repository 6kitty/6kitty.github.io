---
layout: post
title: "자료 정리 2"
categories:
  - Blockchain
tags:
  - aml
last_modified_at: 2025-10-01
published: false
---

UTXO에 대해 소유권이 변경이 되었는지 -> 이부분에서 경우의 수가 다양해짐 
flow보단 Ownership이 중요한데 

경우의 수가 보통 
1. payment : 이전  
2. self-transfer : 
3. change : 거스름돈 

추가적으로 세 가지 정도 더? 
1. 
2. 
3. 

거스름돈의 tx 주소 형태 -> 원본과 비슷? (암호학적으로 증명된건가?)

보통 흐름 분석에서 
1. 
2. 
3. 

## co-spend heuristic 

받은 UTXO를 co-spend 형태로 input을 들어가서 한 tx로 들어간다면 한 사람의 ownership 하에 있는 UTXO라고 휴리스틱 가능 

이부분 한 번 더 보고 

컴플라이언스 이유 때문에 output에 vout이라는 인덱스가 삽입됨 

이더리움에서는 output에 대한 footprint 개념?이 없어서 어려움? -> 비트코인은 

## mixing 

mixing, distribution tx 두 개로 



coinjoin이 깬 co-spend heuristic 

개많은 tx -> 개많은 tx 이정돈 휴리스틱
그치만 소량을 여러번 반복하면? 


payjoin -> 수수료 정리를 위한.. laundering 목적이 아님 -> input 개수가 달라지고 
