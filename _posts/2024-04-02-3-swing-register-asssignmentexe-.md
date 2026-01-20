---
layout: post
title: "[3주차] swing_register_asssignment.exe 분석"
categories: [SWING, Writeup, Self-study]
tags: []
last_modified_at: 2024-04-02
---

swing_register_asssignment에서 다른 모듈 호출하는 명령 목록을 가져온다.  
_cexit을 찾아서 주소로 간다.  

![image1](https://blog.kakaocdn.net/dna/byJpH8/btsGhX9TlQq/AAAAAAAAAAAAAAAAAAAAAJfUbfeXaSS__QRxSHEVyUXqZjX3Qt3tETxdLQZvxzpN/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QelDpk9t1HkP2Ua%2B2Mg3ZS4uQWg%3D)

해당 _cexit 위에 있는 call이 메인 함수인 거 같다. 해당 callee를 더블 클릭해본다.  

![image2](https://blog.kakaocdn.net/dna/budoKE/btsGjOcSexQ/AAAAAAAAAAAAAAAAAAAAACZ73gyB4cf89Rs4RC-zH6BC-k8ftSmbFf7SoP9cWNck/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=dbYIp09njjISHS7ZXpNySo2zvcE%3D)

puts 같은 출력 함수와 문자열에 있는 걸 보아 main이 맞는 거 같다.  

![image3](https://blog.kakaocdn.net/dna/YjrBT/btsGic6LDnB/AAAAAAAAAAAAAAAAAAAAALm0khMiAfNSqxBRCHFtjJLBGp8Z-MYppA9JPHqC97Ns/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=c%2F5mJflwEsEZ5g9Qtzdcuau2f8g%3D)

중단점을 설정하고 해당 부분까지 f9로 와준다.  

![image4](https://blog.kakaocdn.net/dna/ceDIvE/btsGgZ8cmTU/AAAAAAAAAAAAAAAAAAAAAOkEdGAm8swKfw_yJhgoB9yazDyPRCkDHkk6JEC5y1n4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=DQDgHCzcsbHZCrohIl%2B2nIdv2G8%3D)

맨 위 Hello SWING 문자열 명령을 분석해보면  
swing_register_as~(짤렸다)부분 주소를 hex로 보면  

![image5](https://blog.kakaocdn.net/dna/cKDnjQ/btsGidYW7jt/AAAAAAAAAAAAAAAAAAAAAAMAYM62MZorxTVEKcsTNlranmMfbj9edbX1mlAJC95q/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=03RQdm6Cr8F0tUGaFTY82iW2%2FQk%3D)

일단 주소가 00404052임을 알 수 있다. 왼쪽 ascii 부분에 Hello SWING!이 맞게 써져 있다.  
mov로 esp에 옮겨주는데  

![image6](https://blog.kakaocdn.net/dna/c9YHg5/btsGjNdZVfp/AAAAAAAAAAAAAAAAAAAAAOBR4v5d3IqI1paMVaFat40pGF9n2NWfiO87WbPNLT45/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=s%2BJuHa7iZNXtv23pBwXs9Rc4zXE%3D)

원래 이랬던 esp가 명령을 실행하면  

![image7](https://blog.kakaocdn.net/dna/b6jY3Q/btsGjPit6QS/AAAAAAAAAAAAAAAAAAAAAJnHZi7Dr2wYe4xQtDvMkGBp-Is3Uvyk3RswXfVPy9TY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=zNJ866Ry0Ffy99J47hmCczjemhI%3D)

이렇게 변한다. 맨 앞 4bytes만 리틀 엔디안으로 읽으면 00040452  
아까 본 Hello SWING!이 담겨 있는 주소이다.  

출력 명령을 할 차례이다.  

![image8](https://blog.kakaocdn.net/dna/dxM8jU/btsGiA7srYk/AAAAAAAAAAAAAAAAAAAAAJEzneoCEvtLjih6pw21YE0VP0BUFzvuIEgOz3tximZi/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=y3pBb0ZQRSsca7OKQw0iV9VHu9w%3D)

디버깅 창에 문자열이 뜬 걸 알 수 있다.  

![image9](https://blog.kakaocdn.net/dna/ptbK3/btsGh50ZRJ1/AAAAAAAAAAAAAAAAAAAAACtY8apKpHZPqbUPw6oTKZ5IykN8KuqA1PgB9NtnDhzz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=yS%2FdBGFsepn4aR%2B2yK%2Fem5Xn53A%3D)

다음 문자열은 call로 다른 함수를 호출해온다.  

![image10](https://blog.kakaocdn.net/dna/nxLPT/btsGh57KIj7/AAAAAAAAAAAAAAAAAAAAAASr-WRVUJ3xw5rLqQ6SFWXs7SWrG2YIC413SDMa4rPG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2B3Fmm9nO5vOXr8CjcvxVuFOwVto%3D)

f7로 진입해보면 ebp가 나온다.  

![image11](https://blog.kakaocdn.net/dna/bx0gxe/btsGh5s7CMc/AAAAAAAAAAAAAAAAAAAAAOwPldk9gXeCBgvby8gBrEhuRCnfgX1Qz-mmheCQE_et/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=W7CLtJUtvey7n94X6CsDksM1Xic%3D)

들어간 직후 스택 상단에 들어간 00401643값은  

![image12](https://blog.kakaocdn.net/dna/ckbCtX/btsGiBrLx7y/AAAAAAAAAAAAAAAAAAAAAAAoY9xWPNwyBsn06ryLDQjX3jb8C9Cr-95yBq4xK7Of/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=5UAI3j0YA8Uz%2BaGdd8ZNVjfvhxI%3D)

아까 호출했던 명령어 다음 주소이다.  

![image13](https://blog.kakaocdn.net/dna/WH5G3/btsGhVc95uR/AAAAAAAAAAAAAAAAAAAAAJeRKulhnehUd97LnbezY_bHmgjQ4OOps_-p5TOWYulL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=OVQ%2B6RRlQlh8D1PpwL26TuQ%2F5lk%3D)

위 callee를 실행하면 eax가 7이다. 어떻게 7이라는 값이 나왔을까  

![image14](https://blog.kakaocdn.net/dna/k00Xs/btsGhplVB6I/AAAAAAAAAAAAAAAAAAAAADoMCKArZBweat_ghgM5Ua8PZM3Vr4Zwub1xYbi185rt/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=pK9%2BqvokFP4Xs683Yquj27GX7cM%3D)

mov eax, dword ptr ss:[esp+18] 에서 esp+18에 저장된 값이다.  

![image15](https://blog.kakaocdn.net/dna/bbLxXd/btsGhnayXJr/AAAAAAAAAAAAAAAAAAAAADmr8q41kRZGp1AmzmMTRbVzGfrtUegEkPvPLjfYWfU1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=TGba97bPAinH1rCuWfDWEv6N1dU%3D)

mov eax, dword ptr ss:[esp+1C] 에서 esp+1C에 저장된 값이다.  

![image16](https://blog.kakaocdn.net/dna/bd6jvH/btsGjKB7md7/AAAAAAAAAAAAAAAAAAAAAJ3smc6t1pgnTPQt9FqHvAZ646J060q9DeAZTQA4z2Mx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=YLT0M8LZGER%2BOIAllMiiGyFBnak%3D)

해당 값들은 eax에 옮겼다가 스택에 넣어줬기 때문에 스택 상황은 위와 같다.  
마지막 eax에는 3이 있다.  

![image17](https://blog.kakaocdn.net/dna/r95pP/btsGiZsVReI/AAAAAAAAAAAAAAAAAAAAAPv94KbxlXw_KvPtQUM99rP5_M4u4retkbMn7-cC0-10/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=sQp9rRAr%2Bl7uWwMHGtQpwZF5xdg%3D)

callee에 들어간 후 어셈블리이다.  
ebp+8과 ebp+C에 있는 4,3을 가져와서 add 해주므로 값은 7이다.  

![image18](https://blog.kakaocdn.net/dna/c23qF2/btsGizHucMM/AAAAAAAAAAAAAAAAAAAAAN4okxiDIFmTimu6Rbw5QA77WsMwKuhPTIyl9-kstEMV/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=bGxNhs7T2%2Bzq4%2B1QsirWjyWqY5w%3D)

eax값이 7로 반환된 것을 확인할 수 있다.  

![image19](https://blog.kakaocdn.net/dna/cmWO6l/btsGh6FDCRD/AAAAAAAAAAAAAAAAAAAAAIix1H3XK8xnG2eNrm7yuN30dnh_x15i44q0Mf4-D-Nz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=auOLtz3WiX3kPkmolbGpn3rkdUc%3D)

print 함수 실행  

![image20](https://blog.kakaocdn.net/dna/OdNWj/btsGiX9a5b4/AAAAAAAAAAAAAAAAAAAAABDd68d3uKgZX9LAhfrM1VhKphhCphlKDtwUYkdg1Iri/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=LBr%2FoJ92wrX7%2Fm4MxMmas3JPtzo%3D)

디버깅 화면에 7로 알맞게 출력되었다.  

![image21](https://blog.kakaocdn.net/dna/coxno4/btsGjnUbBjN/AAAAAAAAAAAAAAAAAAAAAHfF5ZpMookXM3SGzLCU-u0bdZHSdu86n_iUP4E1OOtd/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Ze%2BCGv162282tK44gtXctDxSXVI%3D)

다음 callee에 진입해보면  

![image22](https://blog.kakaocdn.net/dna/k9hVP/btsGiQWBnDk/AAAAAAAAAAAAAAAAAAAAAFgXXuyEH4ZaOFJ4eupp_4r1jzVKiMfQS7SPTxK5i4hQ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=HfkG4BtLUIRyYa4%2Fvx5a0Ij3B5U%3D)

위와 같다  

![image23](https://blog.kakaocdn.net/dna/bm9LuQ/btsGhmPJTRz/AAAAAAAAAAAAAAAAAAAAAKEyDZ94Ahljsux29WyabvsXff0UlLKlmmQO9i7G5RlY/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=04Sp%2FLfeFYjc5B%2F5OK9g3eysxDY%3D)

아까와 동일하게 caller 다음 주소가 stack에 쌓인다.  

![image24](https://blog.kakaocdn.net/dna/AmlA8/btsGhVYza6r/AAAAAAAAAAAAAAAAAAAAAOw7gRLSQePEusQBHMM7Ns-JPpN_uDKu-kZN-9GKNC8M/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=COGd5HksRQ0O1SW2Ioc9i%2BZcSkw%3D)

분기문이 있다. ebp-C가 어떤 flag의 역할을 해서 계속 대소 연산을 해주는 거 같다.  

![image25](https://blog.kakaocdn.net/dna/DKvyj/btsGicsemXp/AAAAAAAAAAAAAAAAAAAAAOjEikuzAnaLPjMFBpNcD8amb5M3oRu263csWizEjQct/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rI3f7cTsw6UpIlcbAKPKlKkPUj8%3D)

jle less일 때 분기해주는 거 같다.  
바로 직전 명령어가 4와 비교이기 때문에  
if([ebp-C]<4)  
    jmp  
인 거 같다.  

![image26](https://blog.kakaocdn.net/dna/mYk6G/btsGhTNhV8G/AAAAAAAAAAAAAAAAAAAAANNLXrt3tWgUIUVrm4133Z2OETrItt2vv8giHqvywlGB/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WXhFAq6uExp4ozeHifBtraMz1vE%3D)

![image27](https://blog.kakaocdn.net/dna/bMBN0o/btsGh4VnRgI/AAAAAAAAAAAAAAAAAAAAAEfO743LOKBpG0BOMYouVmphy5ulYAuZwHVex8-1mf_j/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Vlx%2FX4yqY38226ZSnW5QGRK8KXA%3D)

I am Main %d을 출력해준다. 이때 %d는 eax 값이다.  

![image28](https://blog.kakaocdn.net/dna/obP6b/btsGhmoE3uq/AAAAAAAAAAAAAAAAAAAAAF0mF_fUg_a8oMikcrc7iYMtIwHyjTJ2r2qsg8Iiou6H/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=WkRMXkp9%2FKw2lyR7uvnHK3meb0s%3D)

디버깅 화면이다.  

![image29](https://blog.kakaocdn.net/dna/ciZf8b/btsGi1X7pGl/AAAAAAAAAAAAAAAAAAAAABRT1s9K_n-3cHIGe7kyv8tx6Tpol6qKQn6bDa2s-Tdr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=43iKAK3yLCle6vBjfIrLW14LFU4%3D)

eax가 4가 될 때까지 루프 돌렸다. eax가 4가 되면 jle가 분기되지 않고  

![image30](https://blog.kakaocdn.net/dna/bdNrY0/btsGiBrN1q4/AAAAAAAAAAAAAAAAAAAAAK2GGX1rUKwSaKiMcOJwSb1byRZtvxpp3DrdxBnqvgM7/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=BEcd3dPYLbFvNxGAEFnEVYME5N0%3D)

다음으로 넘어가는 걸 알 수 있다.  
이후는 return 0을 위한 코드로 종료를 해주는 거 같다.