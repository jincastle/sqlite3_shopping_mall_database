# sqlite3_shopping_mall_database

## 하면서 익힌것들

### csv에 대해 다시 공부(읽고,쓰고, 인코딩, reader)

## sqlite3

> 
> 
> - `sqlite3` 는 `SQLite` 데이터 베이스를 사용하는데 필요한 인터페이스 파이썬 표준 라이브러리
> - 파이썬 설치 시 SQLite 가 함께 자동으로 설치됨
> - SQLite는 주로 개발용이나 소규모 프로젝트에서 사용하는 파일 기반의 가벼운 데이터베이스
> - 개발 시에는 SQLite를 사용하여 빠르게 개발하고 실제 운영 시스템에서는 좀 더 규모 있는 데이터베이스를 사용하는 것이 일반적
> - [GUI 설치 페이지](https://sqlitebrowser.org/dl/)

하지만 저는 brew로 관리 할려고 명령어로 설치

```python
brew install --cask db-browser-for-sqlite
```

## 버전 확인

```python
import sqlite3

# 버전 확인
sqlite3.version
sqlite3.sqlite_version
```

## 데이터 베이스 만들기

```python
conn = sqlite3.connect('test.db')
```

| 칼럼 이름 | 칼럼 타입 | 설명 |
| --- | --- | --- |
| ID | INTEGER | 고유 번호 (PRIMARY KEY) |
| PRODUCT_NAME | TEXT | 상품명 |
| PRICE | TEXT | 가격 |

## 테이블 생성하기

```python
# 커서 생성 # sqlite를 할려면 항상 생성이 필요
c = conn.cursor()

# 쿼리문 작성
query = '''CREATE TABLE test (ID INTEGER PRIMARY KEY, PRODUCT_NAME TEXT, PRICE INTEGER)'''
c.execute(query)

# DB 연결 해제
conn.close()
```

error

```python
---------------------------------------------------------------------------
OperationalError                          Traceback (most recent call last)
Cell In [9], line 3
      1 # 쿼리문 작성
      2 query = '''CREATE TABLE test (ID INTEGER PRIMARY KEY, PRODUCT_NAME TEXT, PRICE INTEGER)'''
----> 3 c.execute(query)

OperationalError: table test already exists
```

해결법

기존 테이블이 존재해서 발생하는 에러, 기존껄 삭제하거나 아니면 기존꺼 사용하면 됨

```python
DROP TABLE test;
SELECT name FROM sqlite_master WHERE type='table' AND name='test';
```

## 데이터 불러오기, 조회

- sqlite3 에서 데이터 조회 방법에는 `fetchone()`, `fetchmany()`, `fetchall()` 3 가지 방법을 사용
- `SELECT` 문을 사용한 조회 결과 범위에서 실제 가져오는 row 수를 결정

| 메서드 | 내용 |
| --- | --- |
| fetchone() | 조회 결과에서 1개의 row 를 가져옴 |
| fetchmany(size=2) | 조회 결과에서 지정한 size 만큼 row 를 가져옴 |
| fetchall() | 조회 결과에서 모든 row 를 가져옴 |

```python
import sqlite3
import pandas as pd

# 연습용 DB 연결
conn = sqlite3.connect("chadwick.db")

# 커서 생성
c = conn.cursor()

# 전체 테이블 현황 조회
c.execute('''SELECT * FROM sqlite_master WHERE type="table"''')

# 보기 편하게 pandas 사용
pd.DataFrame(c.fetchall())

# Parks 데이터 조회
c.execute('''SELECT * FROM Parks''')

# 전체 로우 선택
pd.DataFrame(c.fetchall())

# Parks 데이터 조회
c.execute('''SELECT * FROM Parks''')

# 1개 로우 선택
pd.DataFrame(c.fetchone())

# Parks 데이터 조회
c.execute('''SELECT * FROM Parks''')

# 지정 로우 선택 3개만 가져오기
pd.DataFrame(c.fetchmany(size=3))

```

조회 및 불러와서 그중 하나를 불러오는 방식

## 특정 데이터 조회

```python
# 컬럼(키) 확인
cur = c.execute('''SELECT * FROM Parks''')
cur.description

# 특정 row 만 가져오기 # ?에 NY가 들어간거
c.execute('''SELECT * FROM Parks WHERE state = ?''', ('NY',))
pd.DataFrame(c.fetchall())
```

## 데이터 다루기

- `SELECT` : 데이터 선택
- `INSERT` : 데이터 삽입
- `UPDATE` : 데이터 수정
- `DELETE` : 데이터 삭제

## 데이터 삽입(INSERT)

### ****INSERT 방법 1. 열(키) 항목 순서를 정확히 알고 있는 경우****

```python
# 데이터 삽입
c.execute("INSERT INTO test VALUES(1,'모자',150000)")

conn.commit() # 커밋을해야 데이터가 들어가네요

# 데이터 삽입
c.execute("INSERT INTO test VALUES(2,'코트',200000)")
conn.commit()
```

### ****INSERT 방법 2. 열(키) 항목 순서를 정확히 모르는 경우****

```python
# ?를 이용해서 값을 집어 넣어서 자동매칭이 되네!!!!
# 데이터 삽입
c.execute("INSERT INTO test(PRODUCT_NAME, PRICE, ID) VALUES(?,?,?)", ('티셔츠', 20000, 3))
conn.commit()

# 데이터 삽입
c.execute("INSERT INTO test(ID, PRICE, PRODUCT_NAME) VALUES(?,?,?)", (4, 55000, '블라우스'))
conn.commit()
```

****INSERT 방법 3. 여러 데이터를 한번에 삽입하고 싶은 경우****

```python
# 기존 내용을 삭제하고
c.execute("DELETE FROM test")
conn.commit()

# 추가할 상품 리스트
product_list = [[1, '모자', 15000],
                [2, '코트', 200000],
                [3, '티셔츠', 20000],
                [4, '블라우스', 55000],
                [5, '가디건', 45000],
                [6, '청바지', 50000],
                [7, '구두', 150000],
                [8, '가방', 170000]]

# 데이터 여러줄 삽입
c.executemany("INSERT INTO test(ID, PRODUCT_NAME, PRICE) VALUES(?,?,?)", product_list)
conn.commit()
```

## 데이터 수정(UPDATE)

### ****UPDATE 방법 1. 튜플 형태로 수정****

```python
c.execute("UPDATE test SET PRODUCT_NAME = ? WHERE ID = ?", ('슬랙스', 6))
conn.commit()
```

### ****UPDATE 방법 2. 딕셔너리 형태로 수정****

```python
c.execute("UPDATE test SET PRICE = :price WHERE ID = :id", {"price":55000, "id":6})
conn.commit()
```

### ****UPDATE 방법 3. %s 표시자 사용****

```python
c.execute("UPDATE test SET PRODUCT_NAME = '%s' WHERE ID = '%s'" % ('트랜치코트', 2))
conn.commit()
```

## 데이터 삭제(DELETE)

### ****DELETE 방법 1. 튜플 형태로 삭제****

```python
c.execute("DELETE FROM test WHERE ID =?", (8,))
conn.commit()
```

### ****DELETE 방법 2. 딕셔너리 형태로 삭제****

```python
c.execute("DELETE FROM test WHERE PRODUCT_NAME = :product_name", {'product_name':'슬랙스'})
conn.commit()
```

### ****DELETE 방법 3. 전체 삭제****

```python
# test는 테이블 명
c.execute("DELETE FROM test")
conn.commit()
```

## 데이터 백업

### iterdump

- 데이터 베이스를 백업할 때 사용하는 모듈
- `.sql` 파일 확장자로 테이블을 다시 복원할 수 있는 쿼리문을 저장

```python
import sqlite3

# 연습용 DB 연결
conn = sqlite3.connect("test.db")

# 커서 생성
c = conn.cursor()

# 추가할 상품 리스트
product_list = [[1, '모자', 15000],
                [2, '코트', 200000],
                [3, '티셔츠', 20000],
                [4, '블라우스', 55000],
                [5, '가디건', 45000],
                [6, '청바지', 50000],
                [7, '구두', 150000],
                [8, '가방', 170000]]

# 데이터 여러줄 삽입
c.executemany("INSERT OR REPLACE INTO test(ID, PRODUCT_NAME, PRICE) VALUES(?,?,?)", product_list)
conn.commit()

# iterdump 내용 확인
for line in conn.iterdump():
    print(line)

# 데이터 베이스 백업 파일 생성
with conn:
    with open('backup.sql', 'w') as f: # backup.sql 파일로 백업
        for line in conn.iterdump():
            f.write('%s\n' % line)
        print('Completed.')

# 확인을 위한 삭제
# 테이블 삭제
c.execute("DROP TABLE test")
conn.commit()

# 백업 SQL 파일 로딩
with open('backup.sql', 'r') as sql_file:
    sql_script = sql_file.read()

# 데이터 확인
sql_script

# SQL 스크립트 실행 및 복구
c.executescript(sql_script)
conn.commit()
```