---
layout: post
title: "[4주차] Tryhackme: Investigating Windows"
categories: [SWING, Writeup, Self-study]
tags: [Tryhackme, Windows, CTF, Investigation]
last_modified_at: 2024-04-30
---

![Image 1](https://blog.kakaocdn.net/dna/ld1TE/btsG5Ja8sC4/AAAAAAAAAAAAAAAAAAAAAFhPKnakGFiQ1pqTplAyqIZ5xjU7FAbehhgyg58UwRk1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=i5eABebSPUcWYBp9thqjVUO9O8I%3D)

![Image 2](https://blog.kakaocdn.net/dna/TXHah/btsG2VJYZ5v/AAAAAAAAAAAAAAAAAAAAAP6rn7G8nV23kAfQ1rztbHJXBKYEMEyRNLb_nizU30en/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MDNEMSBSF7R31t%2F0qlV5gu25Qec%3D)

처음에 version 10.0.14393 했는데 안됐다.

![Image 3](https://blog.kakaocdn.net/dna/sReMo/btsG3stSMqr/AAAAAAAAAAAAAAAAAAAAAJ1xdB632ck6o0rnI9A5iLZp3XedBBf-MeeloF_gHu14/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=yhT7oz1rU2M3MS6NjD1nnhSbd0o%3D)

그냥 windows server 2016 입력하는 거였다.

![Image 4](https://blog.kakaocdn.net/dna/mH1D4/btsG5mghn3u/AAAAAAAAAAAAAAAAAAAAAFNwdXB6qQpyOdlA7Jmsesa7_nZX7mfY0NXYjzezyt0w/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=1LWq3SNx%2BTKuI8VjDVrvH9zqa1w%3D)

레지스트리 탐색기로 가서 Windows\CurrentVersion\Authentication\LogonUI

![Image 5](https://blog.kakaocdn.net/dna/bU3qZl/btsG2WWpR76/AAAAAAAAAAAAAAAAAAAAAA_zox4UNIu6of1hqVdheAVTdOoWU7nkY1d8V56aiQVR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=JEO6CfQXE%2BlmPrlGD9nkqNmpv%2Fo%3D)

LastLoggedOn~을 보면 Administrator임을 알 수 있다.

![Image 6](https://blog.kakaocdn.net/dna/DbfGS/btsG3nHpBhW/AAAAAAAAAAAAAAAAAAAAAFJMivnqqNBL7oLCUzE7GTsDYsJ9MHHX77Yg2cLYO4FI/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=5d%2BnUjftNcSI%2FcGCsoy7u%2Fbj5No%3D)

cmd에 위와 같이 입력하면 Last logon을 알 수 있다.

![Image 7](https://blog.kakaocdn.net/dna/bQRbz0/btsG2X9ZfjS/AAAAAAAAAAAAAAAAAAAAALG8yKAYp17mAxt5Hy5sj9uwsSESoCS-EK_p0fzlYYaX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FK7qSfYd4vCGi8e%2FE3DAypZ%2F3y4%3D)

처음에 가상 머신을 켜면 connecting to...가 뜬다. 민첩하게 외워서 작성해준다.

![Image 8](https://blog.kakaocdn.net/dna/sMyjr/btsG5t7uIlJ/AAAAAAAAAAAAAAAAAAAAAPGrXKZ_vKTrLeZaKBRt4ysMnE3nyhvJHreVfqT8DuH6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=t2UY9OsoA3Quj%2FTc2%2FxMyyIUYso%3D)

컴퓨터 관리자 > Local Users and Groups > Groups  
여기서 Administrators Properties를 누르면 세 명이 나온다.

![Image 9](https://blog.kakaocdn.net/dna/QpHw6/btsG2U6zgJu/AAAAAAAAAAAAAAAAAAAAALUCKt10Yfet4WzZpOabtAZJsUJmtMg_kNit8mJAxqDN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hD1Km0xxOF2B6SyrAbS9ZQuPK9w%3D)

schtasks를 입력했는데 너무 많아서 얻은 건 없었다.

![Image 10](https://blog.kakaocdn.net/dna/baGELQ/btsG3Bq3X72/AAAAAAAAAAAAAAAAAAAAAKNwX-NcH6NFx3KDkwthk0wH87V48VrJ2VpahIDFAwHG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WcbtJyTZPmxEGe4UpRGlx%2BKKMeE%3D)

task scheduler로 가서 예약된 task들을 하나씩 살펴본다. Clean file system은 사실 answer format 글자수 보고 찍었다...

![Image 11](https://blog.kakaocdn.net/dna/cwkT8F/btsG3pygw8n/AAAAAAAAAAAAAAAAAAAAAD85uvmIgK8Uu5sNwj2MgH5ikSDAyHvIFM9nA_QtO-ZP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rF1tWIx6jea07nMLIw1VsBSPScU%3D)

앞문제를 풀었으면 해당 문제는 쉽다. action칸으로 가면 수상한 cmd 명령이 있는데 nc.ps1과 port 번호 입력해주면 된다.

![Image 12](https://blog.kakaocdn.net/dna/Q9SV6/btsG3MmsK5L/AAAAAAAAAAAAAAAAAAAAACVRctPXNebJ5nv8imHZ_iUWbB6ot-HOiY8j9Uo67ngF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zBiIl3w8Ox%2Btup7IkFccRh11pPo%3D)

Windows > System32 > drivers > etc에서 hosts를 열면 외부 ip하나가 google.com을 들어가고 있는 기록을 확인할 수 있다.

![Image 13](https://blog.kakaocdn.net/dna/boOX0w/btsG2X9ZO9Q/AAAAAAAAAAAAAAAAAAAAAAm--AHyvdjLGqnK3CL1XUX4P6nldp-WdltoVZLb3SCy/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vLdQOuBIwrpWaNpNGyaJ9wV1YAs%3D)

inbound 방화벽의 최근을 보면 1337 포트 기록이 있다.

![Image 14](https://blog.kakaocdn.net/dna/YgNlj/btsG6aTIuhs/AAAAAAAAAAAAAAAAAAAAAJnPdqmcmjpGKU2kaIBZYlnBkynEF3grhUfq8lfzJHSm/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=cvj%2BWm9ggfnSpy%2BckBh5uhaWx%2FM%3D)

inetpub > wwwroot로 가면 jsp파일들이 있는데 해당 파일이 서버 웹사이트서 업로드된 shell이다.