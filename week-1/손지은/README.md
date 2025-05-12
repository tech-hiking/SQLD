# PART2. SQL 기본 및 활용

## CHAPTER1. SQL 기본

### 1. 관계형 데이터베이스

#### (1) 관계형 데이터베이스(RDB)

관계형 데이터 모델에 기초를 둔 데이터베이스

- 데이터베이스 : 용도와 목적에 맞는 데이터들끼리 모아 저장

- RDBMS: RDB를 관리/감독하기 위한 시스템(Oracle,SQL Server,MySQL,MariaDB,PostrgreSql등)

#### (2) 테이블

RDB는 모든 데이터를 2차원 테이블 형태로 표현한 뒤 각 테이블 간의 관계를 정의

- 데이터모델의 엔티티에 해당
- 컬럼(Column) : 세로 열, 속성
- 로우(Row) : 가로 행, 인스턴스
- 일반적으로 데이터베이스는 여러 개의 테이블로 구성됨

#### (3) SQL (Structured Query Language)

RDB에서 데이터를 다루기 위해 사용하는 언어

---

### 2. SELECT문

- 데이터 조회, 계산 결과 조회 등
- 조회되는 컬럼의 순서는 테이블의 컬럼 순서와 동일
- 논리 순서

```sql
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
```

    1. FROM
    2. WHERE
    3. GROUP BY
    4. HAVING
    5. SELECT
    6. ORDER BY

#### (1) 별칭(Alias)

- 테이블명에 Alias를 설정했을 경우 테이블명 대신 Alias를 사용해야함

```sql
-- AS 사용
SELECT e.name
FROM employees AS e
-- AS 생략
SELECT e.name
FROM employees e
```

#### (2) 산술연산자

- NUMBER나 DATE유형의 데이터와 같이 사용
- NULL이 포함된 경우 결과값은 NULL
- / : 0으로 나눌 경우 에러
- % (SQL Server) : 0으로 나눌 경우 NULL반환

```sql
SELECT COL1+COL2 AS R1 --R1컬럼에 연산 결과 표시
FROM SAMPLE
```

#### (3) 합성연산자 '||'

- 문자와 문자 연결

---

### 3. 함수

#### (1) 문자함수

1. ASCII코드변환 : CHR(ASCII)
   - SQL Server(MSSQL)일 경우 CHAR()
2. 대소문자 변환

- LOWER(str) : 소문자로 변환
- UPPER(str) : 대문자로 변환

3. 공백 제거

- LTRIM(str[,특정문자]) : 왼쪽 공백 제거, 특정문자를 명시했을 경우 왼쪽부터 비교하여 포함되었으면 제거/없으면 멈춤
- RTRIM(str[,특정문자]): 오른쪽 공백 제거, 특정문자를 명시했을 경우 오른쪽부터 비교하여 포함되었으면 제거/없으면 멈춤
- TRIM([위치][특정문자][FROM]str) : 옵션이 없을 경우 왼쪽과 오른쪽 공백 제거
  - 위치 옵션 : LEADING / TRAILING / BOTH
  - 특정문자는 한개만 지정할 수 있음
- MSSQL은 공백제거만 가능

4. str 자르기

- SUBSTR(str,시작점[,길이]) : 원하는 부분만 잘라서 반환, 길이가 없을 때는 시작부터 끝까지 반환

5. str 길이 : LENGTH(str)
6. str 변경

- REPLACE(str, 변경 전 str[,변경 후 str]) : 변경 전 str을 변경 후 str로 바꿔주는 함수, 변경 후 str이 없으면 변경 전 str 제거

7. str 채우기

- LPAD(str, len, 문자) : 설정한 길이가 될 때까지 왼쪽을 특정문자로 채움
- RPAD(str, len, 문자) : 오른쪽을 특정문자로 채움
- 원래 str 길이가 더 길면 지정길이 만큼만 잘려서 출력

#### (2) 숫자함수

1. 부호

- ABS(n) : 절댓값
- SIGN(n) : 1(양수), -1(음수), 0

2. 자릿수

- 공통 옵션 (자릿수)
  - 자릿수가 없을 경우 0
  - 자릿수가 음수일 경우 지정된 정수부를 반올림
- ROUND(n[,자릿수]) : 반올림
- TRUNC(n[,자릿수]) : 버림
- CEIL(n) : 소수점 이하 수 올림
  - MSSQL일 경우 CEILING(str)
- FLOOR(n) : 소수점 이하 수 버림

3. 나머지 반환 : MOD(n1,n2)

- n1를 n2로 나눈 나머지 반환
- n2가 0일 경우 n1 그대로 반환
- 둘다 음수면 음수로 반환

#### (3) 날짜함수

- SYSDATE : 현재 YYYY-MM-DD HH:MI:SS 반환
  - nls_date_format에 따라 출력 양식이 달라질 수 있음
  - MSSQL경우 GETDATE()
- EXTRACT(특정단위 FROM 날짜데이터) : 특정단위만 출력
  - 단위 : YEAR, MONTH, DAY, HOUR, MINUTE, SECOND
  - MSSQL경우 DATEPART(특정단위, 날짜데이터)

```sql
SELECT EXTRACT(YEAR FROM SYSDATE) AS YEAR
FROM DUAL;
```

- ADD_MONTHS(날짜데이터,특정 개월 수) : 특정 개월수를 더한 날짜를 반환
  - 기준 날짜의 일자가 존재하지 않으면 해당 월의 마지막 일자가 반환
  - MSSQL경우 DATEADD(MONTH,특정개월수,날짜데이터)

```sql
SELECT ADD_MONTH(DATE'2022-01-31',1) FROM DUAL; --결과 : 2022-02-28
```

#### (4) 변환함수

- 암시적 형변환 : 내부적으로 알아서 데이터 유형 변환
  - ex)VARCHAR와 NUMBER를 비교할 경우 자동으로 NUMBER로 변경
- MSSQL경우 CONVERT 또는 CAST함수로 형변환

1. 숫자형 변환

- TO_NUMBER(str): 문자열을 숫자로 변환, 숫자가 아닐 경우 ERR

2. DATE 변환

- TO_CHAR(NUM or DATE[,포맷]) : 수나 날짜형 데이터를 포맷 형식의 문자형으로 변환
- TO_DATE(문자열, 포맷) : 포맷 형식의 문자열을 날짜형으로 변환
- 포맷 : YYYY, MM, DD, HH(시12), HH24(시24), MI, SS

#### (5) NULL 관련 함수

- NVL(a,b) : a가 null일 경우 b반환, 아닐 경우 a반환
  - MSSQL : ISNULL(a,b)
- NULLIF(a,b) : a와 b가 같으면 NULL, 아니면 a
- COALESCE(a,b,c,,,) : NULL이 아닌 최초의 인수 반환
- NVL2(a,b,c) : a가 null이 아닌 경우 b를 반환, null인 경우 c를 반환

#### (6) CASE 구문

- '~이면 ~이고, ~이면 ~이다' 구문

```sql
SELECT SUBWAY_LINE, --콤마로 구분
    CASE WHEN SUBWAY_LINE='1' THEN 'BLUE'
         WHEN SUBWAY_LINE='2' THEN 'GREEN'
         ELSE 'GRAY' --ELSE가 없을 경우 NULL이 DEFAULT
    END AS LINE_COLOR
  FROM SUBWAY_INFO;
```

- Oracle의 DECODE함수와 같은 기능

```sql
SELECT SUBWAY_LINE,
    DECODE(SUBWAY_LINE,'1','BLUE','2','GREEN') AS LINE_COLOR
  FROM SUBWAY_INFO;
```

---

### 4. WHERE절

- INSERT를 제외한 DML문(데이터 조작어, SELECT,INSERT,UPDATE,DELETE)을 수행할 때 원하는 데이터만 골라 수행 = 조건에 맞는 행만 출력
- WHERE절이 없으면 테이블 전체 ROW가 조회
- 조건식에서 컬럼명은 좌/우 모두 가능
- 우선순위 : 산술->연결->비교->IN/LIKE/BETWEEN/IS NULL->NOT->AND->OR

#### (1) 비교연산자

- =, <, ,<=, >, >=

```sql
SELECT FIRST_NAME,EMAIL
 FROM MEMBER
WHERE FIRST_NAME="MARK"; -- 인용부호없이 MARK로 할 경우 ERR
```

- 부정비교 : !=, ^=, <>, not 컬럼= , not 컬럼>

#### (2) 논리연산자

- NOT, AND, OR (이 순서대로 우선순위)

#### (3) SQL 연산자

- (NOT) BETWEEN A AND B : A,B포함 A,B사이 (NOT일경우 A,B사이가 아님 / A,B미포함)
- LIKE '비교 문자열'
  - % : 문자열
  - \_ : 하나의 문자
  - ESCAPE
  ```sql
  WHERE COL LIKE 'M%s' --M으로 시작 s로 끝나는 행
  WHERE COL LIKE '%101%' -- 101이 포함된 행
  WHERE COL LIKE '%#%%' ESCAPE '#' --%가 포함된 행
  ```
- (NOT) IN(LIST) : LIST중 하나와 일치 (OR을 이용하여 표현 가능)
- IS (NOT) NULL : NULL 값

---

### 5. GROUP BY

- 데이터를 그룹별로 묶을 수 있도록 해주는 절
- 뒤에 그룹핑의 기준이 되는 컬럼이 옴
- 집계함수가 없는 일반 컬럼이라면 SELECT 컬럼과 같아야 함

#### (1) 집계 함수

- 데이터를 그룹별로 나눈 후 그룹별 집계 데이터를 도출하는 함수
- COUNT(컬럼) : NULL을 제외하고 COUNT, \*일 경우 전체 ROW COUNT
- COUNT(DISTINCT 컬럼): NULL이 아닌 ROW에서 중복 제거한 COUNT
- SUM(컬럼) : 합계
- AVG(컬럼) : NULL제외한 평균
- MIN(컬럼) : 최소
- MAX(컬럼) : 최대

#### (2) HAVING

- 데이터를 그룹핑한(GROUP BY) 후 특정 그룹을 골라낼 때 사용
- WHERE 사용이 가능하면 최대한 사용 후 HAVING사용 => GROUP BY는 비용이 많이 드는 작업임

```sql
-- 2021년 1학기 수학, 평균점수가 90점 이상인 학생(Oracle)
SELECT STUDENT_NO,
    AVG(MATH_SCORE) AS AVG -- 3)그룹별로 계산한 값을 하나의 행으로 출력
  FROM STUDENT_SCORE
 WHERE YEAR='2021'
  AND SEMESTER='1' -- 1)필터링된 원본 데이터 집합
 GROUP BY STUDENT_NO --2) 남은 데이터를 학생번호별로 묶어서 그룹핑
HAVING AVG(MATH_SCORE)>=90; -- 4) 그룹단위의 조건에 맞는 학생 출력
```

### 6. ORDER BY

- 논리적으로 맨 마지막 수행
- SELECT한 데이터 정렬
- 명시하지 않으면 임의의 순서대로 출력
- 한개 이상의 기준 컬럼 적용 가능
- 옵션 가능(ASC, DESC), ASC가 기본값
- Oracle은 NULL이 최댓값, MSSQL은 NULL이 최솟값 => NULLS FIRST(LAST)옵션 사용 가능

```sql
SELECT GRADE, NAME
    FROM MEMBER
    ORDER BY GRADE DESC, NAME DESC;
```

### 7. JOIN

- 두 개 이상의 테이블을 연결하여 데이터를 출력하는 쿼리
- EQUI, NON EQUI JOIN 하나의 쿼리에서 사용 가능
- 테이블 간 PK, FK 연관 관계가 없어도 JOIN 가능
- JOIN되는 두 테이블에 모두 존재하는 컬럼이면 테이블명이나 ALIAS 명시해야함

#### (1) EQUI JOIN

- EQUAL(=)을 조건으로 하는 JOIN
- SELECT 컬럼이 같으면 다른 테이블 컬럼으로 대체 가능 (A.COL1 -> B.COL1)

```sql
SELECT A.PRODUCT_CODE,A.PRODUCT_NAME,B.MEMBER_ID,B.CONTENT,B.REG_DATE
FROM PRODUCT A, PRODUCT_REVIEW B
WHERE A.PRODUCT_CODE=B.PRODUCT_CODE --EQUI JOIN
```

#### (2) Non EQUI JOIN

- EQUAL이 아닌 다른 조건으로 JOIN
- SELECT 컬럼 다른 테이블 컬럼으로 대체 불가능 (JOIN이래도 서로 다른 값을 가질 수 있음)

```sql
WHERE B.REG_DATE BETWEEN A.START_DATE AND A.END_DATE;
```

#### (3) 3개 이상 TABLE JOIN

```sql
WHERE A.PRODUCT_CODE=B.PRODUCT_CODE
AND B.REG_DATE BETWEEN C.START_DATE AND C.END_DATE;
```

#### (4) OUTER JOIN

- JOIN에 만족하지 않은 행도 출력되는 쿼리
- Oracle에서는 모든행이 출력되는 테이블의 반대편 테이블 옆에 (+) 기호 붙이기 (한개만 붙일 수 있음)

```sql
WHERE A.PRODUCT_CODE=B.PRODUCT_CODE(+)--리뷰가 없는 PRODUCT도 모두 출력(A가 모두 출력);
```

### 8. STANDARD JOIN

- ANSI SQL: RDBMS 벤더 사이에 표준 SQL
- STANDARD JOIN : ANSI SQL 중 하나인 JOIN
- 명시적 조인

#### (1) INNER JOIN

- JOIN 조건에 충족되는 데이터만 출력
- ON 에 조건 작성

```sql
FROM PRODUCT A INNER JOIN PRODUCT_REVIEW B
 ON A.PRODUCT_CODE=B.PRODUCT_CODE;
```

#### (2) OUTER JOIN

- JOIN 조건에 충족하는 데이터가 아니어도 출력
- LEFT(RIGHT) OUTER JOIN - ON : 왼쪽(오른쪽)테이블은 무조건 모두 출력
- FULL OUTER JOIN : 왼쪽, 오른쪽 테이블의 데이터 모두 출력
- 데이터가 없는 ROW들은 NULL로 출력
- 조건과 상관없이 OUTER기준 테이블은 모두 출력되고, WHERE절로 필터링 가능

```sql
/*
이벤트에 응모한 횟수가 10회 이상인 회원정보
1. 이벤트 응모 테이블에 회원번호 존재
    - 회원 INNER JOIN 이벤트 응모
    - 회원 RIGHT OUTER JOIN 이벤트 응모
    (회원만 응모가 가능하다면 둘은 동일)
2. 이벤트 응모 테이블에 저장된 ROW가 10건 이상인 회원 : COUNTA(A.회원번호)>=10
*/

SELECT A.이름, A.핸드폰번호
FROM 회원A INNER JOIN 이벤트 응모 B
ON A.회원번호=B.회원번호
GROUP BY A.이름, A.핸드폰번호 --SELECT와 같은 컬럼
HAVING COUNT(A.회원번호)>=10
```

#### (3) NATURAL JOIN

- A,B테이블에서 같은 이름을 가진 컬럼들이 모두 동일한 데이터를 가지고 있을 경우만 JOIN
- ON절 사용 X
- MSSQL에서는 지원 X
- Oracle에서는 USING 조건절로 원하는 컬럼만 JOIN가능

```sql
FROM A NATURAL JOIN B;

--USING
SELECT CAST,GENDER, A.JOB, B.JOB
FROM A JOIN B
USING (CAST,GENDER) --USING으로 정의된 컬럼 앞에는 별도의 ALIAS나 테이블병을 붙이지 않음
```

#### (4) CROSS JOIN

- A,B 테이블 사이에 JOIN조건이 없는 경우, 조합할 수 있는 모든 경우를 출력
- Cartesian Product (카티션 곱)
- A테이블(3), B테이블(4)일 경우 총 12행 테이블 생성
