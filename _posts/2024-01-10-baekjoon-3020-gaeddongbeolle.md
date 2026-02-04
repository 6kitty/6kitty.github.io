---
layout: post
title: "[PY] 백준 3020 개똥벌레"
categories: [Algorithm]
tags: [백준, 개똥벌레, Python]
last_modified_at: 2024-01-10
---

```python
n, h = map(int, input().split())

a = [0] * (h + 1)
b = [0] * (h + 1)

min = n
count = 0

for i in range(n):
    if i % 2 == 0:
        a[int(input())] += 1
    else: 
        b[int(input())] += 1

for i in range(h - 1, 0, -1):
    a[i] += a[i + 1]
    b[i] += b[i + 1]

for i in range(1, h + 1):
    if min > (a[i] + b[h - i + 1]):
        min = a[i] + b[h - i + 1]
        count = 1
    elif min == (a[i] + b[h - i + 1]):
        count += 1

print(min, count)
```