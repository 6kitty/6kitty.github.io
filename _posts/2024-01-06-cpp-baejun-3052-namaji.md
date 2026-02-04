---
layout: post
title: "[CPP] 백준 3052 나머지"
categories: [Algorithm]
tags: [C++, 백준, 나머지, 문제풀이]
last_modified_at: 2024-01-06
---

두 자연수 A와 B가 있을 때, A%B는 A를 B로 나눈 나머지 이다. 예를 들어, 7, 14, 27, 38을 3으로 나눈 나머지는 1, 2, 0, 2이다.

수 10개를 입력받은 뒤, 이를 42로 나눈 나머지를 구한다. 그 다음 서로 다른 값이 몇 개 있는지 출력하는 프로그램을 작성하시오.

```cpp
#include <iostream>
using namespace std;

//괄호, 오타 유의 

int main() {
    //초기 준비 
    int s;
    int n;
    int r[42];
    for(s=0; s<42; s++) {
        r[s] = 0;
    }
    
    //입력과 나머지 
    int R;
    for(s=0; s<10; s++) {
        cin >> n;
        R = n % 42;
        r[R]++;
    }
    
    //출력
    int count = 0;
    for(s=0; s<42; s++) {
        if(r[s] != 0)
            count++;
    }
    cout << count;
    
    return 0;
}
```