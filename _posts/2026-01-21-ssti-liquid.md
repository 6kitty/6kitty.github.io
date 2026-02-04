---
layout: post
title: "jekyll 블로그에서 SSTI 웹해킹 문제 writeup 작성하기"
categories: [Web Hacking]
tags: [SSTI]
last_modified_at: 2026-01-21
render_with_liquid: false
---

`tistory2git`으로 백업 다 하고 openai가 파일명이나.. 적당히 살펴봐야 할 것들이 있는데 SSTI 라이트업이 진짜 코드로 인식되어서 렌더링된 사이트에서도 안 뜨고 자꾸 terminal에 liquid warning이라고 뜨는 문제 발생.. jekyll이 사용하는 liquid 템플릿 엔진이 `{{ }}` `{% %}` 같은 걸 인식한다. 

### 방법 1. {% raw %} 사용 

``` 
{% raw %}
    ```py
    payload = "{{ ''.__class__.__mro__[1].__subclasses__() }}"
    ```
{% endraw %}
```

### 방법 2. html encoding 

```bash
{ → &#123;
} → &#125;
% → &#37;
```

이건 좀 귀찮; 

### 방법 3. md config에 추가 

```
---
layout: post
title: "-"
render_with_liquid: false
---
```

이런 식으로 `render_with_liquid`을 false로 설정하는 방법도 있다. post가 순수 마크다운일 경우에만 사용하자. 