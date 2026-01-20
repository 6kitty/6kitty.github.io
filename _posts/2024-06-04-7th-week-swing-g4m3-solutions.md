---
layout: post
title: "[7주차] SWING G4M3 풀어보기"
categories: [SWING, Writeup, Self-study, +]
tags: []
last_modified_at: 2024-06-04
---

1. **31기 함은지 님 "SWING_JJANG" is not plaintext**

   Correct 문자열이 존재한다.

   ![Image 1](https://blog.kakaocdn.net/dna/sAzzr/btsHLg7FvcJ/AAAAAAAAAAAAAAAAAAAAAMfKpY1YgHQ4K-90zGexgr-OudKTeK3QyqUgN5MiJjgX/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6NL%2BWTxqmh9UHNeTsS7pl0FW%2BSw%3D)

   hi 한 번 입력하고 동적분석 진행하면

   ![Image 2](https://blog.kakaocdn.net/dna/vtGNx/btsHKTrmUHO/AAAAAAAAAAAAAAAAAAAAADkJznd396d-a2coyigD0ne0rzgkMcHd7pMn2w9u40-0/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xNdptmJ4%2BhnYme1gAh7V%2BSupagw%3D)

   NRDIB_EEVIB라는 문자열이 보인다

   ![Image 3](https://blog.kakaocdn.net/dna/D3sN5/btsHLwvECL9/AAAAAAAAAAAAAAAAAAAAAOOMCAaA6aJhJRaVSMW6yp3z_85cBQKGL-_vASXVpnNK/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uIdF6L4lKd2njvW1EhjGd7fbW0k%3D)

   다시 실행하여 입력해주면

   ![Image 4](https://blog.kakaocdn.net/dna/vBCYy/btsHMd97Quu/AAAAAAAAAAAAAAAAAAAAAOrifEF5ZPY9g1iUDS93JKQ0hZJpvNv4r06BzthjcfTw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=eOjmXwDF3zXKoMeEoMDbm795e2o%3D)

   성공

2. **31기 지현아 님**

   문자열을 찾아준다

   ![Image 5](https://blog.kakaocdn.net/dna/ceaYDs/btsHKSlHE95/AAAAAAAAAAAAAAAAAAAAAEKdyn_AsAUab_IPFTNiyYEE38yyZUPGLZXp5FLHKa-L/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=OYRELcr2W2RNRmS%2BjycvlsxJw%2Fo%3D)

   아무거나 입력해보고

   ![Image 6](https://blog.kakaocdn.net/dna/boPpY9/btsHLY6qFyT/AAAAAAAAAAAAAAAAAAAAALH88L3CO63Aczvi1KsWqITMFUj7Fz5NbctF4uW_Dio7/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=s4COryuMBrjuVMHOVTlsxGCgoQ4%3D)

   배열 하나씩 검사하는 거 같다

   ![Image 7](https://blog.kakaocdn.net/dna/MpHtd/btsHLVhBkv6/AAAAAAAAAAAAAAAAAAAAABw1uaApr3t01didFEjdikFAwiVz-Ri2_CdKcivjshX4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xOpeH7Q0UJN6gbTyzaHO0Ly1X4A%3D)

   ![Image 8](https://blog.kakaocdn.net/dna/MidOL/btsHKmOotcz/AAAAAAAAAAAAAAAAAAAAAPwW6pHeYmLhyAzdFt_uhPdAnYphP9pk7XCxM5mQIl2x/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rprsfBO0qCe3NcknZ1fbwqetTDU%3D)

   이렇다 할 문자열이 안 보였다

   ![Image 9](https://blog.kakaocdn.net/dna/Hrax7/btsHMMeJFTg/AAAAAAAAAAAAAAAAAAAAADClFu0oqISNS76jHOUrYUXH7F9FvbZezw6O21DWa2vF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=6BiB8jWQsaXZATOAM48dlwPSQzc%3D)

   아이다로 디컴파일 했더니

   ![Image 10](https://blog.kakaocdn.net/dna/G8Lpb/btsHM4MZXsp/AAAAAAAAAAAAAAAAAAAAACNHEOjfzvsxk3LFtE25Apj4tUZtuFh9g8i52erHH41u/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=V5XGHUCEaQS3LhaJnN52NQ%2BsZnE%3D)

   v8+=v6^0x41 주목

   이 v8이 25가 되면 성공하는 문제이다.

   ![Image 11](https://blog.kakaocdn.net/dna/mdK9s/btsHNuxPJxZ/AAAAAAAAAAAAAAAAAAAAAPnseAre9eef0nZOLJbYiEGgFGuSF2-cDMCGHx1VmShe/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Ji2xRuZ1zk8KCekRcFnqSy6lFDA%3D)

   0x44^0x41이 0x05이므로 대문자 D를 5번 입력해주면 총 25가 된다.

   ![Image 12](https://blog.kakaocdn.net/dna/bemtQD/btsHLu0nebT/AAAAAAAAAAAAAAAAAAAAALakxIsItcTICfnbhPgrk7FhtEWmdGnRR4qhJ8frPgAy/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=56XconwXjPkq9f%2BT9qkY1%2B1RHkc%3D)

   성공

3. **31기 노희민 님 알맞은 코드를 입력하시오.**

   PNG 파일

   ![Image 13](https://blog.kakaocdn.net/dna/rTibe/btsHKAZ2unq/AAAAAAAAAAAAAAAAAAAAAIJVRl0bYaaZXd7449oD79N5PpD0goCpWksM_G_pNNk_/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=XNnoBpBMScELiER2tdFWwcCrUqU%3D)

   푸터 시그니처가 없는 걸 보아 그냥 맨 앞 PNG만 지워주면 될 거 같다

   ![Image 14](https://blog.kakaocdn.net/dna/bszSoY/btsHLcK61p4/AAAAAAAAAAAAAAAAAAAAAFNq-qjU1Gqizcx9TaWqkwkzzLfrvnpaXmwKCC0GaBKo/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=rX05bF0j8kV51iySml8a%2FIoVi%2F4%3D)

   디버거로 열고 correct 문자열을 찾았다.

   ![Image 15](https://blog.kakaocdn.net/dna/clzB2f/btsHKGshH5Y/AAAAAAAAAAAAAAAAAAAAAOUER7CdH072Nmlhmtv8yqu4WfCZq-y6CRpOfZ19W1KP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=jLPeP%2FAwltmkXOu0kis%2FBQMmaQ0%3D)

   문자열 발견

   ![Image 16](https://blog.kakaocdn.net/dna/bndCN4/btsHLVaOapw/AAAAAAAAAAAAAAAAAAAAADT4VjguN8RKzvi1e7QqqU8Pq-QCPUYWjOaTDBg6BBcl/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=d%2FXfgQ9mRvs4N1o7%2F607pgDL1vE%3D)

   덤프에서도 확인 가능하다

   ![Image 17](https://blog.kakaocdn.net/dna/Yodrj/btsHKIjmvoN/AAAAAAAAAAAAAAAAAAAAALsjCQ1i5jsAn5tYizfvUUe13lxg83-jTotsZUfSLbeG/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Maqn3BlLbtVWJiVR18WAiD5lsqc%3D)

   성공

4. **31기 황선영 님**

   문자열 먼저 찾았다.

   ![Image 18](https://blog.kakaocdn.net/dna/bi1sIQ/btsHNoYJyaG/AAAAAAAAAAAAAAAAAAAAAOgiUz4RjxmwSIH3QLbGTmTl3fRD-vC0Hzl4x74_vqm4/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=IV3CYJ5feduniBEVQmc0uBPQLU8%3D)

   중간에 저장되어 있는 값이 보인다

   ![Image 19](https://blog.kakaocdn.net/dna/FcR9h/btsHNZjND41/AAAAAAAAAAAAAAAAAAAAAIhi79UOwpQ5H2rSue34JelbX3SiquMER63IHRx8QKf8/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QrdTpoqvACUCXUJwr3pGXuqr9oo%3D)

   umm... 근데 자꾸 Wrong!이 뜬다...

5. **31기 이시온 님**

   문자열 참조를 해보면 Jokermylove가 수상하다(이쯤되면 감이온다)

   ![Image 20](https://blog.kakaocdn.net/dna/bcTbhQ/btsHSwJldqQ/AAAAAAAAAAAAAAAAAAAAAAezz5UMz2_BZmvicCXBqwF6gzAhOYsn5EWe0Oa0UJeh/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uman1vwHL8ncGd1gDe9iree58r0%3D)

   동적분석하다보면 Jokerlove15가 나온다

   ![Image 21](https://blog.kakaocdn.net/dna/chtnZS/btsHSALz7Yp/AAAAAAAAAAAAAAAAAAAAAN1XV2NyPZebIpTYns3ahFRX1_SmxOs8qep6RGkIQ8se/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=beEbCcVrX%2F0Sfv5EN%2BVb5gSliEY%3D)