SQL
===

### SQL

- 데이터베이스를 액세스하고 조작하기 위한 표준 언어

### RDBMS

- 관계형 데이터베이스 관리 시스템

---

# SQL 구문

```
SELECT * FROM Customers;
```

> Customers 테이블의 모든 레코드를 선택한다

<br>
` SQL 키워드는 대소 문자를 구분하지 않는다. select는 SELECT와 같다.`

---

### SQL에서 세미콜론

- 일부 데이터베이스 시스템에서는 각 SQL 문의 끝에 세미콜론이 필요하

- 세미콜론은 데이터베이스 시스템에서 각 SQL 문을 분리하여 서버에 대한 동일한 호출에서 둘 이상의 SQL 문을 실행할 수있게하는 표준 방법이다

---

### 중요한 SQL명령 

- SELECT - 데이터베이스에서 데이터를 추출합니다.
- UPDATE - 데이터베이스의 데이터를 업데이트합니다.
- DELETE - 데이터베이스에서 데이터를 삭제합니다.
- INSERT INTO - 새로운 데이터를 데이터베이스에 삽입합니다.
- CREATE DATABASE - 새 데이터베이스를 만듭니다.
- ALTER DATABASE - 데이터베이스를 수정합니다.
- CREATE TABLE - 새 테이블을 만듭니다.
- ALTER TABLE - 테이블을 수정합니다.
- DROP TABLE - 테이블을 삭제합니다.
- CREATE INDEX - 색인 (검색 키)을 생성합니다.
- DROP INDEX - 색인을 삭제합니다.

---

### SELECT 구문

- 데이터베이스에서 데이터를 선택하는 데 사용한다
- 리턴 된 데이터는 result-set 이라는 result table에 저장된다

```
SELECT column1, column2, ...
FROM table_name;
```

> column1, 2는 테이블의 필드 이름이다
> `SELECT * FROM table_name;`을 사용하면 모든 필드를 선택한다

---

### SELECT DISTINCT 구문

- 고유 한 (다른) 값만 리턴할때 사용한다
- 테이블내에서 열은 종종 많은 중복 값들이 있는데, 이때 중복 되는 값들을 빼고 서로 다른 값들만 나열할때 사용한다

```
SELECT DISTINCT Country FROM Customers;
```

중복을 제외한 서로 다른 값들의 숫자를 셀수 있다

```
SELECT COUNT(DISTINCT Country) FROM Customers;
``` 

---

### WHERE 구문

- 레코드를 필터링할 때 사용한다
- 지정된 조건을 충족하는 레코드만 추출한다

```
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

```
SELECT * FROM Customers
WHERE Country='Mexico';
```

WHERE절에 있는 연산자

Operator|Description
-----------|-------------
=	|Equal
<>	|Not equal. Note: In some versions of SQL this operator may be written as !=
>	|Greater than
<	|Less than
>=	|Greater than or equal
<=	|Less than or equal
BETWEEN	|Between an inclusive range
LIKE	|Search for a pattern
IN	|To specify multiple possible values for a column

---

### AND, OR, NOT 연산자

- WHERE 절은 AND, OR, NOT 연산자와 결합할 수 있다
- AND, OR 연산자는 둘 이상의 조건에 따라 레코드를 필터링 할때 사용한다
- NOT 연산자는 조건이 참이 아닌 경우 레코드를 표시한다

```
SELECT * FROM Customers
WHERE City='Berlin' OR City='München';

SELECT * FROM Customers
WHERE NOT Country='Germany';

SELECT * FROM Customers
WHERE NOT Country='Germany';

SELECT * FROM Customers
WHERE Country='Germany' AND (City='Berlin' OR City='München');

SELECT * FROM Customers
WHERE NOT Country='Germany' AND NOT Country='USA';
```

---

### ORDER BY KEYWORD

- 결과를 오름차순(ascending) 또는 내림차순(descending)으로 정렬할때 사용한다
- 기본적으로 오름차순이고, `DESC`를 쓰면 내림차순이다

```
SELECT * FROM Customers
ORDER BY Country;

SELECT * FROM Customers
ORDER BY Country DESC;

SELECT * FROM Customers
ORDER BY Country ASC, CustomerName DESC;
```

---

### INSERT INTO 구문

- 테이블에 새 레코드를 삽입할때 사용한다

```
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);

INSERT INTO table_name
VALUES (value1, value2, value3, ...);
```

> 열이름을 직접 지정하거나 모든 열에 값을 추가한다

```
INSERT INTO Customers (CustomerName, ContactName, Address, City, PostalCode, Country)
VALUES ('Cardinal', 'Tom B. Erichsen', 'Skagen 21', 'Stavanger', '4006', 'Norway');
```

--- 

### NULL 값

- NULL값이 있는 필드는 값이 없는 필드이다
- 테이블의 필드가 선택적이면이 필드에 값을 추가하지 않고 새 레코드를 삽입하거나 레코드를 업데이트 할 수 있다. 그런 다음 필드가 NULL 값으로 저장된다 

> 참고 : NULL 값이 0 값 또는 공백이있는 필드와 다르다 NULL 값을 가진 필드는 레코드를 생성하는 동안 비어있는 필드이다

<br>
NULL값을 테스트 하는법

- `IS NULL` 또는 `IS NOT NULL`

```
SELECT column_names
FROM table_name
WHERE column_name IS NULL;

SELECT column_names
FROM table_name
WHERE column_name IS NOT NULL;
```

---

### UPDATE 구문

- 기존 레코드를 수정할때 사용한다

```
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;

UPDATE Customers
SET ContactName='Juan'
WHERE Country='Mexico';
```

> **WHERE** 절은 갱신해야하는 레코드를 지정한다 **WHERE** 절을 생략하면 테이블의 모든 레코드가 업데이트된다

---

### DELETE 구문

- 테이블의 기존 레코드를 삭제할때 사용한다

```
DELETE FROM table_name
WHERE condition;

DELETE FROM Customers
WHERE CustomerName='Alfreds Futterkiste';
```

테이블을 삭제하지 않고 테이블의 모든 행을 지우는법

```
DELETE FROM table_name;

DELETE * FROM table_name;
```

---

#### TOP, LIMIT, ROWNUM 구문

SELECT TOP 구문

- 리턴 할 레코드 수를 지정할때 사용한다
- 수천 개의 레코드가 있는 큰 테이블에서 유용하다
- MySQL에서는 LIMIT, Oracle에서는 ROWNUM으로 사용한다

SQL Server / MS 액세스 구문 :

```
SELECT TOP number|percent column_name(s)
FROM table_name
WHERE condition;

SELECT TOP 3 * FROM Customers;
```

MySQL 구문 :

```
SELECT column_name(s)
FROM table_name
WHERE condition
LIMIT number;

SELECT * FROM Customers
LIMIT 3;
```

오라클 구문 :



```
SELECT column_name(s)
FROM table_name
WHERE ROWNUM <= number;

SELECT * FROM Customers
WHERE ROWNUM <= 3;
```

SQL TOP PERCENT 구문

테이블에서 처음 50%의 레코드를 선택:

```
SELECT TOP 50 PERCENT * FROM Customers;
```

