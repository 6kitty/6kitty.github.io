---
layout: post
title: "[HSpace] empty"
categories: [Writeup]
tags: [CTF, Steganography, QR Code]
last_modified_at: 2024-03-09
---

파일은 하얀 png 파일이다

지금까지 관련해서 풀어본 png는..
1. 스테가노그래피
2. PE파일 시그니처 변경

Look closely, because the closer you think you are, the less you will actually see.

라길래 일단 확대를 해보았다 
일반 윈도우 사진 보기로는 5000%만 확대되는데.. 
픽셀까지 보이면 뭐가있을까 싶어서 찾아보려고 했는데 다 확대에 한계가 있어서 pass 
그리고 같이 푸는 동기가 뭔가 색이 다르다고 했다 -> 스테가노그래피 느낌이.. 

스테가노그래피인가 싶어서 Stegsolve라는 툴로 이미지를 열고 화살표로 슥슥 넘기다가 큐알을 발견했다.

그냥 진짜 나우유씨미 공식 영상이라 이게 아닌가 싶었다. 
큐알 관련한 ctf 라이트업을 찾아보다가 QRazyBox 사이트를 찾았다. 

해당 사이트에 들어가서 tool > Extract QR Information을 누르니까 string에 플래그가 떴다.