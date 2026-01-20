---
layout: post
title: "누적 합(prefix sum)"
categories: [Self-study]
tags: []
last_modified_at: 2024-01-12
---

누적 합: index 0부터 n까지 탐색 -> index i일 때, 0부터 i까지의 총합  

표현 방법  

1. 시간복잡도가 O(N^2)인 방법  
```python
array=[1,2,3,4,5,6]
n=len(array)
prefixsum=[0]*n 

for i in range(n):
    for j in range(i+1):
        prefixsum[i]+=array[j]
```

2. 시간복잡도가 O(N)인 방법 <- 권장  
```python
array=[1,2,3,4,5,6]
n=len(array)
prefixsum=[0]*(n+1)

for i in range(n):
    prefixsum[i+1]=prefixsum[i]+array[i]
```

누적합을 이용하여 구간합 구하기 가능