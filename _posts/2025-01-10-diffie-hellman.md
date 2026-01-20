---
layout: post
title: "diffie-hellman"
categories: [Self-study]
tags: [Cryptography, Diffie-Hellman, Finite Field, Python]
last_modified_at: 2025-01-10
---

cryptohack starter 스테이지 하나씩 풀어보자  

![영어다....](https://blog.kakaocdn.net/dna/IeDkw/btsLJkY95M6/AAAAAAAAAAAAAAAAAAAAAJFb_j-3jo764kWzMBum0dIrdAQX2f8IlUGVTqlp2dKZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=TFPe2Qa0F%2FuQgAoPPqodZeojXFU%3D)  
정수 modulo N으로 이루어진 set은 덧셈과 곱셈을 포함하여 R이 된다.  
n=p(소수)일 때, set 내 모든 요소의 역원은 존재한다. 때문에 R은 F가 가능하다. 이 F를 finite field(?) Fp라고 한다. Diffie-Hellman은 큰 소수인 유한체 Fp의 element들로 이루어진다.  

**문제) p=991과 F(991)의 element g=209가 주어졌을 때, find the inverse element d such that g*d(mod991)=1**  
파이썬 코드를 짜서 해결하자  

```python
for i in range(0, 990):
    res = (i * 209) % 991
    if res == 1:
        print("inverse: %d" % i)
```

![Image](https://blog.kakaocdn.net/dna/Ucppy/btsLIV6lBfB/AAAAAAAAAAAAAAAAAAAAANeX3QD1kMVk3ORwxULenwq_9UEKQ3_xJiXtmE6YrgbF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=aqsCo8IvEsK6hvrbK12Jsz6dqX4%3D)  

Fp의 모든 element는 제곱을 통하여 subgroup H를 만들 수 있다. element g의 subgroup H=<g>  
primitive element(?) Fp는 H가 Fp*이다. 0을 제외한 Fp의 element는 어떠한 정수 n에 의해 g^n(mod p)로 정의할 수 있다.  
이때 primitive element는 generators of the finite field로 불린다.  

**문제) F(28151)에서 primitive element가 될 수 있는 가장 작은 element g 구하기**  
브루트포스보다 더 괜찮은 방법을 찾아보라고 한다..  

일단 기초정수론에 대해 가물가물하므로, 몇가지 복습했다.  
Zn* 기약 잉여계: Zn의 원소 중에서 곱셈에 대한 역원이 있는 숫자들의 집합  
이때 n이 소수 p이면, Zp*={1, .., p-1}  

문제를 수식으로 정리하면,  
F(28151)*=<g>={1,...,28150}={g^n,g^(n+1),...}일 때 g 구하는 방법  
1. 소인수분해 28150=2*5^2*563  
2. 2부터 p-1중에서 다음 조건 만족  

```python
#소인수 n일 때 
g^(28150/n) =/= 1 (mod 28151)
```

파이썬 코드를 짜보자  

```python
p = 28151
arr1 = [2, 5, 563]
arr2 = [(p - 1) // arr1[0], (p - 1) // arr1[1], (p - 1) // arr1[2]]

for g in range(2, p):
    flag = True
    for i in arr2:
        if pow(g, i, p) == 1:
            flag = False
            break
    if flag == True:
        print(f"The smallest g={g}")
        break
```

![Image](https://blog.kakaocdn.net/dna/Ffk3W/btsLJA8wAi8/AAAAAAAAAAAAAAAAAAAAAD-KnqXlI3fahuF2XmU1SBePVNlwFnuD6Z-0bArdqSoV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=NRgeUfBqxq8dNpMZw0NvqcHmlXo%3D)  

Step 1. 소수 p와 생성자 g 설정  
Step 2. p=2*q+1이며 p-1=2q(q는 소수)  
Step 3. a<p-1일 때, g^a(mod p)  

**문제) 주어진 조건에서 g^a mod p 구하기**  

![Image](https://blog.kakaocdn.net/dna/G7wUg/btsLIa4gYZy/AAAAAAAAAAAAAAAAAAAAAN3Orji8y9dQC9GxzldfGCUOSCQ5tp1LmMv2s59K7sYG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MWGk3dgphDA5nUPvGyZ15UHYKw4%3D)  

**문제) shared secret 구하기**  
앨리스한테 받은 거 g^a=A(mod p)에서 a 모름  
내가 보낸 거 g^b=B(mod p)  

shared secret K는 다음과 같다. A,B 값을 서로에게 공유하여  
B^a=A^b=g^ab (mod p)  

때문에 a를 몰라도 A^b로 계산해주면 아래와 같다.  

![Image](https://blog.kakaocdn.net/dna/Rxq1m/btsLJbOGUvS/AAAAAAAAAAAAAAAAAAAAAE3jjZVTHytCSC6hqGYQFdLSCWNyNUP3oH_wt1-BWuPs/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=nMPIgBJyf6WIRlKThcs9uEMY1zo%3D)  

shared secret으로 AES key를 만든다  
어렵지 않고 그냥 shared secret 만들어서 decrypt.py 파일에 넣어주면 된다  

![Image](https://blog.kakaocdn.net/dna/DEmiF/btsLIbWq7Fb/AAAAAAAAAAAAAAAAAAAAACBO-_9egF5QHbgN7GwlAgs_R5NCN_amtt9XO9P9s-T1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=E5eQhGXhRgkwfgv7hzpXP5e%2F%2BTw%3D)  

![Image](https://blog.kakaocdn.net/dna/cNwfgl/btsLJve4opi/AAAAAAAAAAAAAAAAAAAAAMGTeKYeRj1Bx9DWPJdui44rKD4mRhnx7eiQ8r6qrBK0/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QjSgeOOrv0bDwYxc87qvaZvkMKc%3D)