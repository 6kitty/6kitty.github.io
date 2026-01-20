---
layout: post
title: "[심화스터디] GRIZZLY 리서치 읽어보기"
categories: [Self-study, Writeup]
tags: [GRIZZLY, TEMU, 리서치, 보안, 데이터 수집]
last_modified_at: 2024-08-04
---

해당 보고서는 원고의 주장에 뒷받침된 자료이다. TEMU의 앱이 악성행위를 하고 사용자의 정보들을 몰래 무단 수집한다고 주장하는 글이다.

비지니스 모델 구조 상 가격이 싼 중국 쇼핑 앱은 지속 가능성이 부족 -> 손실이 있을 수밖에 없음. 그렇다면 TEMU는 어디서 수익을 얻을까? -> 불법적 데이터 거래 의심.

### TEMU의 위험성

외부 디바이스 데이터 호출과 개인정보에 접근하는 기능이 포함되었는지에 대한 표  
-> TEMU는 all YES (2023.8.30 기준의 앱 버전 1.99)

### 앱 소프트웨어 분석

분석 내용  
1. 앱 스토어 정책을 위반하는 코드 혹은 구성 요소가 있는가?  
2. 개인 정보에 접근하거나 악성 행위를 하는 시스템 호출이 있는가?  
3. 과도한 난독화나 은닉이 있는가?  

#### 분석 결과 1. runtime.exec() 사용

위 함수를 사용해서 동적 컴파일 진행.  
runtime.exec() 함수로 패키지 compile 호출 -> 위 사진 25,53줄 참고.  
프로세스 내부에서 컴파일을 진행함으로써 보안 검사 일차적 회피 (동적 프로세스라 탐지가 어려움).

#### 분석 결과 2. android.permission 항목 참조

안전한 표준 라이브러리에서 발생하는 경우를 제외한 android.permission 항목이 참조됨.  
안드로이드에서는 필요한 권한은 Manifest에 명시해둠.  
하지만, TEMU는 Manifest에 언급하지 않은 권한 존재:  
CAMERA, RECORD_AUDIO, WRITE_EXTERNAL_STORAGE, INSTALL_PACKAGES, and ACCESS_FINE_LOCATION.  
스파이웨어가 다룰 가능성이 높은 권한들.

#### 분석 결과 3. 위치 권한 요청

superuser 권한 및 로그 파일을 참조하여 사용자 기기의 모든 파일에 대한 정보를 취득.  
즉, 특정 Android 버전에서는 chat logs, images, user content on other apps 등 데이터 읽기, 처리, 수정 가능.  

3a) 그들의 API 'us.temu.com'과 연결되는 명령 서버 존재.  
그리고 그 서버를 기반으로 한 file upload 기능 존재.  
사용자가 TEMU 앱에 파일 저장 권한 부여 시, TEMU는 사용자 기기 내 파일을 원격 수집 가능.

3b) 상품 이미지 검색 -> 카메라 접근 권한인 척 위치 권한 요청(FINE_PERMISSION).

![상품 이미지 검색 시 위치 권한 요청](https://blog.kakaocdn.net/dna/EJ21P/btsITCorDvj/AAAAAAAAAAAAAAAAAAAAADDNH-ZSE9wM3D1l2gNirvLkueTnq6ohIl3Fsg8E1mBz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=FViiYqxPYkb0rO0S4PJNO9cFjqE%3D)

#### 분석 결과 4. ACCESS_FINE_LOCATION

안드로이드에서는 위치 관련 기능이 ACCESS_COARSE_LOCATION과 ACCESS_FINE_LOCATION이 있다.  
전자는 개인정보 침해를 하지 않으면서 적절한 수준의 위치 데이터 수집을 허락하고 후자는 더 자세한 위치 데이터를 제공한다.  
때문에 안드로이드에서 사용을 자제한다(지도 앱 제외).  
그러나 해당 함수가 디컴파일된 코드에 내장되어 있었다. 정확한 수집 가능 시기(앱의 ON/OFF 등)는 안드로이드 버전, 부여된 권한 등 여러 변수가 있다.

![ACCESS_FINE_LOCATION](https://blog.kakaocdn.net/dna/SUsOE/btsIVibeLw2/AAAAAAAAAAAAAAAAAAAAAEt14IUPiqnvfmrqk-2lBNJ0ZhnsSB64GDxeIm5hEbYS/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7sgbxjmlHneUMBEmjLamb2M8SEs%3D)

#### 분석 결과 5. Root Access, Debug check

TEMU는 디바이스에 "루트" 액세스 권한이 있는지 check하고 실행중인 디버거가 있는지 check한다.  
쇼핑 앱에 이러한 코드가 들어있다는 건 상당히 의심되는 부분.

#### 분석 결과 6. User's System Logs

시스템 로그 파일은 기기의 모든 프로세스에 대한 기록이 남겨져 있는 파일.  
오류나 경고 등 모든 프로세스 정보가 담겨 있음. TEMU는 앱 범위를 벗어난 사용자 시스템 로그 열람 -> 심각한 개인정보 침해 우려.

![logcat 명령어](https://blog.kakaocdn.net/dna/bY3gZB/btsIU9ejJZg/AAAAAAAAAAAAAAAAAAAAAMsB0jO3A_QWnfjkjPbpwXW5eUOqfPEj_pDCDq9G6JvT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2B6ixyNDfaoFX%2FDnVm1EvRfD7Yfo%3D)

#### 분석 결과 7. User's System Logs

MAC 주소 및 기타 장치 정보를 요청 -> JSON 객체에 삽입하여 서버로 전송.  
쇼핑 앱에 고객 디바이스의 기술적 세부 정보 데이터베이스가 필요?  
MAC 주소: 모든 네트워크에 있는 모든 디바이스의 글로벌 고유 식별자 -> 서버와 통신할 때 특정 디바이스의 특정 사용자에게 정보와 소스 코드 전송 가능.

![MAC 주소 요청](https://blog.kakaocdn.net/dna/batkil/btsITnrygoi/AAAAAAAAAAAAAAAAAAAAAPfHEpBB-LwnAFEtJ2YGGNY49WxOAjeuaOfpznvOdVXT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=GudA%2Br5Wnmlk0dbbGgP%2BizPkvjM%3D)