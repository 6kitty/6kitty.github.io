---
layout: post
title: "Cross Origin Resource Sharing"
categories: [Self-study]
tags: [CORS, SOP, Web Security]
last_modified_at: 2025-03-29
---

일단 위 글 보고 SOP이랑 CORS 정리 

1. SOP이란..  
Same-Origin-Policy  
같은 Origin이서만 document를 읽거나 script를 읽을 수 있음  
origin? protocol, host, port로 이루어져 있는..(걍 url이라고 이해)  

이 정책이 있는 이유?  
-> 아래 시나리오를 생각해볼 수 있음  

![SOP 시나리오](https://blog.kakaocdn.net/dna/bf0G9J/btsM1iM8uYv/AAAAAAAAAAAAAAAAAAAAAB0s-bFuEx637e-D7RVmbiliC59JGKMiSe7PRTZjPwbP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=cXCeYKYb57JB7HHOFmcv0Svufo8%3D)  

여기서 SOP를 쓴다면 5번에서 유출을 방지할 수 있다  
(해커가 넣은 iframe의 origin과 상위 document의 origin이 다르기 때문에)  

2. 그래서 CORS란?  
어쩌다가 다른 origin 통신이 필요할 때 사용하는 기능? 기술?이다  
방법은 아래 세 가지  
1. postMessage  
2. JSONP  
3. CORS HEADER  

3. Checking if a whitelisted string is found is a bad approach  
auth_token을 보여주고 이것을 훔치라고 한다.  

![Checking if a whitelisted string is found](https://blog.kakaocdn.net/dna/nn8ew/btsM1k4Jyih/AAAAAAAAAAAAAAAAAAAAANDi5H7K_LL6zzeRNdl-q-frUaWZvUQ6eTkTz2UN88o3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Y6LxlHsewR2rxU9%2BNXKLzAe%2Bi8Q%3D)  

![Another example](https://blog.kakaocdn.net/dna/dYgmrn/btsM0uNPafj/AAAAAAAAAAAAAAAAAAAAAG71HCIL8jwZ-38PdT3-HlDncGbCyW0CdszQDmyEhS5R/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=OgQQYMTNAZZhYsXwNuxiWI5Gj3k%3D)  

![Yet another example](https://blog.kakaocdn.net/dna/Gr0Tz/btsM1eDMwZX/AAAAAAAAAAAAAAAAAAAAAGDNSlOwjwCurvcCJV8gyKk8ofi5KmhwzGA0sZUNR_Z2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=W1Lt1vh5%2BxTwt%2BlouHGcsHJIfDU%3D)  

![Final example](https://blog.kakaocdn.net/dna/bbjvW2/btsM2pYUGy3/AAAAAAAAAAAAAAAAAAAAAGQOZkBVjquxe2fk7C3EIYS2Dd4spBqP5relUNRpBrFz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=MCurez0bXZd4yrVfydFcBqCXcCo%3D)  

안되는디 ㅜㅜ  

![Error example](https://blog.kakaocdn.net/dna/zJrne/btsM2YGyxvC/AAAAAAAAAAAAAAAAAAAAABr7Uwr_YvlB-xcy4KuPVcQDYhzLygdF05hd7Ub-3lrR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=VGCIAmtFIswf729xRC7UtVrbtFM%3D)  

![Another error example](https://blog.kakaocdn.net/dna/HDq24/btsMZNtJmRU/AAAAAAAAAAAAAAAAAAAAAAUqOi7VHfktD20IkGlk8XUMAFF-Wy8BHSQl-oo-jN8T/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QfCP1E3m1Qj1H14Iw3C8mYF8Ibw%3D)  