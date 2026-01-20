---
layout: post
title: "103 – A suspicious developer"
categories: [SWING, Writeup, Self-study]
tags: [PE, build, debugging, analysis]
last_modified_at: 2024-05-30
---

**Description**  
The auditing department of a software development company received an internal whistleblower report stating that a developer from the development team had outsourced a project to another company for execution. The auditing department initiated an investigation to determine whether there had been a violation of internal labor and security regulations. In response, that developer claimed to have personally developed the project and submitted the program source files and the resulting executable files to the audit team as evidence. The auditing department obtained the previous deliverables (executable files) that the developer had submitted by retrieving them from the company’s project management server. Verify the accuracy of the internal whistleblower’s claims.

#PE #build  

1) **Write the items indicating the build tool version information stored in the given two PE format executable files in the format of “[ProductID].[BuildID].[Count]”.** (80 points)  
- Write 9 items per file and do so for both two files. (40 points each)  

PE 파일을 이용한건데 어떻게 찾아야 할까 서치를 여러번 해보았다.  

![Image](https://blog.kakaocdn.net/dna/2sSFe/btsHHMd377q/AAAAAAAAAAAAAAAAAAAAAML6PEKI7STnHx1MFtcmhaEHxwTWNCp_94VXUjkAq-dN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=JxBleCDKrAuylZ62HufL96mrlNI%3D)  

여긴 Rich header라는 곳이다. Rich header는 DOS 프로그램 뒤에 위치하며 PE 빌드 환경 등을 담은 메타데이터 header이다. 공식 언급이 없어서인지 CFF explorer에 분석이 안되어 있어서 수기로 해줘야 한다.  

![Image](https://blog.kakaocdn.net/dna/qVmax/btsHHoEFcjW/AAAAAAAAAAAAAAAAAAAAAKdWsf-LMpKfU2JIbl_M92fRo_lEqrajCyLYIV7C1kcV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=gkP1lDQrnN1OB6cwieG1VvNHsx0%3D)  

이 Rich identifier 뒤 checksum 4바이트로 xor 연산을 해서 복호화(?) 해야 한다.  

![Image](https://blog.kakaocdn.net/dna/XeDhj/btsHHe99MGX/AAAAAAAAAAAAAAAAAAAAAFIuuXBNXd-FoPXdLrBYyZ2LT0BKb-lk9JLTeLZMYJ4K/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xzCIUMeR65WRFmiNeVavbwrkU6c%3D)  

PE-bear라고 Rich hdr까지 분석해주는 툴이 있었다 (포렌식은 템빨이구나!)  

![Image](https://blog.kakaocdn.net/dna/rOSO3/btsHHcYP3jp/AAAAAAAAAAAAAAAAAAAAACg0Wlj-Cw9tbS_-XErSY6PnipeYeXtmFh0pNYZFVAzP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Iss7wXbYXyT2yvqt1wmLA0P2V48%3D)  

DOS header -> Rich Hdr을 가면 위와 같이 productId, buildId, count가 적혀 있다.  

2) **Write the build folder paths for the given two executable files.** (20 points)  
- Write the build folder paths for both files. (10 points each)  

원래 exe 파일이라 ida로 열었었는데 pdb 없다는 메세지박스가 떴었다. pdb는 심볼 파일인데 링크를 위해 디버깅 및 프로젝트 상태 정보가 저장되어 있다.  

[Learn more about PDB files](https://learn.microsoft.com/ko-kr/previous-versions/visualstudio/visual-studio-2010/ms241903(v=vs.100))  

![Image](https://blog.kakaocdn.net/dna/mnnX5/btsHHpKgc4R/AAAAAAAAAAAAAAAAAAAAAMpnzM4_x7HlcW8ZQbKNGEzTTQSUawjIDpFI-KztIy3P/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zxjTe0JWCeza%2BjpIrBh22HIXb3c%3D)  
![Image](https://blog.kakaocdn.net/dna/yYlYT/btsHGYT72N7/AAAAAAAAAAAAAAAAAAAAAE1QXDltuVPE3A_BIFcx5rLiVXHDbCYJzoB41LyWKKrT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=l%2BzJ5HFYj7TAmKn%2F5bM7%2BNJvInM%3D)  

Debug를 가면 AddressOfRawData를 클릭하면 데이터들이 나오는데 PDB 위치가 build된 exe 파일 위치와 동일하다.