---
layout: post
title: "[DownUnderCTF] REV-number mashing"
categories: [System Hacking]
tags: [CTF, Wargame, Problem Solving]
last_modified_at: 2024-07-10
---

if문들만 피해주면 되는 간단한 문제  

v4=v4*v5가 되는 v4와 v5를 찾아주면 된다  
이건 항등식인데 v4는 0이 안되고 v5는 0이나 1이 되면 안된다.  
아마 int의 최대최소 제한을 이용해서 푸는 문제 같다.  
대충 int의 최대최소 간의 거리를 v5번 순회해서 v4로 돌아오는 값 생각해주면 될 거 같은데  
지금 노트가 없어서 집 가서 푸는 걸로 하고...  

처음 생각한 식은 0부터 거리가 a일 때 4a=2^32(int의 자료형 크기)라 생각하는 a=v4와 5=v5였다  
근데 이게 INT_MAX에서 넘어가는 부분에서 내가 계산을 잘못한 건지 안되었다..  

signed라 헷갈리나 싶어서 unsigned로 먼저 생각해보았다.  

결론적으로 안됐다..^^  
뭔가 또 식을 잘못 세운 거 같다.  

다시 식을 세워서 챗지피티한테 물어봤다.

```python
# Constants
TARGET_PRODUCT = -2_147_483_648
CONSTANT_SUM = 2_147_483_648

# Function to find valid (a, n, m) triplets
def find_triplets(target_product, constant_sum):
    for a in range(-2_147_483_648, 0):
        if target_product % a == 0:
            n = target_product // a
            if n > 0:  # n must be a positive integer
                m = (constant_sum - a) // (a - 1)
                if m > 0 and (constant_sum - a) % a == 0:  # m must be a positive integer
                    return a, n, m
    return None

result = find_triplets(TARGET_PRODUCT, CONSTANT_SUM)
print(result)
```

이렇게 나왔는데 v5는 1이 불가능하다.. if에 걸림  

라이트업 참고했더니 1 대신 -1을 넣으면 되는 문제였다..  

자료형 오버플로우 관련한 문제는 생각보다 많이 나오는 거 같다.  
아이디어 생각한 건 좋았는데 언제나 그랬듯이 마지막을 제대로 풀지 못해서 solve 못하는 경우가 많은데...  
조금 양치기 하면 괜찮아지겠지 생각중이다.  