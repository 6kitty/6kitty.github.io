---
layout: post
title: "[4주차] PE 구조 분석 & notepad.exe PE 헤더 분석, 가상 메모리 구조 분석"
categories: [Digital Forensics]
tags: [PE, notepad.exe, 구조 분석, 가상 메모리]
last_modified_at: 2024-04-30
---

### PE파일이란?
리눅스 실행파일이 ELF인 것처럼 윈도우에서 사용하는 실행파일입니다.

### PE파일의 종류
EXE, DLL, OBJ 등 윈도우OS에서 돌아가는 실행파일들이 있습니다.

### PE파일의 구조
PE 파일은 아래 그림처럼 구조체 형식으로 저장되어 있습니다. 크게는 PE header와 body가 존재합니다.

![PE 파일 구조](https://blog.kakaocdn.net/dna/cGlTU3/btsG0GzZHXj/AAAAAAAAAAAAAAAAAAAAAACcCh3ywWwEmBYtBGUKLm3EBbC-Fj0FaslZenrYK1jd/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=7dkY1RC0VOsfJNLe%2B%2BJYctpx6ik%3D)

메모장을 PEview로 열면 구조화되어 있는 걸 알 수 있습니다.

#### 1. PE header
**1. DOS Header**

![IMAGE_DOS_HEADER 구조체](https://blog.kakaocdn.net/dna/dbO6tY/btsGZZNr7Kj/AAAAAAAAAAAAAAAAAAAAAIqCU_iKgIN1LIVhsPaiHmrHLutbbpvAA4QN9xXEPB8d/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Y7v01%2BUeX95an7jKptvsebbwE64%3D)

IMAGE_DOS_HEADER 구조체
1. e_magic: DOS signature이다. PE 파일이라는 걸 나타낸다. MZ로 총 2바이트. 위 메모장에서도 data가 5A4D로 고정됨을 알 수 있습니다.
2. e_lfanew: NT header가 시작되는 위치 offset입니다.

**2. DOS stub(옵션)**

![DOS stub](https://blog.kakaocdn.net/dna/TW5uK/btsGZZGFhEU/AAAAAAAAAAAAAAAAAAAAAFufYur66d3iguMUheCxxkIWVvOEhCSMRYnv9zbr7AFz/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=3BGGWjrnvlrboe2b6KEy3vAeEyk%3D)

DOS환경에서 실행되는 코드+데이터가 적혀있습니다. 위를 DOS환경에서 실행하면 "This program cannot be run in DOS mode"를 출력하고 종료합니다.

**3. NT Header**

![IMAGE_NT_HEADERS 구조체](https://blog.kakaocdn.net/dna/dDYgQj/btsGZXB6JsL/AAAAAAAAAAAAAAAAAAAAADMhI4SckKCm0SXjGR0QoIj60-Aj2I6oqEm_ZEQITwms/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=SeMI8KAkrRlJhhQahPO9wwkb8ys%3D)

IMAGE_NT_HEADERS 구조체는 파일 실행에 필요한 정보를 저장합니다(중요).
1. signature: data를 읽으면(참고로 리틀엔디안) 00004550인데 이것은 PE를 뜻합니다.
2. file header: 파일의 간략 정보를 나타냅니다. IMAGE_FILE_HEADER 구조체를 갖습니다.

| 주요 멤버명          | 의미                          | notepad.exe                          |
|-------------------|-----------------------------|-------------------------------------|
| Machine           | 실행되는 아키텍처               | 8664는 AMD64를 나타냅니다.               |
| NumberOfSections  | 파일에 존재하는 섹션 개수         | 7개입니다.                           |
| SizeOfOptionalHeader | Optional Header의 크기         | 0x00F0만큼 차지                     |
| Characteristics   | 속성                          | bit OR계산을 해보면 <br/>0x0002 EXECUTABLE_IMAGE <br/>0x0020 LARGE_ADDRESS_AWARE를 갖는 걸 알 수 있습니다. |
| TimeDateStamp     | 파일의 생성 일시                | value를 보면 2007/06/20에 생성된 걸 알 수 있습니다. |

3. optional header: 파일 실행에 필요한 정보들을 저장합니다.

![Optional Header](https://blog.kakaocdn.net/dna/cbLDi9/btsG6hSJKi5/AAAAAAAAAAAAAAAAAAAAAH5AaYTXadjAtUdpRzpBNbBRl8T3QuSNDw5u89PjBFdL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hvUqu23X5jgvTBaep2rMaeF%2Bm78%3D)

위 파일 헤더에 적혀있던 것처럼 0x00F0만큼 갖습니다.

| 주요 멤버명          | 의미                          |
|-------------------|-----------------------------|
| Magic             | 32비트 or 64비트               |
| AddressOfEntryPoint | 로더가 실행을 시작할 주소 <br/> 처음으로 실행될 코드 주소 |
| ImageBase         | 메모리 상의 시작 주소           |
| SectionAlignment, FileAlignment | 섹션의 최소단위           |
| SizeOfImage       | Image 전체 크기               |
| SizeOfHeader      | PE header의 전체 크기          |
| Subsystem         | 동작환경 정의                  |
| NumberOfRvaAndSize | datadirectory배열의 개수       |
| DataDirectory     | IMAGE_DATA_DIRECTORY 구조체의 배열 <br/><br/> 디렉토리별 각각의 정보 작성 |

**4. Section Header**

![Section Header](https://blog.kakaocdn.net/dna/ciDHce/btsG3fn0y9K/AAAAAAAAAAAAAAAAAAAAAHm9OloolTL3NE5LxDTJKQOzeuk4WFkAE7W_FhjiBzsu/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=3QllUHJpTRLQrOat9VPccrMXbhs%3D)

아래서 설명할 Section에 대한 header입니다. 메모리에 로드될 때 참조됩니다. 섹션의 자세한 .text, .rdata, .data등은 후술하고 주요 멤버만 짚고 넘어갑니다.
1. VirtualSize: 메모리에서 섹션 크기
2. VirtualAddress: 메모리에서 섹션의 시작주소(RVA라고 합니다)
3. SizeOfRawData: 파일에서 섹션의 크기
4. PointerToRawData: 파일에서 섹션의 Offset(길이)
5. Characteristics: 섹션의 속성(위에서 본 멤버와 같은 원리)

### 2. PE body
body는 여러 개의 섹션으로 구성되어 있습니다.

| code 영역 | .text | 실행 코드를 담고 있는 섹션 |
|-----------|-------|--------------------------|
| data 영역 | .data | 읽고 쓰기가 가능한 데이터 섹션 <br/> 초기화된 전역변수와 정적 변수 등 위치 |
|           | .rdata | 읽기만 가능한 데이터 섹션 <br/> 상수형 변수, 문자열 상수 등 위치 |
|           | .bss   | 초기화되지 않은 전역변수 위치 |
| import API | .idata | import할 DLL, API 정보 섹션 <br/> IAT 존재 |
|           | .didat | Delay-loading import DLL 정보 섹션 |
| export API | .edata | export할 DLL 정보 섹션 |
| Resource    | .rsrc | 리소스 관련 데이터 정보 섹션 |
| relocation   | .reloc | 재배치 정보 섹션 |
| TLS         | .tls   | 스레드 지역 저장소 |
| Debugging   | .debug$p | 미리 컴파일된 헤더가 있을 시 |

notepad.exe에는 위와 같은 섹션이 존재합니다. 중요한 code, data 영역 헤더를 살펴보겠습니다.

![.text 섹션 헤더](https://blog.kakaocdn.net/dna/dkhIYG/btsG1xQdmgQ/AAAAAAAAAAAAAAAAAAAAAFcV9uiLj5OVsnxRPZFFTiUKZ_IiOTfbUHPaEyAGKAm6/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=GVpbLa%2FXCkh2%2FZSLMcrgVmJxdeI%3D)

위는 .text의 섹션 헤더입니다. characteristics를 보면 실행, 읽기 권한이 있습니다.

![.data와 .rdata 섹션](https://blog.kakaocdn.net/dna/bysyKh/btsG3mUQxMF/AAAAAAAAAAAAAAAAAAAAANuYEjSJOTRjozdCPnmPMNP6SzOYWIHjIih_rtacIkni/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=y5qwiZYGp6TI%2FX8Q1sHPiJlAb1M%3D)

왼쪽이 .data이고, 오른쪽이 .rdata입니다. 속성 차이를 보면 왼쪽은 읽기, 쓰기 권한이 있지만 오른쪽은 읽기 권한만 존재합니다. rdata에는 문자열 상수 등이 위치합니다.

### 가상 메모리 구조 분석
위 그림과 같이 메모리에 PE파일이 로드됩니다. 왼쪽 파일에서는 offset의 개념을 쓰지만 오른쪽 메모리에서는 RVA(relative virtual address)의 개념을 차용합니다. RVA는 시작 주소에 대한 상대적 VA를 말합니다. 이 RVA를 이용한 VA 식은 다음과 같습니다.

| VA = ImageBase + RVA |

image_file_header 뒷부분 IMAGE_OPTIONAL_HEADER 구조체에 메모리 로드 관련 멤버들이 존재합니다. 아까 위에서 설명했던 optional header에서는 주요 멤버만 설명하고 넘어갔는데 좀 더 자세히 설명해보겠습니다.

![IMAGE_OPTIONAL_HEADER 구조체](https://blog.kakaocdn.net/dna/cbLDi9/btsG6hSJKi5/AAAAAAAAAAAAAAAAAAAAAH5AaYTXadjAtUdpRzpBNbBRl8T3QuSNDw5u89PjBFdL/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=hvUqu23X5jgvTBaep2rMaeF%2Bm78%3D)

| 주요 멤버명          | 의미                          |
|-------------------|-----------------------------|
| Magic             | 실행되는 아키텍처               |
| SizeOfCode        | .text의 크기                  |
| AddressOfEntryPoint | 로더가 실행을 시작할 주소 <br/> 처음으로 실행될 코드 주소 |
| ImageBase         | 메모리 상의 시작 주소           |
| SectionAlignment, FileAlignment | 섹션의 최소단위           |
| SizeOfImage       | Image 전체 크기 -> 위 alignment의 배수 |
| SizeOfHeader      | PE header의 전체 크기 -> 위 alignment의 배수 |
| Subsystem         | 동작환경 정의                  |
| NumberOfRvaAndSize | datadirectory배열의 개수       |
| stack, heap 사이즈 관련 멤버 | 프로세스마다 가지는 자신만의 스택, 힙 <br/> 그것들을 생성하기 위한 크기, 속성 지정 |
| DataDirectory     | IMAGE_DATA_DIRECTORY 구조체의 배열 <br/><br/> 디렉토리별 각각의 정보 작성 |

각 섹션별 RVA와 size가 정의되어 있습니다.