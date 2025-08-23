---
layout: post
title: "mac m3 포너블 환경 설정"
categories:
  - Pwnable
  - +
tags:
  - mac
  - pwn
last_modified_at: 2025-08-21
---

포스트라고 하기엔 대충이지만 
1. UTM 설치 
2. ISO 다운 (필요한 아키텍처로) 
3. UTM에서 가상환경 생성 후 **에뮬레이터** 로 설치

이러면 이제 엄청나게 느리지만 amd 우분투를 하나 만들 수 있다.
와 잠만 진짜 충격적으로 느리네.  
VScode로 ssh 연결되나 함 보자. 

vm에 sudo apt-get install openssh-server <br>
응 실패다 dkpg 오류 뜨는데.. 너무 느려서 트러블슈팅 하는 것보다 server로 다시 깔았다. 아예 ssh로만 써야겠다. 

[참고한 글 : M1M2-환경에서-Pwnable-환경-구성하기](https://rasser.tistory.com/entry/M1M2-환경에서-Pwnable-환경-구성하기)

1. emulator로 ubuntu server 설치 
2. 초기 config 설정 
3. Openssh server 설치 
4. 설치 다 되면 재부팅을 해야함 -> **이때 iso 파일 꺼내기** 
5. Vscode에 remote ssh extension 설치 
![익스텐션 설치](../../../assets/images/250822_02.png)
6. 연결~

하면 되는데 우분투 서버 설치가 너무 오래 걸린다.. 
다 설치하면 마저 쓰는 걸로 

자고 일어나서 다시 실행해보니까 됐다. 포너블 설치할 스크립트 하나 두자

[chedahub/pwn_set/blob/main/pwnset20.sh](https://github.com/chedahub/pwn_set/blob/main/pwnset20.sh)

여기서 코드 가지고 왔다. 
```sh
#!/bin/bash
export LC_CTYPE=en_US.UTF-8
export LC_ALL=C.UTF-8
source ~/.bashrc

cd ~
sudo apt update -y
sudo apt-get update -y

# vim
sudo apt-get install vim -y

# gdb, gcc
sudo apt-get install gdb -y
sudo apt-get install gcc -y
sudo apt-get install gcc-multilib g++-multilib -y
echo "set disassembly-flavor intel" >> ~/.gdbinit

# netcat
sudo apt-get install netcat -y

# git
sudo apt-get install git -y

# python3, pwntools
sudo apt-get install python3 python3-pip python3-dev libssl-dev libffi-dev build-essential -y
sudo apt-get install libc6-i386 libc6-dbg make -y
python3 -m pip install --upgrade pip
python3 -m pip install --upgrade pwntools
sudo apt-get install libcapstone-dev -y # to rop

# pwndbg
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh
cd ..

# checksec
git clone https://github.com/slimm609/checksec.sh
sudo cp checksec.sh/checksec /usr/local/bin/

# ROPgadget
sudo pip3 install ropgadget

# OneGadget
sudo apt-get install ruby-full -y
sudo gem install one_gadget

# seccomp-tools
sudo gem install seccomp-tools

# pwninit
sudo apt install openssl
sudo apt install pkg-config
sudo apt-get install cargo -y
sudo apt-get install -y liblzma-dev
cargo install pwninit
sudo cp ~/.cargo/bin/pwninit /usr/local/bin/

# visual studio code
sudo apt-get install curl -y
sudo sh -c 'curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > /etc/apt/trusted.gpg.d/microsoft.gpg'
sudo sh -c 'echo "deb [secarch=amd64] https://packages.microsoft.com/repos/vscode stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code -y
```
![스크립트 실행](../../../assets/images/250823_01.png)
이럼 또 한참 걸린다(이런미친) 

자 오류가 떴다. 고치자 ㅋ 

```bash
$sudo rm /var/lib/apt/lists/lock
$sudo rm /var/cache/apt/archives/lock
$sudo rm /var/lib/dpkg/lcok*

$sudo dpkg --configure -a
$sudo apt update
```

이랬더니 오류 났다. nano로 주석 처리 해야 된대서 

```bash 
sudo nano /etc/apt/sources.list
```

이러면 nano 창이 뜰 텐데 `deb file:/cdrom`으로 시작하는 줄을 주석처리를 한다(#으로 처리하면 된다) 이후에 다시 시스템 업데이트를 하자. 

```bash 
sudo apt update
sudo apt upgrade -y
```

이러고 다시 스크립트 실행 ㄱㄱ 


설치는 됐는데 1804도 일단 필요할 거 같아서 얘는 도커로 넣으려고 한다. '

[https://github.com/superkojiman/pwnbox](https://github.com/superkojiman/pwnbox)

이걸로 가지고 왔다. 

```bash
git clone https://github.com/superkojiman/pwnbox
sudo apt install jq 
sudo ./run.sh my_ctf
```

하고 열심히 기다리고 다시 또 ip addr 가져와서 ssh 연결해주면 된다. 

재접속 및 삭제는 아래 

```bash 
sudo docker start my_ctf && ./my_ctf-attach.sh
sudo ./my_ctf-stop.sh
```
