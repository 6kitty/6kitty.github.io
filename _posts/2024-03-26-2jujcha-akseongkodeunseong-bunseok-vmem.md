---
layout: post
title: "[2주차] 악성코드 분석 - vmem"
categories: [Digital Forensics]
tags: [악성코드, vmem, 분석, Forensics]
last_modified_at: 2024-03-26
---

![Image](https://blog.kakaocdn.net/dna/bEs2ei/btsF5VD6SKM/AAAAAAAAAAAAAAAAAAAAAKOmf_2DH54Gd5bsjHrCZv6N6Vnm25CTyMfgwovtscY6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=K5p2ltG9lJqyUUMwiYtuouPfKxM%3D)

```sql
python vol.py -f <vmem이름> windows.info.Info
```

볼라 2.6에서는 imageinfo로 프로파일 정보를 찾아야 했지만 볼라 3에서는 알아서 찾아준다. 이외의 정보가 궁금하면 위처럼 입력해주면 된다.

64bit인지.. symbols가 뭔지 kernel base 주소까지 나온다.

![Image](https://blog.kakaocdn.net/dna/qOYxH/btsF7ZldNoD/AAAAAAAAAAAAAAAAAAAAAHAtVdMKFmZ18sP-lixNaNQAauODAztMYUM1_PZ0t7JK/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WQ7zc27YMGDES5D3czp4%2FQVLRJk%3D)

![Image](https://blog.kakaocdn.net/dna/ISJsc/btsF3lDPhWa/AAAAAAAAAAAAAAAAAAAAAOcCCvH_8DZalHZre3GxMsjMZhq-8CqQCqRZv7KO3KbS/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ZydC5fQ7ljNZfSGVqD7rpLbyQmI%3D)

```sql
python vol.py -f retry.vmem windows.pstree.PsTree
```

process 목록을 보여주는 플러그인은 pstree, psscan, pslist 등이 있다. 위 실습 화면은 pstree를 사용하였다.

캡쳐한 부분은 악성코드 MariyelsTherapy에 대한 프로세스 부분이다. 언제 실행했는지.. 어떤 위치에 있는지.. 실행하고 있는 cmd까지 보여준다. 마지막 PID 5328인 프로세스가 뭔가를 실행하고 있는 거 같다.

<div data-ke-type="moreLess" data-text-less="닫기" data-text-more="더보기"><a class="btn-toggle-moreless">더보기</a>
<div class="moreless-content">
pslist: 시간순  
psscan: 오프셋 순서, 숨겨진 파일 로드 가능  
pstree: pid와 ppid의 관계를 파악 가능  
</div>
</div>

![Image](https://blog.kakaocdn.net/dna/WRdoZ/btsF6KhYWKZ/AAAAAAAAAAAAAAAAAAAAAIhK3lA7SkCoz1OwA4UHcM1zSMnBzTLfKGwfrQgCuIl2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=twsYb3bzm0pNH18dN0H%2BZEiLDPM%3D)

```shell
python vol.py -f retry.vmem windows.dlllist --pid 5504
```

위는 dlllist 플러그인이다. 여기서 사용한 pid는 악성코드 실습에서 했던 멀웨어 파일 pid이다.

dlllist를 이용하여 pid와 관련된 dll 정보를 볼 수 있다.

5504가 사용한 ppid를 보면 KERNEL32.DLL이나 KERNELBASE.dll.. OLEAUT32.dll도 있고 system32에 있는 dll이 많은 걸 알 수 있다.

![Image](https://blog.kakaocdn.net/dna/mSoes/btsF79VIx6I/AAAAAAAAAAAAAAAAAAAAACWqIkxPVr7IDpLimclMcW21ThWUxYx8AED5796g9uVk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=sYPi%2BalL9nKZqKXmqu7vmaAaSIk%3D)

```shell
python vol.py -f retry.vmem windows.dumpfiles --pid 5504
```

해당 플러그인은 process에 대한 dump를 떠주는 플러그인이다.

실습에서는 >>prodump.txt를 이용해서 텍스트 파일로 저장해주었다.

![Image](https://blog.kakaocdn.net/dna/brCayw/btsF7JQyJQq/AAAAAAAAAAAAAAAAAAAAANPF9PjRrkHqeijmlRxEI1lWXx7h-KCq3PWZIlNGbSGe/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=W4Lt9i1HbF21c0BqVa0%2BJw3%2BE3U%3D)

prodump.txt을 열면 이렇게 저장된다. >>를 이용하지 않으면 해당 내용이 cmd에서 출력된다.

다만, 해당 플러그인은 Error dumping file이라는 오류가 많이 뜨는 거 같다..

아래로 내려가면 대부분이 디스크 img 파일로 만들어진다.

![Image](https://blog.kakaocdn.net/dna/b6uiup/btsF6HyUAcz/AAAAAAAAAAAAAAAAAAAAAEJDG0LC_xUwWF48Ng-f9rebD6UljNzgYs4vfwaW7LMh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6NIh1dFcioNB%2BI4O2%2BkyKS7vHTA%3D)

```shell
python vol.py -f retry.vmem windows.cmdline
```

cmd에서 했던 작업들을 보여준다. 해당 플러그인도 출력이 많이 되기 때문에 >> 텍스트 파일로 저장해주었다.

![Image](https://blog.kakaocdn.net/dna/yW151/btsF64tUVF7/AAAAAAAAAAAAAAAAAAAAALl4j51dndENWyeZDSMnTb6PppGn_xfmuIM8MdiK1Cm_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=HULXkm3n6i1vcgMn5GBej7IpI7w%3D)

svchost.exe에서 해준 명령이 많다. svchost.exe은 서비스를 총괄하는 실행 파일이다.

svchost.exe로 위장하는 악성코드가 많다는데 해당 멀웨어도 이런 흐름인가 싶었다.

```shell
python vol.py -f retry.vmem windows.getservicesids.GetServiceSIDs
```

위 플러그인은 SID를 얻을 수 있다. SID는 session ID로 어떤 프로세스 묶음을 나타낸다.

직관적으로 SID와 service 내용을 보여준다.