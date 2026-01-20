---
layout: post
title: "[UIUCTF2024] rev-Summarize"
categories: [SWING, Writeup, Self-study]
tags: [CTF, Reverse Engineering, Python, Z3]
last_modified_at: 2024-07-23
---

9자리 양의정수 6개 찾으면 끝남  
sub_40137B를 먼저 보자  

```cpp
_BOOL8 __fastcall sub_40137B(
    unsigned int a1,
    unsigned int a2,
    unsigned int a3,
    unsigned int a4,
    unsigned int a5,
    unsigned int a6)
{
  unsigned int v7; // eax
  unsigned int v8; // ebx
  unsigned int v9; // eax
  unsigned int v10; // ebx
  unsigned int v11; // eax
  unsigned int v12; // eax
  unsigned int v18; // [rsp+20h] [rbp-30h]
  unsigned int v19; // [rsp+24h] [rbp-2Ch]
  unsigned int v20; // [rsp+28h] [rbp-28h]
  unsigned int v21; // [rsp+2Ch] [rbp-24h]
  unsigned int v22; // [rsp+30h] [rbp-20h]
  unsigned int v23; // [rsp+34h] [rbp-1Ch]
  unsigned int v24; // [rsp+38h] [rbp-18h]
  unsigned int v25; // [rsp+3Ch] [rbp-14h]

  if ( a1 <= 0x5F5E100 || a2 <= 0x5F5E100 || a3 <= 0x5F5E100 || a4 <= 0x5F5E100 || a5 <= 0x5F5E100 || a6 <= 0x5F5E100 )
    return 0LL;
  if ( a1 > 0x3B9AC9FF || a2 > 0x3B9AC9FF || a3 > 0x3B9AC9FF || a4 > 0x3B9AC9FF || a5 > 0x3B9AC9FF || a6 > 0x3B9AC9FF )
    return 0LL;
  v7 = sub_4016D8(a1, a2);
  v18 = (unsigned int)sub_40163D(v7, a3) % 0x10AE961;
  v19 = (unsigned int)sub_40163D(a1, a2) % 0x1093A1D;
  v8 = sub_4016FE(2LL, a2);
  v9 = sub_4016FE(3LL, a1);
  v10 = sub_4016D8(v9, v8);
  v20 = v10 % (unsigned int)sub_40174A(a1, a4);
  v11 = sub_40163D(a3, a1);
  v21 = (unsigned int)sub_4017A9(a2, v11) % 0x6E22;
  v22 = (unsigned int)sub_40163D(a2, a4) % a1;
  v12 = sub_40163D(a4, a6);
  v23 = (unsigned int)sub_40174A(a3, v12) % 0x1CE628;
  v24 = (unsigned int)sub_4016D8(a5, a6) % 0x1172502;
  v25 = (unsigned int)sub_40163D(a5, a6) % 0x2E16F83;
  return v18 == 4139449
      && v19 == 9166034
      && v20 == 556569677
      && v21 == 12734
      && v22 == 540591164
      && v23 == 1279714
      && v24 == 17026895
      && v25 == 23769303;
}
```

하 수식 개많네  
음 일단 먼저 구할 수 있는 a'n'을 찾아야 함  

![Image](https://blog.kakaocdn.net/dna/ntY7z/btsIkqhMnov/AAAAAAAAAAAAAAAAAAAAAEYw8WKD069xtFX2bHeMvS8ytxgLJgsFdIjpM84dGNt5/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=5yKkNx0kHhPlTVSlKHELi%2Bvh5xE%3D)

0x3b9ac9ff < a1 <= 0x5F5E100, 0x3b9ac9ff < a2 <= 0x5F5E100일 때 sub_40163d의 최대 최소값  
200000000, 1999999998  

v19=0x1093a1d*n+9166034일 때, n의 범위  

![Image](https://blog.kakaocdn.net/dna/br5nZV/btsIKgxZPZq/AAAAAAAAAAAAAAAAAAAAADoPdaMwOED9uQsdYbPvTkTnARHHswnH68icDc3FKWjK/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=qn3OTuBGTxLn7VDyyG529f%2Fv9fs%3D)

```python
# Define the function based on the provided code
def sub_4016FE(a1, a2):
    v4 = 0
    v5 = 0
    while a1:
        v4 += (a1 & 1) * (a2 << v5)
        a1 >>= 1
        v5 += 1
    return v4

# Define the range for the inputs based on the problem statement
a1 = 3
a2_min = 0x3b9ac9ff  # 1,000,000,000
a2_max = 0x5F5E100   # 100,000,000

# Calculate the function output for the minimum and maximum a2 values
min_a2 = 0x3b9ac9ff
max_a2 = 0x5F5E100

# Calculate the minimum and maximum return values
min_return_value = sub_4016FE(a1, min_a2)
max_return_value = sub_4016FE(a1, max_a2)

print(f"Minimum return value: {min_return_value}")
print(f"Maximum return value: {max_return_value}")
```

200000000 < v8 <= 1999999998  
300000000 < v9 <= 2999999997  

나보고 어드카라고 안된다고  

hint가 9자리 수인듯 hex값이라 몰랐는데 0x5f5e101이 100000001  
라업에서는 z3이라는 파이썬 lib로 사용  

아.. 쟤네 함수가 간단한 사칙연산이었음  
매개변수들끼리의... 오...  

```python
from z3 import *

def add(param_1, param_2):
    return param_1 + param_2

def sub(param_1, param_2):
    return param_1 - param_2

def mul(param_1, param_2):
    return param_1 * param_2

def xor(param_1, param_2):
    return param_1 ^ param_2

def and_(param_1, param_2):
    return param_1 & param_2

a, b, c, d, e, f = BitVecs('a b c d e f', 32)

s = Solver()

s.add(a > 100000001)
s.add(b > 100000001)
s.add(c > 100000001)
s.add(d > 100000001)
s.add(e > 100000001)
s.add(f > 100000001)

s.add(a < 1000000000)
s.add(b < 1000000000)
s.add(c < 1000000000)
s.add(d < 1000000000)
s.add(e < 1000000000)
s.add(f < 1000000000)

uVar1 = sub(a, b)
uVar2 = add(uVar1, c)
uVar3 = add(a, b)
uVar1 = mul(2, b)
uVar4 = mul(3, a)
uVar5 = sub(uVar4, uVar1)
uVar6 = xor(a, d)
uVar1 = add(c, a)
uVar7 = and_(b, uVar1)
uVar11 = add(b, d)
uVar1 = add(d, f)
uVar8 = xor(c, uVar1)
uVar9 = sub(e, f)
uVar10 = add(e, f)

s.add(uVar2 % 0x10ae961 == 0x3f29b9)
s.add(uVar3 % 0x1093a1d == 0x8bdcd2)
s.add(uVar5 % uVar6 == 0x212c944d)
s.add(uVar7 % 0x6e22 == 0x31be)
s.add(uVar11 % a == 0x2038c43c)
s.add(uVar8 % 0x1ce628 == 0x1386e2)
s.add(uVar9 % 0x1172502 == 0x103cf4f)
s.add(uVar10 % 0x2e16f83 == 0x16ab0d7)

if s.check() == sat:
    m = s.model()
    print(f'a = {m[a]}\nb = {m[b]}\nc = {m[c]}\nd = {m[d]}\ne = {m[e]}\nf = {m[f]}')
else:
    print("No solution found")
```