---
layout: post
title: "just read memory"
categories: [System Hacking]
tags: [CTF, Memory, Analysis]
last_modified_at: 2025-01-16
---

그냥 읽으라고 한다 

설명에 script 쓰라고 하는데.. 안 써봐서 약간의 노가다를 각오하고 풀고 라업 참고해서 배워보려한다(변명하자면 script 예시가 잘 안 보여서 어떻게 쓰는지 숙지가 안됐다)  

![Image](https://blog.kakaocdn.net/dna/bAnADM/btsLMnJwI1N/AAAAAAAAAAAAAAAAAAAAAP-bjdmB53GdEC1WQiMTXt9bJrMwUPfGCA5UgeeIqlmh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=U%2Fsv1TXDZnmZus3R44wsiadS04E%3D)

어떤 연쇄적인 리스트에 대해 malloc 해주고 append_list하는 코드이다. 

append_list 함수를 읽어보자. 
```cpp
__int64 unknown_func1()
{
  return (unsigned int)cnt;
}
```
cnt를 return한다 
```cpp
__int64 unknown_func2()
{
  int v1; // [rsp+0h] [rbp-4h]

  return (unsigned int)(9 * (3 * (v1 + 5) - 7));
}
```
v1에 대한 계산값을 출력한다. 
```cpp
__int64 generate_flag()
{
  unsigned int v1; // [rsp+0h] [rbp-4h]

  cnt = v1;
  return v1 % 0x4A + 48;
}
```
v1에 대한 계산값을 출력한다. 
```cpp
QWORD *append_list()
{
  _QWORD *result; // rax
  _QWORD *v1; // [rsp+10h] [rbp-10h]
  char flag; // [rsp+1Fh] [rbp-1h]

  unknown_func1();
  unknown_func2();
  flag = generate_flag();
  v1 = malloc(0x10uLL);
  *(_BYTE *)v1 = flag;
  v1[1] = 0LL;
  if ( tail_node )
    *(_QWORD *)(tail_node + 8) = v1;
  result = v1;
  tail_node = (__int64)v1;
  return result;
}
```
위에서 봤던 generate_flag는 v1에 저장된다. malloc으로 할당되는 v1을 동적분석하며 관찰해봐야 한다. 

v1으로 받는 값들을 64번.. 관찰해보면 된다.  

![Image](https://blog.kakaocdn.net/dna/kRqXb/btsLMoaOeuZ/AAAAAAAAAAAAAAAAAAAAAOTDmP7HHY20sUDsyHM7H7nxCH2kuj1q0vIjk8UbUTMW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=q8%2BDf7RTwTRnOalmf4FeGbkoniA%3D)

함수 BP 걸고 내부로 들어가보자  

![Image](https://blog.kakaocdn.net/dna/CFZHx/btsLMDZJiDO/AAAAAAAAAAAAAAAAAAAAAKxrAQpxiCPXuDgdorRJXl6frRY240n8KGiAEPeUjEYB/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=oENKw7j1NwKEf9ZXqhs%2BlLexP2Q%3D)

+72에 mov rax, QW rbp-0x10로 flag를 v1(rax)으로 옮기는 거 같다. 그 다음 줄은 76에 bp를 걸어보자 
```cpp
b *main+76
```

![Image](https://blog.kakaocdn.net/dna/dwz3Li/btsLMYvXBUF/AAAAAAAAAAAAAAAAAAAAALwFLgaNxjFHxiBu4sc5jTE3xdPbTHTyLsVw5OMASmQf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=jk427VBMx2XEnWree0amEunWi0I%3D)

rax값은 0x4d이다. 하나씩 모아보자.. 

한 16번쯤 하면 공부 열심히 해야겠다는 생각이 든다.  

![Image](https://blog.kakaocdn.net/dna/lFoXt/btsLL6Bgd5v/AAAAAAAAAAAAAAAAAAAAAAGXJyDVbacMXkruE25zgI9mKIbz9aFaHAD2pL5SAzJX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rMl6e12nsrpR5phW%2F%2Fj9HXVrBd0%3D)

라업 코드를 보니 아래와 같았다. 주석은 내가 추가하면서 코드 이해했다. 
```python
# gdb -q -x script1.py
# gdb 라이브러리 
import gdb

ge = gdb.execute
gp = gdb.parse_and_eval

# gdb 실행 
ge("file ./main")

# gdb.Breakpoint로 중단점 설정 
class check_generated_flag(gdb.Breakpoint):
    def __init__(self, address):
        super(check_generated_flag, self).__init__(spec=f"*{address}", type=gdb.BP_BREAKPOINT)
        #cnt가 카운트 개수인 거 같다. ida 분석하면서도 있었던 변수 
        self.hit_cnt = 0
        self.flag = ""

    def stop(self):
    	#rax값을 읽는다. 0xff를 해서 ax만 남기는 거 같다. 
        ax = int(gp("$rax")) & 0xFF
        #flag에 ascii값 추가 
        self.flag += chr(ax)

        self.hit_cnt += 1
        #count 64개 채웠으면 다 찾은 거니까 stop 
        if self.hit_cnt >= 64:
            print("input:", self.flag)
        return False

# 여기 주소 뭐지 -> 아래서 후술 
pie_base = 0x555555554000
check_generated_flag(pie_base + 0x149F)

ge("run <<< asdf")
exit()
```
pie_base에 +0x149f(오프셋이다)는 아래 주소이다.  

![Image](https://blog.kakaocdn.net/dna/FIRjA/btsLQDj2kjz/AAAAAAAAAAAAAAAAAAAAAIOxo-ebt9NmUXOro8x_MJx21CSdzsKnSh618mlc2TZT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=59jCWlbQHy3ZSes%2BpGxPnx7n53s%3D)

![Image](https://blog.kakaocdn.net/dna/djC3jX/btsLQDqOuuZ/AAAAAAAAAAAAAAAAAAAAANhmPzxBnX7udsuY7OLpwkmJFCCB5pt63FPxVxF7n8n5/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=LfJPMHcU6VTrwleaF15yztcDhiY%3D)

generate_flag 바로 이후 명령이다.  

gdb로 코드영역 pie base 주소는 code라고 적어주면 된다.