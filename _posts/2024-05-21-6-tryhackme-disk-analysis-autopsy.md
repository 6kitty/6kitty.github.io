---
layout: post
title: "[6주차] TryHackMe : Disk Analysis & Autopsy"
categories: [Self-study, Writeup]
tags: [TryHackMe, Disk Analysis, Autopsy, Forensics]
last_modified_at: 2024-05-21
---

파일을 열면 metadata에 md5 해시가 있다.

![Metadata MD5 Hash](https://blog.kakaocdn.net/dna/6fCvz/btsHvZylesB/AAAAAAAAAAAAAAAAAAAAAG7HxZad8c3cN_74lpwQKrQhJy6JWaMjyn6sZWrzVttw/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=POOYK0hFbmxkUXe0LyWG4CcZC94%3D)

operating system information에 가면 desktop name이 있다.

![Operating System Information](https://blog.kakaocdn.net/dna/GLnpC/btsHv9PpDOQ/AAAAAAAAAAAAAAAAAAAAAHZ7pjaDJymFdDdrLr_ytcr7HVz3U2uRjRMI1kz-rY1w/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=%2FQR64lI6%2FB7a4JQ1UMpxnWuzgcM%3D)

그 아래 Operating system user account를 가면 users가 있다.

![Operating System User Account](https://blog.kakaocdn.net/dna/bKDq3h/btsHxgtqXUX/AAAAAAAAAAAAAAAAAAAAAHE5n13uhBtjiZVuWDgXKfrU13RW553vS6yi7BhK1PBi/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Qru7KaKwkzK8bHY29UyfW2Mk2xc%3D)

data accessed 순으로 정렬하면 맨 아래 sivapriya이다.

![Data Accessed](https://blog.kakaocdn.net/dna/cbKP8u/btsHxeh5idg/AAAAAAAAAAAAAAAAAAAAAGaYLpr4LzosKWpViHuwX2goTdzDgS3T9UJnWk__RN1l/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=NMDeesjwnWKePKYLrRAl%2B2k%2FAlY%3D)

Look@Lan이라는 프로그램에서 network를 모니터링 한다. 이부분에 LANIP와 LANNIC(MAC)주소가 있다.

![Network Monitoring](https://blog.kakaocdn.net/dna/bgK49M/btsHwttiOUa/AAAAAAAAAAAAAAAAAAAAADBdRvDd5U3UGul-m4c5lEhb7jeY4K-CmVhVyDEUprdK/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=O1G0Zz5hFL%2FCppTWdeBskv%2FwxDU%3D)

programs 목록을 열람하여 Look@LAN 확인.

![Programs List](https://blog.kakaocdn.net/dna/Insa7/btsHxGFacvf/AAAAAAAAAAAAAAAAAAAAAJtzXNb1lkxLi3n0RPI7u6eNpWWfcYZStPN48kJWFN06/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Y0Z9YEewNP4Ha6X0AcKuOGWOni0%3D)

shreya/AppData/Roaming/Microsoft/Windows/Powershell을 열람하여 readline에서 history를 확인한다.

![PowerShell History](https://blog.kakaocdn.net/dna/cxTBfd/btsHw5eoFZF/AAAAAAAAAAAAAAAAAAAAAEIYMrSL-9MR3vre0hk0Y57rZ1lbQ9nAdnInBznPK2BJ/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=vc9Wrfr1iCEHZ2HEUKw9i23U6bE%3D)

mimikatz zip파일을 더블클릭 하면 다음과 같다. kiwi_passwords.yar에 author name이 있다.

![Mimikatz](https://blog.kakaocdn.net/dna/oAmBP/btsHwqwyCBg/AAAAAAAAAAAAAAAAAAAAALhD2tVjpZPIjM3Fr-m2VvyCEfYOqj9DfnGYpNZDlJuW/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uWlqIg%2BSI7Me5RWScBNDUNfOq5k%3D)

recent documents를 가면 zerologon Ink 파일이 답이라고 한다. 이게 왜 domaincontroller를 exploit 할 수 있는지는 모르겠다...

![Recent Documents](https://blog.kakaocdn.net/dna/bBaSik/btsHxKm5Bvk/AAAAAAAAAAAAAAAAAAAAAEkMvq8DV0d8bNOb3G72zrfjhmfNVVg7C7HWvk_efsC-/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=H1s%2B5vBPsY1AaF4nAIG5Xu1Iwh0%3D)