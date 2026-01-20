---
layout: post
title: "[PY] 백준 10988 팰린드롬인지 확인하기"
categories: [Self-study]
tags: [Python, Algorithm, Palindrome]
last_modified_at: 2024-01-10
---

```python
n = input()
a = list(n)
m = list(reversed(a))

ans = 1
for i in range(len(a)):
    if a[i] != m[i]:
        ans = 0
        break

print(ans)
```