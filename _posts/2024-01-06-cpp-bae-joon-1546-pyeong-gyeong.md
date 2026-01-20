---
layout: post
title: "[CPP] 백준 1546 평균"
categories: [Self-study]
tags: [C++, Algorithm, Average]
last_modified_at: 2024-01-06
---

세준이는 기말고사를 망쳤다. 세준이는 점수를 조작해서 집에 가져가기로 했다. 일단 세준이는 자기 점수 중에 최댓값을 골랐다. 이 값을 M이라고 한다. 그리고 나서 모든 점수를 점수/M*100으로 고쳤다.

예를 들어, 세준이의 최고점이 70이고, 수학점수가 50이었으면 수학점수는 50/70*100이 되어 71.43점이 된다.

세준이의 성적을 위의 방법대로 새로 계산했을 때, 새로운 평균을 구하는 프로그램을 작성하시오.

```cpp
#include <iostream>
#include <string>
#include <sstream>
#include <algorithm>
using namespace std;

int main() {
    //입력
    int N;
    cin >> N;
    double arr[1001];
    
    //둘째줄 입력
    for(int i = 0; i < N; i++) {
        cin >> arr[i];
    }
    
    //평균은 자료형 int 아님 유의 
    //알고리즘헤더로 정렬하고 최댓값 고르기 
    sort(arr, arr + N);
    double sum = 0;
    for(int t = 0; t < N; t++) {
        arr[t] = (arr[t] / arr[N - 1]) * 100;
        sum += arr[t];
    }
    
    //출력 
    cout << sum / N;
    return 0; 
}
```