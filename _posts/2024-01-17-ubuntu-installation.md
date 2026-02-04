---
layout: post
title: "우분투 설치"
categories: [CS & Development]
tags: [Ubuntu, WSL, VMware]
last_modified_at: 2024-01-17
---

**리눅스 환경구축**

![리눅스 환경구축](https://blog.kakaocdn.net/dna/bSMnle/btsDz96xaQC/AAAAAAAAAAAAAAAAAAAAABdvoW808lHPo9QHKxUalATeuhVLG7CsMZHN6zbk0wbW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zs3hY7axjmzeUF4mqK4tqa9j3LM%3D)

윈도우 버전 확인

![윈도우 버전 확인](https://blog.kakaocdn.net/dna/MOcxP/btsDEigp5JP/AAAAAAAAAAAAAAAAAAAAADbEhz3el0wRZqvlSJbaAsqpNgCDfxpTCc9q_zgIckXx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=wW3YdN6iT%2FVQxww0Fu4jYUjCFJM%3D)

Powershell에 다음과 같이 입력한다.

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

![Powershell 명령어 입력](https://blog.kakaocdn.net/dna/sM0Qs/btsDBydqgbA/AAAAAAAAAAAAAAAAAAAAAM-4THtNUeFs0FmdpXBt4F9wPMsCPwiMVzvy8GwK-g5t/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=J0a3VJF4mDOz3CjmWcplUv5WAxs%3D)

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

![가상 머신 플랫폼 활성화](https://blog.kakaocdn.net/dna/9mAq0/btsDByRZ59C/AAAAAAAAAAAAAAAAAAAAAEkphb4bi2Txa7M4oU3BPMwALhFdQR00pr_OqJ1lUpth/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2B3yLX7Y8ptbpbhV2agPfSE95JT8%3D)

```
wsl --set-default-version 2
```

![WSL 기본 버전 설정](https://blog.kakaocdn.net/dna/bZFUJn/btsDDY3yB0l/AAAAAAAAAAAAAAAAAAAAANp4Crv-PUpX5pbEiPs5vbXYwy-Dl9PcPv55KmimP-hS/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2Br3unvnhBA4fGYbDniK0GBXuW78%3D)

스토어에서 Ubuntu 22.04 설치 후 간단한 설정

![Ubuntu 22.04 설치](https://blog.kakaocdn.net/dna/bnv4g7/btsDDX4EfLv/AAAAAAAAAAAAAAAAAAAAAJXQM08TnYIRyOFtWIHK1ec-0vmOPB-Z3isjdGNH1EiJ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zoiuf9g8VRGJE0Kytwnduux88bc%3D)

hello ubuntu! 출력

![hello ubuntu! 출력](https://blog.kakaocdn.net/dna/kYm52/btsDyADAY1D/AAAAAAAAAAAAAAAAAAAAAK6br0IL_henqqxm9VE6jvJDkoDjGBZfUSfTl1DNBkhX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2BIyjkSqmFxGjhH85Gv2F2IklNc0%3D)

vmware에도 우분투를 깔아주었다.