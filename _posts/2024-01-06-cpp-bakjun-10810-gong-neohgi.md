---
layout: post
title: "[CPP] 백준 10810 공 넣기"
categories: [SWING, Writeup]
tags: [백준, C++, 알고리즘]
last_modified_at: 2024-01-06
---

도현이는 바구니를 총 N개 가지고 있고, 각각의 바구니에는 1번부터 N번까지 번호가 매겨져 있다. 또, 1번부터 N번까지 번호가 적혀있는 공을 매우 많이 가지고 있다. 가장 처음 바구니에는 공이 들어있지 않으며, 바구니에는 공을 1개만 넣을 수 있다.

도현이는 앞으로 M번 공을 넣으려고 한다. 도현이는 한 번 공을 넣을 때, 공을 넣을 바구니 범위를 정하고, 정한 바구니에 모두 같은 번호가 적혀있는 공을 넣는다. 만약, 바구니에 공이 이미 있는 경우에는 들어있는 공을 빼고, 새로 공을 넣는다. 공을 넣을 바구니는 연속되어 있어야 한다.

공을 어떻게 넣을지가 주어졌을 때, M번 공을 넣은 이후에 각 바구니에 어떤 공이 들어 있는지 구하는 프로그램을 작성하시오.

```cpp
#include <iostream>
using namespace std; 

int main() {
	//N,M 입력받기 
	int N, M;
    cin >> N >> M;
    
    //바구니 초기화 
    int t; 
    int bagu[N];
    for(t=0; t<N; t++){
    	bagu[t]=0;
    }
    
    //i,j,k 입력받기 
    int i,j,k;
    int p[M],q[M],s[M];
    for(t=0; t<M; t++){
    	cin >> i >> j >> k;
        p[t]=i;
        q[t]=j;
        s[t]=k;
    }
    
    //공 넣기 
    int x;
	for(t=0; t<M; t++){
    	for(x=p[t]; x<=q[t]; x++){
        	bagu[x]=s[t];
        }
    }
            
    //출력 
    for(t=0; t<N; t++){
    	cout << bagu[t] << " ";
    }
}
```

```cpp
#include <iostream>
using namespace std; 

int main() {
	//N,M 입력받기 
	int N, M;
    cin >> N >> M;
    
    //바구니 초기화 
    int t; 
    int bagu[101] = {0, };
    
    //i,j,k 입력받고 바로 넣기
    int i,j,k,s;
    for(t=0; t<M; t++){
    	cin >> i >> j >> k;
        for(s=i; s<=j; s++){
        	bagu[s]=k;
        }
    }

    //출력 
    for(t=1; t<=N; t++){
    	cout << bagu[t] << " ";
    }
    //1번부터 매겨져 있는데 0부터 시작해서 틀렸던듯 
    
    return 0;
}
```