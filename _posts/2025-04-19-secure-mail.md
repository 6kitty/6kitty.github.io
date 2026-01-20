---
layout: post
title: "Secure mail"
categories: [Writeup]
tags: [Bruteforce, JavaScript, CTF]
last_modified_at: 2025-04-19
---

f12 눌러서 개발자모드 들어가본다

obfuscation 되어 있다. js 난독화는 유명하니까 web으로도 deobfus를 할 수 있는데 유의미한 함수가 나오진 않았다.

하나하나 분석하는 문제는 아닌 거 같고 6자리밖에 안되니까 BF로 풀어준다.

```javascript
// 브루트포스 함수 정의
async function bruteforcePassword() {
  // 알림 핸들러 설정
  let isWrong = false;
  const originalAlert = window.alert;
  
  window.alert = function(message) {
    if (message === 'Wrong') {
      isWrong = true;
      // 알림 창 자동으로 닫기 (확인 버튼 클릭)
      setTimeout(() => {
        const alertButton = document.querySelector(".alert-button") || 
                           document.querySelector("[data-dismiss='alert']") || 
                           document.querySelector(".close");
        if (alertButton) alertButton.click();
      }, 100);
    } else {
      // Wrong이 아닌 다른 알림이 뜨면 원래 alert 함수 호출하고 브루트포스 중단
      originalAlert(message);
      console.log("패스워드 발견: " + inp);
      window.alert = originalAlert; // 원래 alert 함수 복원
      return true; // 성공 신호 반환
    }
    return false;
  };

  // 연도 범위: 1980(80)~2024(24)
  for (let year = 80; year <= 99; year++) {
    for (let month = 1; month <= 12; month++) {
      for (let day = 1; day <= 31; day++) {
        // 유효하지 않은 날짜 건너뛰기 (간단한 검증)
        if ((month === 4 || month === 6 || month === 9 || month === 11) && day > 30) continue;
        if (month === 2) {
          // 윤년 계산 (간단한 버전)
          const fullYear = year + 1900;
          const isLeapYear = (fullYear % 4 === 0 && fullYear % 100 !== 0) || (fullYear % 400 === 0);
          if (day > (isLeapYear ? 29 : 28)) continue;
        }
        
        // 생년월일 형식으로 변환 (YYMMDD)
        const yearStr = year.toString().padStart(2, '0');
        const monthStr = month.toString().padStart(2, '0');
        const dayStr = day.toString().padStart(2, '0');
        const inp = yearStr + monthStr + dayStr;
        
        console.log("시도 중: " + inp);
        
        // 입력 필드에 값 설정하고 제출 버튼 클릭
        document.querySelector("input[id=pass]").value = inp;
        document.querySelector("button[type='submit']").click();
        
        // 응답 대기
        await new Promise(resolve => setTimeout(resolve, 500));
        
        // Wrong 알림이 아닌 다른 결과가 나왔다면 성공으로 간주하고 중단
        if (!isWrong) {
          console.log("패스워드 발견: " + inp);
          window.alert = originalAlert; // 원래 alert 함수 복원
          return;
        }
        
        isWrong = false; // 다음 시도를 위해 초기화
      }
    }
  }
  
  console.log("모든 가능한 패스워드를 시도했지만 찾지 못했습니다.");
  window.alert = originalAlert; // 원래 alert 함수 복원
}

// 브루트포스 시작
bruteforcePassword();
```

코드는 지피티 도움 받았고 이걸 console에 입력했다

열심히 기다리면

![발견](https://blog.kakaocdn.net/dna/rvX66/btsNrvqYvK9/AAAAAAAAAAAAAAAAAAAAAEIoQZA0fhyBGZcPimwVuuiwSuZqIPV583W8WPXCfKOt/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=13If9uzfO3zu0s%2BFdct8bUKtSaU%3D)

발견

![결과](https://blog.kakaocdn.net/dna/ci985j/btsNrnOlNj2/AAAAAAAAAAAAAAAAAAAAAExwV4BoXd6fzVOtFIpnejcq5dBPHToJ8hJuo4RU3Df1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=VH8GCljlqcupPok6dRGGaC1llN8%3D)

lv1 치고 오래 걸리지 않나..