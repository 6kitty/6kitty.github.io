---
layout: post
title: "[CPP] 백준 27866 문자와 문자열"
categories: [Algorithm]
tags: [C++, 백준, 문자열]
last_modified_at: 2024-01-08
---

```cpp
#include <iostream>
#include <string>
using namespace std;

int main() {
    string s;
    cin >> s;
    int i;
    cin >> i;
    i--;
    
    cout << s[i];
    return 0;
}
```