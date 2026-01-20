---
layout: post
title: "dll 호출 방법"
categories: [SWING]
tags: []
last_modified_at: 2024-01-09
---

dll이란: 실행파일 안에서 변수, 함수 등을 공유하기 위해 만든 라이브러리

dll을 끌어오는 lib파일 안에는 dll이 export한 함수 각각에 대한 stub이 있다.

stub: 함수 호출에 쓰이는 정보, 진짜 함수와 동일한 이름/인수 리스트를 가진 pseudo 함수

lib를 import library라고 부른다

호출 방법
1. implicit(생략)
2. explicit
   2-1. HINSTANCE LoadLibrary( LPCTSTR IpLibFileName );
   2-2. GetProcAddress 함수
   2-3. IpFactoryFunc(num) 함수
   2-4. FreeLibrary(hDll)

![Image](https://blog.kakaocdn.net/dna/CzuGq/btsDhV7lgRL/AAAAAAAAAAAAAAAAAAAAAGsQ3BtqTJBSWtuvg0Yw9Vy5fpClqP41zHxkUGupCOgN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=b2G8bBM5pKxOwmqU6iFJKlbL8ec%3D)

변수 공유

![Image](https://blog.kakaocdn.net/dna/d1WWHq/btsDavWqQDS/AAAAAAAAAAAAAAAAAAAAAKKRK4N27yquIZCso-b7rwTLmScrwFZscHesJZot1kps/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QavNxPjOrWWHtuVTBZjxZNAg0GU%3D)

함수 공유

![Image](https://blog.kakaocdn.net/dna/ojqT2/btsC9a6sLEQ/AAAAAAAAAAAAAAAAAAAAAKK2wmhuczYmgrYzZ5vNI-_u-KUs6u1AihY8W6ixJXam/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BFfUNX9hxPQAUyfDeTzT6By70jQ%3D)

1. implicit

![Image](https://blog.kakaocdn.net/dna/ldUyk/btsDh613UyW/AAAAAAAAAAAAAAAAAAAAAIR-LMKiWIEKRVFwbpZxW37-1RyeJjh66EWE_fpuUfGo/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Wbk2P2%2FSReV6Ru3wqtPlLxmu0fQ%3D)

2. explicit

![Image](https://blog.kakaocdn.net/dna/ZJ3or/btsDhVM2nbi/AAAAAAAAAAAAAAAAAAAAAHdaYqSPQEKZeJDTffybqsKmNOjD4VUIZX5jLO6_LaLc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=24RVFJV9qkqJFM6wn8thUht8G5I%3D)

[더 알아보기](https://clarus.tistory.com/entry/%EC%89%BD%EA%B2%8C-%EC%93%B4-dll)