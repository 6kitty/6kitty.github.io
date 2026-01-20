---
layout: post
title: "pico-rev_cipher"
categories: [SWING, Writeup]
tags: [CTF, picoCTF, reverse engineering]
last_modified_at: 2025-04-12
---

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  _BYTE inputbyte[23]; // [rsp+0h] [rbp-50h] BYREF
  char v5; // [rsp+17h] [rbp-39h]
  int v6; // [rsp+2Ch] [rbp-24h]
  FILE *output; // [rsp+30h] [rbp-20h]
  FILE *input; // [rsp+38h] [rbp-18h]
  int j; // [rsp+44h] [rbp-Ch]
  int i; // [rsp+48h] [rbp-8h]
  char v11; // [rsp+4Fh] [rbp-1h]

  input = fopen("flag.txt", "r");
  output = fopen("rev_this", "a");
  if ( !input )
    puts("No flag found, please make sure this is run on the server");
  if ( !output )
    puts("please run this on the server");
  v6 = fread(inputbyte, 0x18u, 1u, input);
  if ( v6 <= 0 )
    exit(0);
  for ( i = 0; i <= 7; ++i )
  {
    v11 = inputbyte[i];
    fputc(v11, output);
  }
  for ( j = 8; j <= 22; ++j )
  {
    v11 = inputbyte[j];
    if ( (j & 1) != 0 )
      v11 -= 2;
    else
      v11 += 5;
    fputc(v11, output);
  }
  v11 = v5;
  fputc(v5, output);
  fclose(output);
  return fclose(input);
}
```

1~8글자까지는 inputbyte 그대로  
9~23글자까지는  
j가 짝수일 때 v11+=5  
j가 홀수일 때 v11-=2  

이걸 리버싱하는 코드를 짜면 되겠군  

![rev_this output](https://blog.kakaocdn.net/dna/drgoYw/btsNkoLQPHl/AAAAAAAAAAAAAAAAAAAAAJ3e2-T2trHE7pqwjL0lENWNQ_1x6Z5RbArRz2UM4zKl/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=AkEARDB%2BP%2Br3RTNW7fWDP5zoaOo%3D)

일단 rev_this 하면 위와 같다.  

```python
revthis="picoCTF{w1{1wq85jc=2i0<}"
flag=""

for i in range(0,8):
    flag+=revthis[i]

for i in range(8, 23):
    char = revthis[i]
    if(i % 2 != 0):
        flag = flag + chr(ord(char) + 2)
    else:
        flag = flag + chr(ord(char) - 5)
print(flag)

print(flag)
```

닫는 괄호 추가해서 solve  