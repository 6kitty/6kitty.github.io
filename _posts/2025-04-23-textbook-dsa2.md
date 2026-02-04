```
---
layout: post
title: "Textbook-DSA2"
categories: [Algorithm]
tags: [dsa, signature, security, python, cryptography]
last_modified_at: 2025-04-23
---

```python
#!/usr/bin/env python3
from Crypto.Util.number import getPrime, inverse, bytes_to_long, isPrime
from random import randrange, choices
from hashlib import sha1
import string

class DSA(object):
    def __init__(self):
        print('generating parameters...')
        while True:
            self.q = getPrime(160)
            r = randrange(1 << 863, 1 << 864)
            self.p = self.q * r + 1
            if self.p.bit_length() != 1024 or isPrime(self.p) != True:
                continue
            h = randrange(2, self.p - 1)
            self.g = pow(h, r, self.p)
            if self.g == 1:
                continue
            self.x = randrange(1, self.q)
            self.y = pow(self.g, self.x, self.p)
            self.k = randrange(1, self.q)
            break
    
    def update(self):
        self.k = randrange(1, self.q)

    def sign(self, msg):
        r = pow(self.g, self.k, self.p) % self.q
        h = bytes_to_long(msg) % self.q
        s = inverse(self.k, self.q) * (h + self.x * r) % self.q
        self.update()
        return (r, s)

    def verify(self, msg, sig):
        r, s = sig
        if s == 0:
            return False
        s_inv = inverse(s, self.q)
        h = bytes_to_long(msg) % self.q
        e1 = h * s_inv % self.q
        e2 = r * s_inv % self.q
        r_ = pow(self.g, e1, self.p) * pow(self.y, e2, self.p) % self.p % self.q
        if r_ == r:
            return True
        else:
            return False


dsa = DSA()
token = "".join(choices(string.ascii_letters + string.digits, k=32)).encode()

print("Welcome to dream's DSA server")
while True:
    print("[1] Sign")
    print("[2] Verify")
    print("[3] Get Info")

    choice = input()

    if choice == "1":
        print("Input message (hex): ", end="")
        msg = bytes.fromhex(input())
        if msg == token:
            print("Do not cheat !")
        else:
            print(dsa.sign(msg))

    elif choice == "2":
        print("Input message (hex): ", end="")
        msg = bytes.fromhex(input())
        if len(msg) > 100:
            print("Too long message")
        else:
            print("Input signagure (r, s as decimal integer): ", end="")
            sig = map(int, input().split(", "))
            if dsa.verify(msg, sig) == True:
                print("Signature verification success")
                if msg == token:
                    print(open("flag", "rb").read())
            else:
                print("Signature verification failed")

    elif choice == "3":
        print(f"p = {dsa.p}")
        print(f"q = {dsa.q}")
        print(f"g = {dsa.g}")
        print(f"y = {dsa.y}")
        print(f"token = {token}")

    else:
        print("Nope")
```

전자 서명 알고리즘은 DSA라고 함 
플래그 획득 조건: 2번 메뉴에서/ r,s 값을 받고 / msg, sig를 verify해서 True / 그리고 msg==token이면/ flag 프린트 
h값이 생성되는 방법:
```python
from pwn import *
from Crypto.Util.number import *

io = remote("host3.dreamhack.games",20726)

io.sendline(b"3")

def recv():
    io.recvuntil(f"= ".encode())
    return eval(io.recvline())

p, q, g, y, token = [recv() for _ in range(5)]

token_int = bytes_to_long(token)
forge_token = long_to_bytes(token_int + q)

io.sendline(b"1")
io.sendlineafter(b": ", bytes.hex(forge_token).encode())
r, s = eval(io.recvline())


io.sendline(b"2")
io.sendline(bytes.hex(token).encode())
io.sendline(f"{r}, {s}".encode())

io.recvuntil(b"success\n")

flag = eval(io.recvline()).decode()
io.close()

print(flag)
```