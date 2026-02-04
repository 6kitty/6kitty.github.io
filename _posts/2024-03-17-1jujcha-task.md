---
layout: post
title: "[1주차] task"
categories: [Digital Forensics]
tags: [CTF, Steganography, EXIF, Signature Analysis]
last_modified_at: 2024-03-17
---

지원하지 않는 파일이라고 뜬다.   
1주차에 배운 exif 문제, steg 문제 했으니 필시 시그니처 분석일 것이다.  

![Image](https://blog.kakaocdn.net/dna/6YfCo/btsFPsbSLxs/AAAAAAAAAAAAAAAAAAAAAG9F15Bcfm3Av9-y7A8zdGcHKyANdTm6GIXnA4QE1UlI/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=KQM5g%2Fu4Wv25t5H7srWCPEjsnqo%3D)  

PNG 시그니처가 섞여있다. 해당 파일을 PNG로 바꾸거나 JPG로 바꿔야 하는데 옆에 IHDR이 있는 걸 보아 일단 PNG 파일 구조로 맞춰줬다.  

![Image](https://blog.kakaocdn.net/dna/qs1eU/btsFRq4UZy8/AAAAAAAAAAAAAAAAAAAAAMylGx8l2-cO59RV0eYrThMPzOCSHgLybDFF4Vor0zAN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=giszODZvOrMC42mDTZNoWRAeebM%3D)  
![Image](https://blog.kakaocdn.net/dna/be2o2C/btsFRa2j2pM/AAAAAAAAAAAAAAAAAAAAAHST7x2qVpRJjSPmNhnfDyzUf9oZeSazkzsEOaNxjWd4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=kwRpj8wvgJ%2Bl57%2FCIC6NdFwtg4Y%3D)  
![Image](https://blog.kakaocdn.net/dna/n5Pda/btsFQeqUfgm/AAAAAAAAAAAAAAAAAAAAADY2p0V-xS7GcTVOciqUaZ9OHs-cA-pPmrcDzC_ICwzc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=bSVFMBOx5%2BsMndXJWrhdgww%2BV2I%3D)  
![Image](https://blog.kakaocdn.net/dna/bVSdxJ/btsFQfDraoP/AAAAAAAAAAAAAAAAAAAAACs3Zw9Pwy0alhTDSMRSGhlodkb4u1AHjrOINB1jTrAh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=f2D0jz6s78h2vW7rsCLez3U9V7E%3D)  

일반 미리보기는 아무것도 안 떠서 steg로 이어지나 싶었는데 뭔가 추가로 포맷을 맞춰줘야 하는 거 같다.  

![Image](https://blog.kakaocdn.net/dna/ct0oPd/btsFPGhc4fW/AAAAAAAAAAAAAAAAAAAAALe8CFIlD16p65_jpjmxp25HIkkN1qvVdzyNXsity3U1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BVL87RIckAqhbl6evsdr9S7oNZk%3D)  

IHDR 청크부터 보겠다.  

![Image](https://blog.kakaocdn.net/dna/OnXnr/btsFPHNV0Xe/AAAAAAAAAAAAAAAAAAAAAPPxKDsWG_yOuv7udaT0ZpwcvFTMx1JLiGSa-o9AKrGv/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zcHi%2Fpa4iIGlHB8lV9v%2BQ2TpNLg%3D)  

CRC 뒤 더미가 있길래 지워줬다.  

![Image](https://blog.kakaocdn.net/dna/cR6HKC/btsFOYB3Gk3/AAAAAAAAAAAAAAAAAAAAAK-h79fcFuUr9FBFhFUE4yHV0LmsPgIFgkqZEjsecGKD/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=FXlzisGOclN3MzWRbIpbfXsdpdY%3D)  

IDAT를 봐야 한다. chunk type 앞 4 bytes는 length인데 이값이 맞는지 확인해줘야 한다.  

![Image](https://blog.kakaocdn.net/dna/b2fifx/btsFQZ06NUv/AAAAAAAAAAAAAAAAAAAAAOfu-G0oWOw4G7RoXofm5TZqqlizXeF4XRBrKgocnSIR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=geJvuyi6N%2BaCfK2To6leogsGo8I%3D)  

청크 데이터를 긁어오니까 오프셋이 a5 b5이길래 수정해줬다.  

![Image](https://blog.kakaocdn.net/dna/HxthJ/btsFQcGBzZR/AAAAAAAAAAAAAAAAAAAAAL1Pi5LtyjwQMtuB7T-t1wnxJDb6wYuPHzHnQ67Zz-2c/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ZutfvnUg4EKrSu3ZqxOD0q30jwA%3D)  

a5 b0이라고 썼지만 후에 b5로 수정했다(그래도 안됐다)  

![Image](https://blog.kakaocdn.net/dna/DjTPX/btsFO1yQd0m/AAAAAAAAAAAAAAAAAAAAAKpzDAlat2NROxL5vp0IClMHKZW2eV4jkto3JWFEKvsm/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=dwWK%2FXrtU8dTMgZjOJNcKE0ceqw%3D)  

IEND 청크는 그대로였다.  

그래도 오류가 뜬다. 원본 파일 IDAT를 다시 검색해보니  

![Image](https://blog.kakaocdn.net/dna/Zz8G5/btsFQdMjj3P/AAAAAAAAAAAAAAAAAAAAAB7bgZn5NHrJDyZq8-wAtHAigrcSXTkqHeTWBZGIPJgj/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=1FFE9B%2BlY9KgwjIIcmSEbQaj3Lk%3D)  

두 개였다.  

![Image](https://blog.kakaocdn.net/dna/bs3ACs/btsFQv0ds0u/AAAAAAAAAAAAAAAAAAAAAFpwkRs8MsYZVip8C0EJNU9nCPcuinQf1XQMN7XQP36t/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=GSBl7GJJTUW6z9ML3GrP8%2BymWOU%3D)  

그래서 length 한 번 더 바꿨는데도 오류가 났다. 고민하다가 다른 툴을 써보았다.  
010 에디터가 청크 분석을 해주는 거 같아서 깔아보았다.  

![Image](https://blog.kakaocdn.net/dna/bsvTxx/btsFQwdKuWL/AAAAAAAAAAAAAAAAAAAAAGzl4SrGB-_ip5vvxeaymTGC3bI7gipwysrmkqoxPzoV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vyhQ45GOV8UEDbSSWFmY6fOM%2F7M%3D)  

청크 분석을 해주었는데 fcTL이 뭔지 몰라서 검색했다가 본 문제 라이트업을 발견했다..  

tweakpng 툴로 바꿨다. 010 에디터는 파일이 온전할 때 제대로 분석해주는 거 같다.  

![Image](https://blog.kakaocdn.net/dna/bAqKKP/btsFO0fDFE3/AAAAAAAAAAAAAAAAAAAAAFBABrHHkuzUAyQDFEbk-kr_Z6Rm16-QjVnQwVew4EwE/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ty1PlxBv85ty9VbvunB%2FODlWNsc%3D)  
![Image](https://blog.kakaocdn.net/dna/c30VxL/btsFQg3lmc7/AAAAAAAAAAAAAAAAAAAAAGoXJ05Yicrukbw63ayH-qbTC01CxIpqtHaUNCsgiMtg/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=kG5Kw6Eg3aFfkwddeMhi3oo2NqQ%3D)  

위 순서를 지켜 바꿔줘야 한다. 그리고 IDAT는 크기가 큰 것부터 배열해야 한다.  

![Image](https://blog.kakaocdn.net/dna/Kd9UE/btsFQXa92Jz/AAAAAAAAAAAAAAAAAAAAAMhnCzuwut9iWZRrpaUOQrPJ6BAe8eP9dG6xGwQSrg4a/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=eau4jr01qGXomR2LHUPoP5YYLDg%3D)  

티스토리에 사진으로 업로드 하니까 플래그가 나온다.