---
layout: post
title: "[2주차] 써니나타스 30"
categories: [Digital Forensics]
tags: [CTF, Wargame]
last_modified_at: 2024-03-25
---

![Image 1](https://blog.kakaocdn.net/dna/tCnpY/btsF2QQ1YmG/AAAAAAAAAAAAAAAAAAAAAP1rbhpUTU-maxiD10syr4vxeSa3HdAXQ96migN63Wws/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ZsTizGL2opEm7lU5DYVKGMR0P6s%3D)

일단 imageifo가 오류난다

![Image 2](https://blog.kakaocdn.net/dna/3avYP/btsF2vGrFi3/AAAAAAAAAAAAAAAAAAAAAFNdNDthcCuF1KGf045DuFR6u7HsRPUjQkzcJWQdGLpw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=qCk9zEvMmIkQtwDjl0tnhzj4wqU%3D)

소문자로 해줘야 했다

![Image 3](https://blog.kakaocdn.net/dna/q7fkC/btsF3fQHmw2/AAAAAAAAAAAAAAAAAAAAALnEoKF0ohGRvbRTQzK5d4hhh_uPIhcW7BNIpQb6fdAJ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6TviVekRAcnB4pi0IvV2tBnh7FQ%3D)

ip니까 netscan을 써야 할 거 같았다

![Image 4](https://blog.kakaocdn.net/dna/dfAQPK/btsF2Pq9e6s/AAAAAAAAAAAAAAAAAAAAAE-43unLG0Gb0szAfPxad6K6gj_br9U1qSxurs0yxGnh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=9EBvONq4XrspmBz%2F10avSBMiRRI%3D)

왼쪽 PID가 제대로 인식을 못하는 거 같아서 의심스러운 부분이지만 확실한 근거가 없어서 일단 넘겼다

![Image 5](https://blog.kakaocdn.net/dna/cIneDH/btsF18dAtu3/AAAAAAAAAAAAAAAAAAAAAHh3UFs8Dwdjxs0suX-XHTYu6Kyozt32PotCcwirFtC_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=btjsRj20BtCnfgugCxeUlmIQ7Vc%3D)

cmdline을 검색했다

![Image 6](https://blog.kakaocdn.net/dna/oejuv/btsF5Vjf2Lr/AAAAAAAAAAAAAAAAAAAAABkFhfTgRYBD2z_0LQGFAi1EHYa3iJ0b4l2p2DjQ9XpN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=dYz4m2zwhVvmXWVe0oYt5jatpRc%3D)

해당 문서 이름이 수상하다

![Image 7](https://blog.kakaocdn.net/dna/xga78/btsF5Evh2ys/AAAAAAAAAAAAAAAAAAAAAE7RaAZ2zivJ7Dj8-dtYtObIWzvdOolUfGP7Vyhia-14/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FIVDXelelQHKflAPihhgFmKu%2B%2FU%3D)

dumpfiles를 이렇게 하면 안되는 거 같다

![Image 8](https://blog.kakaocdn.net/dna/bF13NV/btsF4x4pC2s/AAAAAAAAAAAAAAAAAAAAAJU_8e0KI9CNt73ygFKtdfmdfQznZDubzIvSc8RBFL1Z/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QOvWSpGC7mZjAEiSXJRgln4eP4U%3D)

올바르게 입력해줬다 근데 이건 notepad.exe 프로세스 덤프라 버렸다

![Image 9](https://blog.kakaocdn.net/dna/l2FGx/btsF4YHz9Bz/AAAAAAAAAAAAAAAAAAAAADYYdd1IbVMO8rnRL3-qhQ2UiZsREn4HKeQ0sIC71Qb9/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BruNymcX8rfK58ZlpqP9PAk0UMQ%3D)

그냥 filescan을 하면 너무 많이 나와서 파이프로 findstr txt검색했더니

SecreetDocume7.txt의 오프셋이 나온다

![Image 10](https://blog.kakaocdn.net/dna/wE469/btsF6hGnAqq/AAAAAAAAAAAAAAAAAAAAAFPMIyjnFVpuDZ8d83fXQQQ_yZZLwxhoqMB528nIqLRf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Lg4Vz9wnz28SP3Nossr519fyzJ4%3D)

오프셋을 이용해서 파일을 덤프해준다. 확장자가 txt가 아니라서 어떻게 하나 했는데 strings.exe라는 툴을 설치하면 된다

![Image 11](https://blog.kakaocdn.net/dna/bnjqM3/btsF5Ty0rax/AAAAAAAAAAAAAAAAAAAAAHlobkAVWjrsh3aJaNGtAoXEaP-UWa4Iq1vzMopDqpyR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ipkpFgxQhfrMQjAw93uWxGt6BxA%3D)

위처럼 txt 변환해준다

![Image 12](https://blog.kakaocdn.net/dna/HZ68x/btsF5vyvICo/AAAAAAAAAAAAAAAAAAAAAKrIY4d1ORrWppXRq86-k9pQENo98e7tUirkL25V-vLQ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=S%2BcHaYzRRFbHGrhZbGUQ3ZUc09o%3D)

Key가 올바르게 들어있다!

2번,3번은 풀었는데 1번 김장군 PC의 IP주소가 맞는지 의심만 가고 확실한 근거가 없어서 라업을 참고하기로 했다.

![Image 13](https://blog.kakaocdn.net/dna/bOdo3Y/btsF3lQLy0K/AAAAAAAAAAAAAAAAAAAAACgWxh2FgngUTAL_iapjZzMedmwPEDO3zOT3oDwU1aKG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=DbMsNbPviszrUDVXExFmNp6Wscw%3D)

netscan을 돌려보면 나오는 ip 주소는 아래 3개이다.

127.0.0.1  
0.0.0.0  
192.168.197.138  

이중에서 1번째는 localhost이고, 2번째 0은 지정하는 ip없이 모든 ip를 의미하므로 김장군 IP주소는 192.168.197.138이라고 한다.

192.168.197.138SecreetDocume7.txt4rmy_4irforce_N4vy

![Image 14](https://blog.kakaocdn.net/dna/bKI4Fk/btsF3VK6vJ2/AAAAAAAAAAAAAAAAAAAAAEClTV7tsQy-MPrqIq9WzdTcCEF-6IdD-Z-Jiz-X-B7h/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=PjAk8z2Kdf%2Fpd%2Fnp0yCKiaZ%2BJqw%3D)

여기서 소문자로 바꿔줘서 c152e3fb5a6882563231b00f21a8ed5f이 답이다