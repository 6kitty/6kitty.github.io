---
layout: post
title: "[3주차] network_recover"
categories: [Digital Forensics]
tags: [CTF, Writeup, Network]
last_modified_at: 2024-03-31
---

뭔 파일인지 몰라서 hxd로 열어보았다.  
왼쪽에 Decoded text를 보니 packet 구조인 거 같아서 확장자를 pcap으로 저장했다.  

![스크린샷 2024-03-31 011757.png](https://blog.kakaocdn.net/dna/cgdODG/btsGfynKj6U/AAAAAAAAAAAAAAAAAAAAAHPnU3S-9WBEflkGWPcbbebRihlzm-U5Ial6fGAvUgxu/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CrIzfsKDIm4389OKXC20cx4Fc%2B0%3D)

와이어샤크로 열어보았다.  

![스크린샷 2024-03-31 011949.png](https://blog.kakaocdn.net/dna/d80Qfq/btsGd9JoMcN/AAAAAAAAAAAAAAAAAAAAAE2TGBemPOwaCRkJaAAz-C4hhcKcBkzI258S0H937BFj/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Q5K%2BtpCHQgg8kb0r7YWpAm0VMRw%3D)

![스크린샷 2024-03-31 014049.png](https://blog.kakaocdn.net/dna/Utu08/btsGeecIqwa/AAAAAAAAAAAAAAAAAAAAADE8nsbjc1kYlrQVf8SjYsaMCJY85KFMAqyaZdYHS47a/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=h7%2FPT2PkjDEFsF5AK8ToRd9fy34%3D)

위 Malformed Packet이 수상하다.  
검색했더니 라이트업을 발견했다 ㅎ  

![스크린샷 2024-03-31 020937.png](https://blog.kakaocdn.net/dna/tikbo/btsGfeQtXun/AAAAAAAAAAAAAAAAAAAAAB6gCD9Wj3bm3sayfof7A_xh45tG3QnqtVFsX-4yFkai/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=l63CFJnatBZBcJhg61mdMC7x%2BS0%3D)

해당 문제는 treasure1만 보면 안되고 2,3도 봐야 한다.  
treasure3까지 hex를 추가해야 푸터 시그니처가 들어간다.  

![스크린샷 2024-03-31 020948.png](https://blog.kakaocdn.net/dna/dkat6L/btsGgc5PvZ3/AAAAAAAAAAAAAAAAAAAAAKn_S1CWTAlrdwZNB1tHqzO5eLiOZMjYctM3aW_del81/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MFZhLrYnHrEKD%2FTuz4ybPc05zDw%3D)

hxd에 treasure1,2,3 내용을 다 복붙하고 저장해주면  

![network.png](https://blog.kakaocdn.net/dna/bZPsql/btsGePpRQk3/AAAAAAAAAAAAAAAAAAAAAIJ1BvgrTx7xtjxtG6YlYnbGrOUbrZ8Ag_nL-7bZaDUb/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=DgXtuFqUIObmY7Py6g2awsd%2B71c%3D)

플래그가 나온다.