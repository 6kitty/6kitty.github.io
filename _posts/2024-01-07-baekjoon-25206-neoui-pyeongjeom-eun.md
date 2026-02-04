---
layout: post
title: "[PY] 백준 25206 너의 평점은"
categories: [Algorithm]
tags: [Python, 백준, 평점]
last_modified_at: 2024-01-07
---

```python
a=['']*20
b=[0]*20
c=[0]*20
cnt=0
for i in range(20):
    a[i],e,f=input().split()
    b[i]=float(e)
    if f=='P':
        b[i]=0
        continue
    if f=='F':
        c[i]=0
        cnt+=1
        continue
    list(f)
    c[i]=4-(ord(f[0])%65)
    if f[1]=='+':
        c[i]+=0.5
    c[i]*=b[i]

print(sum(c)/sum(b))
```