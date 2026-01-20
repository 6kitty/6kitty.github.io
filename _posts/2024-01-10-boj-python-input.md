---
layout: post
title: "[PY] 백준에서 파이썬 입력 받기"
categories: [Self-study]
tags: [Python, Input, Baekjoon]
last_modified_at: 2024-01-10
---

입출력 처리 맨날 찾아보는 게 귀찮아서...

```python
#한 단어 입력 
a = int(input()) #당연히 문자열이면 Int 제거 
print(a)
```

```python
# 한 줄 입력 - 구분자, 띄어쓰기 미포함 
#input = 54321, 모두 더해서 출력 
print(sum(map(int, input()))) # 15
```

```python
#한 줄 입력 - 구분자, 띄어쓰기 포함 
#split
a, b = map(int, input().split())

#콤마로 split 
a, b = map(int, input().split(','))
print(a + b)
```

```python
#여러 줄 입력, 몇 줄이 입력될지 아는 경우
n = int(input())
for i in range(n):
    a, b = map(int, input().split())
```

```python
#여러 줄 입력, 몇 줄이 입력될지 모르는 경우
while True:
    try:
        a, b = map(int, input().split())
    except:
        break
```