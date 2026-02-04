---
layout: post
title: "Parameter Injection"
categories: [Web Hacking]
tags: [CTF, Wargame, Self-study]
last_modified_at: 2025-01-10
---

아마도 중간자 공격을 말하는 것 같다.

접속해보면 이렇게 intercept하고 send할 수 있는 형태이다.

| Alice(개인키 a) | me | Bob |
|------------------|----|-----|
| shared secret=K1-> | intercept-> | send-> | shared secret=K2 |
| shared secret=K1 | <- send | <- intercept | <- shared secret=K2 |
| iv, ciphertext<br/>이것 역시 K1으로 만들어짐 |  |  |  |

이러면 구해야 할 값은 K1이다. pow(alice가 처음 보낸 값,alice에게 보낼 때 사용한 key,p)하면 K1을 구할 수 있다. 내가 쓸 key와 내용을 random으로 만들어준다.

![Image](https://blog.kakaocdn.net/dna/mu9MS/btsLIjNINwe/AAAAAAAAAAAAAAAAAAAAAN__hYxJtV3C7JN0E2jUYzMtuZyP3guSV8xRxNyh44s2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=a8c0eokfov92nvNK%2BBkRadDU6ME%3D)

![Image](https://blog.kakaocdn.net/dna/t7Hnf/btsLIO0PC6x/AAAAAAAAAAAAAAAAAAAAAGrYRixOJFPA2VnSSaQKmcu3IjOITGlzCcYwGOLh6jNG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=kCASlLJpIm0njtavm09aJP9Mn5c%3D)

deriving symmetric keys에서 사용했던 코드 살짝 바꿔서 플래그를 구하면 된다.

![Image](https://blog.kakaocdn.net/dna/zjlbc/btsLIdfUAGu/AAAAAAAAAAAAAAAAAAAAAIIBkBeo2eL4BoGLZp8BXgM81Q9oKk6bUxV28w8lyr3T/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ptzCtbIG9q9gSlCWhR9Gv8Lwkg0%3D)