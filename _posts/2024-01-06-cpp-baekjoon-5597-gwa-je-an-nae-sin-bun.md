---
layout: post
title: "[CPP] 백준 5597 과제 안 내신 분..?"
categories: [Algorithm]
tags: [C++, 백준, 알고리즘]
last_modified_at: 2024-01-06
---

X대학 M교수님은 프로그래밍 수업을 맡고 있다. 교실엔 학생이 30명이 있는데, 학생 명부엔 각 학생별로 1번부터 30번까지 출석번호가 붙어 있다.

교수님이 내준 특별과제를 28명이 제출했는데, 그 중에서 제출 안 한 학생 2명의 출석번호를 구하는 프로그램을 작성하시오.

```cpp
#include <iostream>
using namespace std; 

//오타 확인, 괄호 확인

int main() {
	//입력
    int pass[31];
    for(int s=0;s<31;s++){
    	pass[s]=0;
    }
    
    int n;
    for(int s=0;s<28;s++){
    	cin>>n;
        pass[n]=n;
    }
    
    //검사와 출력
    for(int s=1;s<31;s++){
    	if(pass[s]==0){
        	cout<<s<<endl;
        }
    }
    
    return 0;
}
```