---
layout: post
title: "pico- WinAntiDBG0x300"
categories: [System Hacking]
tags: [Reverse Engineering, Debugging, CTF]
last_modified_at: 2025-04-13
---

![Image 1](https://blog.kakaocdn.net/dna/bS8nng/btsNgiUlvx2/AAAAAAAAAAAAAAAAAAAAALI2X-1bgoyuc5cFxd4d1-kctOWcudGBf40Qtui_gRyW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=I4NFao6FV9OcHMnxkt9Gt4G%2Fb94%3D)

admin으로 실행해봄  
dbg 쓰지 말라고 했지만 써봐야징  

![Image 2](https://blog.kakaocdn.net/dna/Cwsuk/btsNkjKOk1M/AAAAAAAAAAAAAAAAAAAAALQ3ioCt3AeAaHqbyvfx52bBLBsyQlXgSoTdhsUwGwC2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=R4Lr6j%2BMoN%2BRB59wCJbU4ko27nM%3D)

일케 뜸  

![Image 3](https://blog.kakaocdn.net/dna/bfR5EC/btsNiWvz0ia/AAAAAAAAAAAAAAAAAAAAAAcbfN_Ysy37HXAIGv6o1d0zbjILe28wIa_g7e_BIx_A/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=4IVzfAfr29E%2Bfrqb3ZNT5eb4XWA%3D)

f9 계속하면 일케 뜸  

![Image 4](https://blog.kakaocdn.net/dna/bHxw3O/btsNkkbSnvs/AAAAAAAAAAAAAAAAAAAAAFKpwovyoeqB4em78SE32m9DcnI5cDX5gYCmb1rOcJbi/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zu4PI2N%2FY3PgxakUeljjic73U0Y%3D)

메모리랩 갔는데 UPX  

![Image 5](https://blog.kakaocdn.net/dna/K5Op2/btsNjyBImv0/AAAAAAAAAAAAAAAAAAAAAH3-2iBYriy5P0iTeaS8kKoSu_CshAelXWhlv9_gIlh7/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=qO6yfrSExk7y3C9Tdq5AZMzcKPU%3D)

일단 풀어줌  

![Image 6](https://blog.kakaocdn.net/dna/byMuDn/btsNjauIQx5/AAAAAAAAAAAAAAAAAAAAALVrpBLkPaQ0uf-CwGr7_TZqkin2ZPDHokDJUZgNRkhF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=qPmOquOmrD6uotuwFgNuHvt9ess%3D)

디버거 탐지를 하니까 아이다로 일단 정적 분석만 진행  

![Image 7](https://blog.kakaocdn.net/dna/bjlUe7/btsNiNzKlGF/AAAAAAAAAAAAAAAAAAAAAEvORKYf54kcIqCxB0lrtEecuyS453vPkYdehkVeceJq/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zAf3SO%2F7EAL9vFmw6Cek4r87WE8%3D)

문자열 먼저 찾았다.  

![Image 8](https://blog.kakaocdn.net/dna/bVDhRf/btsNi58aSLm/AAAAAAAAAAAAAAAAAAAAALN30ctsa_ZchwrVWQIj2my4nN2Gw1KmJ1f31mhPvPPV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=TQmZEAmulVX1mUdASfYAqwdPfmk%3D)

여기가 플래그 출력하는 함수이니까 여길 참조하는  

![Image 9](https://blog.kakaocdn.net/dna/V6Zc5/btsNjhmVADp/AAAAAAAAAAAAAAAAAAAAAH66DsdCa2ikL8u87gOElO9pibywEMtvFHjKhjxUHw7O/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=unZvb4vHVpuTsvIXofo3q4H0kNk%3D)

위 함수가 플래그 계산하는 함수일 것  
최대한 여기로 분기하도록 막 설정했더니 프로그램 자체가 안 열렸다;;  

![Image 10](https://blog.kakaocdn.net/dna/bOLxTd/btsNjCLaIPe/AAAAAAAAAAAAAAAAAAAAAFhQgMhV8haJbdbsTHGhVPcQ_gzovFeQXX6fjciKXeMI/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=IyscZyI2wdXLyrzhE6aCkGmjm%2B0%3D)

이리저리 패치했는데 모르겠어서 일단 라업 봤다  
무엇보다 startadress의 출처를 모르겠다  
라업을 봤는데 offset으로 해서 스레드 호출을 이렇게 표현한다고 하더라(아래 참고)  

![Image 11](https://blog.kakaocdn.net/dna/IVh19/btsNi33yFtl/AAAAAAAAAAAAAAAAAAAAANrgSPEz8CSEqHHYvQ7MowHVYh94LmCJwP3mvwbhDDx8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=sspu7NW7gbKdBEHSv3x5cicxDlU%3D)

라업은 디버거 중단점 설정하면서 바꿨는데 나는 아예 패치 써보려고 jz 또는 jnz로 있던 분기문을 무조건 분기인 jmp로 바꾸고 패치했다.  
아이다로 패치할 때 edit>patch program.. 들어가면 다 있다.  

![Image 12](https://blog.kakaocdn.net/dna/ykNGl/btsNiK4xYo7/AAAAAAAAAAAAAAAAAAAAAOz6b1vE2uDnidJjyZKamqi-rDwPXGjxuzGc6aUa5T_y/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=EZObdYZgikDUyd5S3l0NV9wsV3A%3D)