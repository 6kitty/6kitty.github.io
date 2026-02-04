---
layout: post
title: "[WebHacking.kr] old-54번 문제 풀이 (Write-up)"
categories: [Web Hacking]
tags: [javascript, exploit, ctf, web]
last_modified_at: 2026-01-12
---

Web2/old-54 문제

자바스크립트의 동작 방식을 분석 -> 로직을 수정하여 실행하는 Client side scripting 문제

1. **문제 분석 (Analysis)**

f12로 소스 코드를 확인 src/script.js 

```javascript
function run(){
  if(window.ActiveXObject){
   try {
    return new ActiveXObject('Msxml2.XMLHTTP');
   } catch (e) {
    try {
     return new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) {
     return null;
    }
   }
  }else if(window.XMLHttpRequest){
   return new XMLHttpRequest();

  }else{
   return null;
  }
}

x=run();

function answer(i){
  x.open('GET','?m='+i,false);
  x.send(null);
  aview.innerHTML=x.responseText;
  i++;
  if(x.responseText) setTimeout("answer("+i+")",20);
  if(x.responseText=="") aview.innerHTML="?";
}

setTimeout("answer(0)",1000);
```

answer(i) 함수가 핵심적인 역할이다. 

XMLHttpRequest 객체를 생성 -> ?m=i 파라미터를 통해 서버에 요청을 보냄 -> 여기서 i는 0부터 1씩 증가한다. 

서버로부터 받은 응답(x.responseText)을 aview.innerHTML에 덮어쓴다. 

응답 값이 존재하면 20ms 뒤에 i를 1 증가시켜 재귀적으로 answer 함수를 호출한다. 

2. **익스플로잇**

overwrite 말고 concatenate로 출력 로직 변경 

```haxe
var merged = '';
var i = 0;
var xhr = new XMLHttpRequest();

// 무한 루프를 돌며 순차적으로 요청을 보냄
while(true) {
  // 동기(false) 방식으로 요청하여 순서 보장
  xhr.open('GET', '?m=' + i, false);
  xhr.send(null);

  // 응답이 없으면(플래그 끝) 반복 종료
  if(!xhr.responseText) break;

  // 응답 받은 글자를 누적
  merged += xhr.responseText;
  i++;
}

// 결과 출력
console.log("FLAG:", merged);
alert(merged);
```

**코드 설명:**

while(true) 문을 사용하여 계속해서 요청을 보낸다. 

xhr.open의 세 번째 인자를 false로 설정하여 동기(Synchronous)로 설정해야 응답이 올 때까지 대기(만약에 비동기로 설정하면 응답 수신을 생각 안 하고 다음 명령으로 넘어갈 수 있으므로 오류날 수 있다) 

merged += xhr.responseText를 통해 한 글자씩 받은 데이터를 계속 이어 붙인다. 

서버에서 더 이상 문자를 반환하지 않으면 break로 루프를 탈출하고 완성된 문자열을 출력한다. 

XMLHttpRequest의 open()함수의 인자 설명 

1. http 메서드 지정( get 혹은 post) 
2. 접속할 url을 입력한다 
3. true가 비동기, false가 동기 
```