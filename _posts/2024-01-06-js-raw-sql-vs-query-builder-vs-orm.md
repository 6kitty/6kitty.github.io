---
layout: post
title: "[JS] raw SQL vs query builder vs ORM"
categories: [CS & Development]
tags: [JavaScript, SQL, ORM, Query Builder]
last_modified_at: 2024-01-06
---

자바스크립트와 DB 간의 상호작용 방식 

1. 데이터베이스 드라이버: 클라이언트와 커넥터 사용하여 직접 상호작용 **SQL 인젝션 취약점**
2. 쿼리 빌더: DB 클라이언트보다 한 단계 높은 계층에서 동작, 자바스크립트 코드로 쿼리 데이터 생성 -> 데이터베이스와 상호작용 
3. ORM: 데이터베이스를 객체 형식으로 다룰 수 있게 해주는 툴킷 **원시 쿼리 기능 등으로 SQL 인젝션 취약점**

자바스크립트와 DB 상호작용 시 사용할 수 있는 방법: raw SQL, query builder, ORM 

### 1. raw SQL 
native SQL이라고도 함, 가장 낮은 수준의 상호 작용 방식 
데이터베이스 언어로 수행 작업을 데이터베이스에 전송 

예제) 사용자는 책에 대한 데이터 열람 가능
every book has exactly one author, but every author might have an arbitrary number of books. 
저자 페이지에서는 authors.id 제공
해당 작가가 작성한 모든 책 목록을 위해서는...

```javascript
pstmt = con.prepareStatement("SELECT * FROM books WHERE author_id = %author_id");
ResultSet rs = pstmt.executeQuery(); 

while (rs.next())
{
	System.out.printIn("ID = " + rs.getInt(1) + ", NAME = " + rs.getString(2)); 
}
```

이 방식을 사용하면 다음과 같은 문제점 존재 
- SQL injection
- Typos in SQL commands
- Missing Editor Support 
- Typos in Table or Columns Names 
- Change management: 직접 마이그레이션 해야함.. 
- Query Extension 

### 2. Query Builder 
프로그래밍 언어로 작성, 네이티브 클래스 및 함수 사용하는 라이브러리임 
method chaining을 사용하는 객체 지향 인터페이스로 작성 

```javascript
query = Query.from_(books) \
				.select("*") \
                .where(books.author_id == aid);
```

ORM보다 사용 단순 
knex.js: 자바스크립트에서 사용하는 SQL 쿼리 빌더 

### 3. ORM 
테이블에 대한 개체(object) 생성 
다음과 같은 문제점 존재 
- over-fetching problem 
- JPA N+1, initial under-fetching 