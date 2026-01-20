---
layout: post
title: "[6주차] FAT.001 문제 풀이"
categories: [Self-study, Writeup]
tags: [FAT, Disk Recovery, Forensics]
last_modified_at: 2024-05-21
---

디스크를 fix하라고 한다.

![Disk Fix](https://blog.kakaocdn.net/dna/RapRN/btsHu4tOG5N/AAAAAAAAAAAAAAAAAAAAAL2FsEJA4h7ag6DanVGBAiCSNd3a74aQRgHseejDNzTq/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=tifcEl07%2FEJTpXwVVyQ%2BGZt6vCc%3D)

일단 Fix the Disk!!!!는 지웠다.  
FAT 파일 복구를 검색해서 일단 시그니처인 55 AA를 검색했다.

![FAT Signature](https://blog.kakaocdn.net/dna/cJdqED/btsHxkvbiBL/AAAAAAAAAAAAAAAAAAAAAClAnMUeqMzGeTwuOlifiYYv-0_rff10WICSFbsTNWmr/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uI9AgJ%2FZQCgwrb1O03x%2F3F7iAOU%3D)

파티션 4개까지 존재 가능하다는데 3번부터 위치할 수도 있는 건가...? MBR 위치 자체를 잘 모르겠다.

![Partition Info](https://blog.kakaocdn.net/dna/V0PoM/btsHvrbKpoA/AAAAAAAAAAAAAAAAAAAAAO9f6WHN1yLDWPVMpZC5knEJaIJkBhEy14x3CZ9yhiJR/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=4UaFFLLUXUFBOsYija2ny91g7Uc%3D)

일단 파티션에서 짚어야 할 부분은 여기다. 앞 4바이트는 시작 주소, 뒤 4바이트는 끝 주소를 의미한다.  
앞 4바이트를 읽어서(리틀엔디언) 계산한 섹터가 BR 위치라는데...

![BR Location](https://blog.kakaocdn.net/dna/QnxF3/btsHwvRKBIx/AAAAAAAAAAAAAAAAAAAAAG8Wjfrzl_7Zz-H7A2M8f6lZAaiajM2ZrBYHJ-gOCWfk/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=S2WXN0Wrhr0SSvqr9AIxMnHlTq0%3D)

...존재하는 섹터보다 큰 수가 나왔다.  

![Sector Size](https://blog.kakaocdn.net/dna/c98sS8/btsHxqWuQjo/AAAAAAAAAAAAAAAAAAAAAES8ymQf1E0dMmfpPM4-I0zeMI6-JPJQrg3CP9OKkJrZ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=sgzPZMRkZrQwoW8MfGbmhZvL%2BS0%3D)

일단 55 AA 보이는 곳중에서 가장 내용이 많고 그럴싸한(?) 곳이다. MSDOS5인 것을 보아 구조가 맞는 거 같은데...

라업 봤다. 위 섹터를 0 영역에 덮어씌워야 한다. 섹터 0에 부트 레코드가 존재하기 때문이라고 한다.

![Boot Record](https://blog.kakaocdn.net/dna/cBtjml/btsHx0p8S9A/AAAAAAAAAAAAAAAAAAAAAIV6w9SPOpnFU4zsAbWfRXxOb3Y17Lws2-s6mbP7ELDf/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=qPtQL9ZnckZ4y%2F5qB3nvLBg0DQk%3D)

다시 ftk에서 열어보자 (개인적으로 autopsy보다 ftk가 더 편해서 해당 툴 사용했다...)  
잠시 라업 넣어두고 다시 내가 풀었다.

![FTK Tool](https://blog.kakaocdn.net/dna/K6CCq/btsHywWDD5O/AAAAAAAAAAAAAAAAAAAAAGtq85LYHiqQitfgUWMxwzPki5zaW7-zvxU0cv0PEtw3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Skr%2FqM9rsLWlVTsAusY90aMw4Xc%3D)

막 둘러보니까 Dreamhack 폴더가 있다.

![Dreamhack Folder](https://blog.kakaocdn.net/dna/8AZth/btsHwshm1kf/AAAAAAAAAAAAAAAAAAAAAG8wUyyIiAS9gwq3jHch1wi2SY_IOdjyJnfKa6ba20ui/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Huv%2BY1URgyou2u%2BUGjP2z0lxFos%3D)

flag 있는 파일을 찾은 거 같은데 암호가 있다.

![Flag File](https://blog.kakaocdn.net/dna/A9lK4/btsHxAFxLl6/AAAAAAAAAAAAAAAAAAAAACbd7_qGzsimCWq0_4yBbPYe25as9stUymI8NU-ZJFFT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=yY7lINJGkF8P8gUG9jAZZvK5sN0%3D)

GG.PNG가 안열리고 다른 파일보다 swp이라는 파일도 있다.

![SWP File](https://blog.kakaocdn.net/dna/Q7W71/btsHxZx05Fk/AAAAAAAAAAAAAAAAAAAAAE8bEHdWQKB9gFgbOtI8wyNhYn9nY_6leanAN3GUCvte/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=CKHb0TMhRMGC6PiyXcQEKGh30cs%3D)

zip key가 있다.

![Zip Key](https://blog.kakaocdn.net/dna/tpHDB/btsHxTdJj3V/AAAAAAAAAAAAAAAAAAAAAEZ-VnEAtwqphZU-bKQwI7ZWadN01TIQbQYfmkiWu2M3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=xe8Okw9rsezBdnex7lwnNclN0Yw%3D)

._flag에는 더미값이 있었다.

![Dummy Value](https://blog.kakaocdn.net/dna/coR4o0/btsHwOLdQmj/AAAAAAAAAAAAAAAAAAAAANg1GgPgtXHNz52NyEMiYiE184bsIrgmHUlNlJU9YFTt/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=ttfRsxP7xAxDlQgdgAgr0CCGNcI%3D)

FINISH 파일에 존재한다.