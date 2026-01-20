---
layout: post
title: "드림핵 도커 파일 구축"
categories: [Self-study, Writeup]
tags: [Docker, WSL, gdb, pwndbg]
last_modified_at: 2024-02-26
---

도커 위치와 deploy 파일 그대로 우분투에 넣어줘야 함  
[https://diary-developer.tistory.com/8](https://diary-developer.tistory.com/8)

![Docker Error](https://blog.kakaocdn.net/dna/kykTE/hyVqlUQHFN/AAAAAAAAAAAAAAAAAAAAADqDULLvz_OXpPqle8HhoarLGTomFaMBkp48f-qt24jx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xUIGug6WSNHw2hX%2Bi12AfxnzOd4%3D)

도커 데몬 running? 비슷한 오류나면 위 링크 따라하기  

도커파일이 있는 위치로 가서 아래 명령어 실행 

```shell
sudo docker build -t my_docker_image .
```

![Docker Build](https://blog.kakaocdn.net/dna/bMKnxd/btsFhgpo49V/AAAAAAAAAAAAAAAAAAAAAFXg83TElE6yQa0z3qol6-TlO2tgFyt6qQD16HZmyd8X/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FxEG40ADlekYUXckltE29DSizJk%3D)

```shell
sudo docker run -d -p 7182:7182 --name my_docker_container my_docker_image
```

![Docker Run](https://blog.kakaocdn.net/dna/A2OtW/btsFmmaCpSo/AAAAAAAAAAAAAAAAAAAAAGXqfgavrP8jZCe71v4MeKONqGTguMme6GukX3Sjedur/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=fVttF1a4ndeDgEroVF5hqaB6JyE%3D)

```shell
sudo docker exec -it my_docker_container /bin/bash
```

![Docker Exec](https://blog.kakaocdn.net/dna/pNUtS/btsFhT8DXGE/AAAAAAAAAAAAAAAAAAAAAJ2ki8s_C0EvVL32H1gW-5E2cMOwIljIIrYGKEjsG9HN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=drkfH%2FK%2BdV92VuRheBPiNvLjIjU%3D)

- gdb/pwndbg 깔기  
루트 권한으로 접속 

```shell
sudo docker exec -u root -it my_docker_container /bin/bash
```

```shell
apt-get update
apt-get install gdb -y

apt-get install -y python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
```