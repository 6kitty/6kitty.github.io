---
layout: post
title: "[CPP] 백준 2444 별 찍기 - 7"
categories: [Self-study]
tags: [C++, 백준, 별찍기]
last_modified_at: 2024-01-06
---

```cpp
#include <iostream>
using namespace std;

int main() {
    int N=0;
    cin >> N;
    
    for(int i=1; i<=N; i++) {
        for(int j=N-i; j>0; j--)
            cout << " ";
        for(int j=2*i-1; j>0; j--) {
            cout << "*";
        }
        cout << endl;
    }
    
    for(int i=1; i<N; i++) {
        for(int j=0; j<i; j++) {
            cout << " ";
        }
        for(int j=2*(N-i)-1; j>0; j--) {
            cout << "*";
        }
        cout << endl;
    }
    return 0;
}
```

이렇게 함

메모리 초과임 

줄여야 함 

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    int N;
    cin >> N;
    string ar[199];
    
    int M=1;
    int Q=N-M;
    int s, t;
    for(s=0; s<N; s++) {
        for(t=0; t<Q; t++) {
            ar[s] += " ";
            Q--;
        }
        for(t=0; t<M; t++) {
            ar[s] += "*";
            M += 2;
        }
        cout << ar[s] << endl;
    }
    for(s=N-2; s>=0; s--) {
        cout << ar[s] << endl;
    }
    return 0;
}
```

음 얘도 실패 

아래는 모범 답안 

```cpp
#include <iostream>
using namespace std;

int main() {
    int N=0;
    cin >> N;
    
    for(int i=1; i<=N; i++) {
        for(int j=N-i; j>0; j--)
            cout << " ";
        for(int j=2*i-1; j>0; j--) {
            cout << "*";
        }
        cout << endl;
    }
    
    for(int i=1; i<N; i++) {
        for(int j=0; j<i; j++) {
            cout << " ";
        }
        for(int j=2*(N-i)-1; j>0; j--) {
            cout << "*";
        }
        cout << endl;
    }
    return 0;
}
```