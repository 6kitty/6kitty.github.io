---
layout: post
title: "[CPP] 백준 10811 바구니 뒤집기"
categories: [Algorithm]
tags: [CPP, Algorithm, Baekjoon]
last_modified_at: 2024-01-06
---

도현이는 바구니를 총 N개 가지고 있고, 각각의 바구니에는 1번부터 N번까지 번호가 순서대로 적혀져 있다. 바구니는 일렬로 놓여져 있고, 가장 왼쪽 바구니를 1번째 바구니, 그 다음 바구니를 2번째 바구니, ..., 가장 오른쪽 바구니를 N번째 바구니라고 부른다.

도현이는 앞으로 M번 바구니의 순서를 역순으로 만들려고 한다. 도현이는 한 번 순서를 역순으로 바꿀 때, 순서를 역순으로 만들 범위를 정하고, 그 범위에 들어있는 바구니의 순서를 역순으로 만든다.

바구니의 순서를 어떻게 바꿀지 주어졌을 때, M번 바구니의 순서를 역순으로 만든 다음, 바구니에 적혀있는 번호를 가장 왼쪽 바구니부터 출력하는 프로그램을 작성하시오.

```cpp
#include <iostream>
#include <algorithm>
using namespace std; 

//오타, 괄호 확인

int main() {
    //입력 
    int bagu[101];
    int s;
    int N,M;
    cin >> N >> M;
    for(s=0; s<=N; s++) {
        bagu[s] = s;
    }
    
    //바구니 순서 바꾸기 
    int i,j;
    for(s=0; s<M; s++) {
        cin >> i >> j;
        //두번째 인자 전까지 리버스 
        reverse(bagu+i, bagu+j+1);
    }
    
    //출력 
    for(s=1; s<=N; s++) {
        cout << bagu[s] << ' ';
    }
    return 0;
}
```