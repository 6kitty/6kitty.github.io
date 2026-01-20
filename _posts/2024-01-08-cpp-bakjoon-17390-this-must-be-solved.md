---
layout: post
title: "[CPP] 백준 17390 이건 꼭 풀어야 해!"
categories: [Self-study, Writeup]
tags: [백준, C++, 알고리즘, 정렬, prefix sum]
last_modified_at: 2024-01-08
---

길이 N짜리 수열 A  
A를 비내림차순 정렬 B  
이걸 Q개  

*비내림차순 (좌항) <= (우항)  

B 합 출력  

```cpp
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr), cout.tie(nullptr);
    
    int n, q;
    cin >> n >> q;
    vector<ll> v(n + 1);
    for (int i = 1; i < n + 1; ++i)
        cin >> v[i];

    sort(v.begin(), v.end());

    for (int i = 1; i < n + 1; ++i)
        v[i] += v[i - 1];
    
    while (q--) {
        int l, r;
        cin >> l >> r;
        cout << v[r] - v[l - 1] << '\n';
    }

    return 0;
}
```

모르겠어서 코드 긁어옴  
모르는 부분 배워 보겠삼  

1. bits/stdc++.h 헤더 : 자주 사용하는 라이브러리 컴파일함  
2. typedef long long ll;  
3. ios_base::sync_with_stdio(false); cin.tie(nullptr), cout.tie(nullptr); : 시간을 줄이기 위함.. 자세한 건 추가글로 설명..  

문제 풀이  
ll타입 벡터 생성하고 sort로 정렬  
prefix sum 생성하고 v[r]-v[l-1]  
이러면 마지막항일 때는 v[l]-v[l-1]  
첫번째항일 때는 v[r]-v[0]  
v[0]은 0으로 초기화 (int vector 0으로 자동 초기화 된다)  