---
layout: post
title: "[PY] 백준 1157 단어 공부"
categories: [Self-study]
tags: [Python, Baekjoon, Algorithm]
last_modified_at: 2024-01-07
---

```python
n = input().lower()
nlist = list(set(n))  # 중복 없음
cnt = []

for i in nlist:
    c = n.count(i)
    cnt.append(c)

if cnt.count(max(cnt)) > 1:
    print("?")
else:
    print(nlist[(cnt.index(max(cnt)))].upper())
```