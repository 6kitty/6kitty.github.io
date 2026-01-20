---
layout: post
title: "assembly quiz 정리"
categories: [SWING, Self-study]
tags: []
last_modified_at: 2024-01-22
---

1-1. rbx+8부터 rax -> 0xC0FFEE  
1-2. rbx+8의 주소 -> 0x401a48  
1-3. 원래 rax 0x31337+555555554016주소에 있는 값 3 -> 0x3133a  
1-4. 2번째 줄 rcx=0x4, 0x3133a에서 32 참조 값 3133A 빼기 0  
1-5. inc rax는 +=1  
1-6. f면 그대로 출력, ~8까지 출력  
1-7. f면 그대로 출력, 9~부터 출력  
1-8. 0이면 그대로 출력, 모두 출력  
1-9. 1과 f xor->e, 4와 e xor-> 0100 1110 -> 1010, 10이면 a  
1-10. xor의 역연산은 xor  
1-11. 반전 eax, cafebabe 합해서 15가 되도록 맞춰주기  
1-12. 대충 0x30과 xor 계산하고 이걸 rcx가 0x19가 될 때까지 반복하는 어셈블리  
잔머리를 굴려서 welcome의 세번째 l과 to의 o와 assembly의 e 정도만 비교  
각각 자리의 값 0x5c, 0x5f, 0x55를 0x30과 xor 연산  
0101 1100, 0101 1111, 0101 0101을 0011 0000과 xor 연산  
0110 1100, 0110 1111, 0110 0000  
0x6c, 0x6f, 0x65 l o e  
Welcome to assembly world! 로 유추 가능  

2-1.  
0x30으로 xor하므로 1의 자리(?)는 그대로  
6,5,1,4만 10의 자리에 옴  
각각을 3으로 xor  
0110, 0101, 0001, 0100을 0011로 xor  
0101, 0110, 0010, 0111이므로 5, 6, 2, 7  
57, 65, 6c, 63, 6f, 6d, 65, 20 Welcome  
74, 6f, 20, 61, 73, 73, 65, 6d to assem  
62, 6c, 79, 20, 77, 6f, 72, 6c bly worl  
64, 21 d!  

3-1.  
- call 앞부분  
rdi는 0x400500  
esi는 0xf  

- call 0x400497 <write_n>  
스택 프레임 할당  
rsi, DWORD PTR [rbp-0xc] , 0xf  
rdi, QWORD PTR [rbp-0x8], 0x400500  

edx값이 0xf  
rsi는 QWORD PTR [rbp-0x8]이므로 0x400500의 값  
리틀 엔디언이므로 오른쪽부터 읽기  
72 33 34 64 79 20 37 30 20 64 33 62 75 36 3f(총 rsi만큼 읽음, 0xf)  
r34dy 70 d3bu6?  
출력 syscall  
스택프레임 반환  

- call 뒷부분  
원래 함수 흐름으로 돌아옴  