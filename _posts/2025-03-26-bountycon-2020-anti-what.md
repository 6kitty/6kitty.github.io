---
layout: post
title: "BountyCon 2020 - Anti What"
categories: [System Hacking]
tags: [CTF, BountyCon, ptrace, reverse engineering]
last_modified_at: 2025-03-26
---

![Image 1](https://blog.kakaocdn.net/dna/bqCZtr/btsMT6ffkZb/AAAAAAAAAAAAAAAAAAAAAMw7IzVsukDrnPv7alPgT83LIEDt83EzciHO3o7fBb65/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=baurNgFpDAn9o0fyrPxc4KH5mRo%3D)

안 열림

![Image 2](https://blog.kakaocdn.net/dna/vrWkH/btsMVsV0j8h/AAAAAAAAAAAAAAAAAAAAAL-13TzXGTvW-7DQQUB5oFpLxkvLvK4_YmPJIw52OkqA/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=2rrKD9%2Fqaq%2BuEve1vyjZHqmDZHY%3D)

기드라 굿

```cpp
int main(undefined8 param_1,undefined8 *param_2)
{
  int iVar1;
  long lVar2;
  long in_FS_OFFSET;
  int local_15c;
  termios local_158;
  long local_10;

  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  local_15c = 0;
  lVar2 = ptrace(PTRACE_TRACEME,0,1,0);
  if (lVar2 == 0) {
    local_15c = 2;
  }
  lVar2 = ptrace(PTRACE_TRACEME,0,1,0);
  if (lVar2 == -1) {
    local_15c = local_15c * 3;
  }
  if (local_15c == 6) {
    RC4_set_key((RC4_KEY *)&stack0xfffffffffffffee8,0x32,key);
    RC4((RC4_KEY *)&stack0xfffffffffffffee8,0x4e,(uchar *)&ptext,(uchar *)&ptext);
    runPayload();
    puts("Press any key to quit...");
    tcgetattr(0,&local_158);
    local_158.c_lflag = local_158.c_lflag & 0xfffffffd;
    local_158.c_cc[6] = '\x01';
    local_158.c_cc[5] = '\0';
    tcsetattr(0,0,&local_158);
    getchar();
    iVar1 = 0;
  }
  else {
    iVar1 = unlink((char *)*param_2);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return iVar1;
}
```

main을 보면 위와 같다. 

local_15c를 6으로 만들면 뭔가 개많이 실행이 됨 

근데 local_15c는 ptrace의 반환값인 lVar2 값에 의해 조정 가능 

-----> ptrace 우회가 필요!!!

1. ==을 !=으로 바꾸기 

![Image 3](https://blog.kakaocdn.net/dna/b6D9g9/btsMWu6z1JP/AAAAAAAAAAAAAAAAAAAAAEEnD4niMHDct62FrvPrih0BWUhDNek7gVhIaJ4hAJB2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=oxtKO36IoL5u33wm%2FBSxWg%2B8KeE%3D)

if를 누르면 그에 해당하는 어셈블리 코드가 좌측에 나옴 

![Image 4](https://blog.kakaocdn.net/dna/WpSMh/btsMURIKZhP/AAAAAAAAAAAAAAAAAAAAAOBshMoJ5srl10jXilpKxr4GPLaB3GOHizul43vMzOn3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ekiZELCcKgG4gBJ7wYxGxgP3%2FPc%3D)

74를 75로 바꿔주면 JZ가 JNZ로 바뀜 

2. if부분을 통으로 0x90(NOP)으로 채우기 

![Image 5](https://blog.kakaocdn.net/dna/bPOroM/btsMWXaqeHs/AAAAAAAAAAAAAAAAAAAAAE-jvi89YwPHqdoa1nF6wegB9Vr3AFQrh3zGv6tgKOXC/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=cynVtjNV68TOTlat1sKRKdNOtUc%3D)

3. 기드라 저장이 힘들어서 아이다로 어셈블함 

![Image 6](https://blog.kakaocdn.net/dna/dfaD0r/btsMVyiFkRH/AAAAAAAAAAAAAAAAAAAAAP40Iu_g2cEb4-s-IqL9xmmzLVZL5WkfC46c4tWhuQQi/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=YaYk4qgx8VHn4lHETJryHjTYRWw%3D)

저장하고 ubuntu에서 실행 

일단 뭔가 runPayload까지 실행해보고 싶어서 

![Image 7](https://blog.kakaocdn.net/dna/z2fyr/btsMVcts7ga/AAAAAAAAAAAAAAAAAAAAACkrnrNB0MqMYfFrW5OXDrROgZ8vSZjDHh5tvaky99mR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7KkRVk2NhEH6TlEC3LSxJQhPSDk%3D)

해봤는데 바로 플래그가 나옴 

...그치만 일단 설명을 해야하니까 

![Image 8](https://blog.kakaocdn.net/dna/3QoAG/btsMVWDlafU/AAAAAAAAAAAAAAAAAAAAAOQpZ8gnECu86umMfxNniTHsZR39m9ekpEViapEa-u8G/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=aN5K3iwniaNSq35hWyKyquBR22g%3D)

runPayload() 함수를 보자 v1에 malloc으로 메모리를 할당하고 unpackPayload를 함 

할당된 곳에 배열과 ptext값을 넣고 있음 + runPayload 하기 전 ptext는 RC4구문을 거침 

1. 어셈블로 ptrace bypass하기 
2. RC4로 돌린 ptext 구하고 저장된 배열 조립해서 result 구하기