---
layout: post
title: "sql 시스템 테이블 정리"
categories: [Self-study]
tags: [SQL, Database, System Tables]
last_modified_at: 2023-12-31
---

[WHA-S] ExploitTech: system table fingerprinting

1. System tables  
시스템 테이블: 설정 및 계정 정보, 테이블과 컬럼 정보, 실행되고 있는 쿼리 정보  
SQL injection을 통해 시스템 테이블에서 테이블과 컬럼명 획득 가능  
ex) 게시판에서 SELECT 구문으로 게시물 조회하는 창이 있다면?  
시스템 테이블 조회 쿼리를 작성하여 DB 정보 획득  

2. MySQL  
스키마 = 데이터베이스  
information_schema, mysql, performance_schema, sys  

스키마 정보: TABLES 테이블 사용  
```sql
mysql> select TABLE_SCHEMA from information_schema.tables group by TABLE_SCHEMA;
```

테이블 정보: TABLES 테이블 사용  
```sql
mysql> select TABLE_SCHEMA, TABLE_NAME from information_schema.TABLES;
```

컬럼 정보: COLUMNS 테이블 사용  
```sql
mysql> select TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME from information_schema.COLUMNS;
```

실시간 실행 쿼리 정보: PROCESSLIST 테이블 사용  
```sql
mysql> select * from information_schema.PROCESSLIST;
```

실시간 실행 쿼리 정보(with. 실행 중인 계정): SYS 데이터베이스의 SESSION 테이블 사용  
```sql
mysql> select user, current_statement from sys.session;
```

DBMS 계정 정보: USER_PRIVILEGES 테이블 사용  
```sql
mysql> select GRANTEE, PRIVILEGE_TYPE, IS_GRANTABLE from information_schema.USER_PRIVILEGES;
```

DBMS 계정 정보: MYSQL 데이터베이스의 USER 테이블 사용  
```sql
mysql> select User, authentication_string from mysql.user;
```

3. MSSQL  
master, tempdb, model, msdb  

데이터베이스 정보: SYSDATABASES 테이블 사용  
```sql
SELECT name FROM master..sysdatabases;
```

데이터베이스 정보: DB_NAME(num) 형식 사용  
인자로 전달되는 숫자 중 0은 현재 DB  
```sql
SELECT DB_NAME(1);
```

테이블 정보: SYSOBJECTS 테이블 사용  
```sql
SELECT name FROM dreamhack..sysobjects WHERE xtype='U';
```
*xtype='U'는 이용자 정의 테이블을 뜻함  

테이블 정보: INFORMATION_SCHEMA 스키마의 TABLES 테이블 사용  
```sql
SELECT table_name FROM dreamhack.information_schema.tables;
```

컬럼 정보: SYSCOLUMNS 테이블 사용  
```sql
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'users');
```
서브쿼리를 사용하여 조건 적용  

컬럼 정보: INFORMATION_SCHEMA 스키마의 COLUMNS 테이블 사용  
```sql
SELECT table_name, column_name FROM dreamhack.information_schema.columns;
```

DBMS 계정 정보: master 데이터베이스의 sys.sql_logins 테이블 사용  
```sql
SELECT name, password_hash FROM master.sys.sql_logins;
```

DBMS 계정 정보: master 데이터베이스의 syslogins 테이블 사용  
```sql
SELECT * FROM master..syslogins;
```

4. PostgreSQL  
postgres, template1, template0  

스키마 정보:  
주요 정보를 담고 있는 테이블을 포함 스키마: pg_catalog, information_schema  
```sql
postgres=$ select nspname from pg_catalog.pg_namespace;
```

테이블 정보: information_schema 사용  
```sql
postgres=$ select table_name from information_schema.tables where table_schema='pg_catalog';
```
```sql
postgres=# select table_name from information_schema.tables where table_schema='information_schema';
```

DBMS 계정 정보: pg_catalog.pg_shadow 테이블 사용  
```sql
postgres=$ select usename, passwd from pg_catalog.pg_shadow;
```

DBMS 설정 정보: pg_catalog.pg_settings 테이블 사용  
```sql
postgres=$ select name, setting from pg_catalog.pg_settings;
```

실시간 실행 쿼리 확인: pg_catalog.pg_stat_activity 테이블 사용  
```sql
postgres=$ select usename, query from pg_catalog.pg_stat_activity;
```

테이블 정보: information_schema.tables 테이블 사용  
```sql
postgres=$ select table_schema, table_name from information_schema.tables;
```

컬럼 정보: information_schema.columns 테이블 사용  
```sql
postgres=$ select table_schema, table_name, column_name from information_schema.columns;
```

5. Oracle  
데이터베이스 정보  
all_tables: 현재 사용자가 접근할 수 있는 테이블 집합  
```sql
SELECT DISTINCT owner FROM all_tables;
```
```sql
SELECT owner, table_name FROM all_tables;
```

컬럼 정보: all_tab_columns  
```sql
SELECT column_name FROM all_tab_columns WHERE table_name = 'users';
```

DBMS 계정 정보: all_users 테이블 사용  
```sql
SELECT * FROM all_users;
```

6. SQLite  
sqlite_master 시스템 테이블  
```sql
sqlite> .header on -- 콘솔 실행 시 컬럼 헤더 출력을 위한 설정
sqlite> open dreamhack.db -- 데이터베이스 연결
sqlite> select * from sqlite_master;
```