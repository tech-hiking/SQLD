# Part 2) SQL 기본

## 1. 관계형 데이터베이스 개요

### 1. 데이터 베이스

데이터베이스는 용도와 목적에 맞는 데이터들끼리 모아서 저장한다.

### 2. 관계형 데이터베이스

RDB(Relational Database)라고 불리는 관계형 데이터베이스는 모든 데이터를 2차원 테이블 형태로 표현한 뒤 각 테이블 간의 관계를 정의한다.

### 3. TABLE

컬럼: `속성`에 해당

로우: `인스턴스`에 해당

### 4. SQL(Structured Query Language)

데이터베이스가 이해할 수 언어인 SQL. 복잡한 SQL은 어떻게 작성하느냐에 따라 성능 차가 확연히 드러나기 때문에 SQL을 잘 작성하고 튜닝하는 것이 매우 중요하다.

## 2. SELECT 문

### 1. SELECT

데이터 조회 명령어

- `*` 사용 시 전체 `컬럼` 조회
- `WHERE` 절이 없는 경우 전체 `row` 조회
- 별칭을 쓰면 테이블명을 짧게 쓸 수 있다.

<aside>
⚠️

테이블 명에 Alias를 쓴 경우, 테이블명 대신 Alias를 **꼭** 사용해야 한다!

</aside>

### 2. 산술 연산자

| 연산자        | 의미                      | 우선 순위 |
| ------------- | ------------------------- | --------- |
| ()            |                           | 1         |
| \*            |                           | 2         |
| /             | (0으로 나누면 에러)       | 2         |
| +             |                           | 3         |
| -             |                           | 3         |
| %(SQL Server) | 나머지(0으로 나누면 NULL) | 3         |

<aside>
⚠️

다른 컬럼끼리의 연산에 NULL이 포함되어 있는 경우 결과값은 `NULL`이다!

</aside>

### 3. 합성 연산자

`||` 문자와 문자를 연결할 때 사용한다.

```sql
SELECT 'S' || 'Q' || 'D' AS SQL FROM DUAL;
-- SQL
```

## 3. 함수

### 1. 문자 함수 \*[]는 옵션

1. CHR(ASCII 코드): 아스키 코드. `CHR(65) → A`
2. LOWER(문자열): 문자열을 소문자로 변환. `LOWER('JENNIE') -> jennie`
3. UPPER(문자열): 문자열을 대문자로 변환. `UPPER('jennie') -> JENNIE`
4. LTRIM(문자열, [특정문자]): 왼쪽에서 trim. `LTRIM('     SQL') -> SQL`
5. RTRIM(문자열, [특정문자]): 오른쪽에서 trim. `LTRIM('SQL       ') -> SQL`
6. TRIM([위치][특정문자][FROM] 문자열): 양쪽에서 trim.
   1. 위치 `LEADING` `TRAILING` `BOTH`
   2. `TRIM('   SQL       ') -> SQL`
   3. `TRIM(LEADING 'S' FROM 'SQL') -> QL`
7. SUBSTR(문자열, 시작점, [길이]): 문자열 잘라서 반환해주는 함수.
   1. 길이를 명시하지 않는 경우, 시작부터 끝까지 반환.
   2. `SUBSTR(’블랙핑크제니’, 3, 2) → 핑크`
   3. `SUBSTR(’블랙핑크제니’, 3, 4) → 핑크제니`
8. LENGTH(문자열): 길이 반환. `LENGTH(’블랙핑크’) → 4`
9. REPLACE(문자열, 변경 전 문자열, [변경 후 문자열]): 문자열에서 문자열을 찾아 변경해주는 함수.
   1. 변경 후 문자열이 없는 경우, 문자열을 제거한다.
10. LPAD(문자열, 길이, 문자): 문자열이 설정한 길이가 될 때까지 왼쪽을 채우는 함수.
    1. `LPAD(’SQLD’, 10, ‘V’) → VVVVVVSQLD`

### 2. 숫자함수

1. ABS(수): 절대값 반환.
2. SIGN(수): 양수 1, 음수 -1, 0이면 0 반환.
3. ROUND(수, [자릿수]): 소수점 자릿수까지 반올림하여 반환. 자릿수가 음수일 경우 지정된 정수부를 반올림하여 반환함.
   1. `ROUND(100.76, 1) → 100.8`
   2. `ROUND(163.76, -2) → 200`
4. TRUNC(수, [자릿수]): 버림. 자릿수가 음수일 경우 지정된 정수부에서 버럼하여 반환함.
   1. `TRUNC(54.29, 1) ⇒ 54.2`
   2. `TRUNC(54.29, -1) → 50`
5. CEIL(수): 소수점 이하의 수를 올림한 정수 반환
   1. `CEIL(72.86) → 73`
   2. `CEIL(-33.4) → -33`
6. FLOOR(수): 소수점 이하의 수를 버림한 정수 반환
   1. `FLOOR(22.3) → 22`
   2. `FLOOR(-22.3) → -23`
7. MOD(수1, 수2): 수1을 수2로 나눈 나머지를 반환. 단, 수2가 0일 경우 수1을 반환.
   1. MOD 함수의 두 인자값이 모두 음수이면 나머지도 그대로 음수로 도출된다.
      1. `MOD(-15, -4) ⇒ -3`
   2. `MOD(15, 7) → 1`
   3. `MOD(15, -4) → 3`

### 3. 날짜 함수

1. SYSDATE: 연월일시분초를 반환해주는 함수
   1. `SYSDATE⇒ 2024-05-04 15:26:03`
2. EXTRACT(특정 단위 FROM 날짜 데이터)
   1. `EXTRACT(YEAR FROM SYSDATE) → 2025`
   2. `EXTRACT(MONTH FROM SYSDATE) → 5`
   3. `EXTRACT(DAY FROM SYSDATE) → 4`
3. ADD_MONTHS(날짜 데이터, 특정 개월 수): 날짜 데이터에서 특정 개월 수를 더한 날짜를 반환.
   1. 기준 날짜에 일자가 존재하지 않는 경우, 해당 월의 마지막 일자가 반환됨.
   2. `ADD_MONTHS(TO_DATE(’2025-12-31’, ‘YYYY-MM-DD’), -1) → 2025-11-30`
   3. `ADD_MONTHS(TO_DATE(’2025-12-31’, ‘YYYY-MM-DD’), 1) → 2026-01-31`

### 4. 변환 함수

SQL 성능과 에러를 피하기 위해서는 되도록 명시적 형변환을 사용하자.

1. 명시적 형변환과 암시적 형변환
   1. **명시적 형변환**: 변환 함수를 사용하여 데이터 유형 변환을 명시적으로 나타냄
   2. **암시적 형변환**: 데이터베이스가 내부적으로 알아서 데이터 유형을 변환함
2. 명시적 형변환에 쓰이는 함수
   1. TO_NUMBER(숫자인 문자열): 문자열을 숫자형으로 반환
   2. TO_CHAR(수 or 날짜, [포맷]): 수나 날짜형을 문자형으로 반환
      1. `TO_CHAR(SYSDATE, ‘YYYYMMDD HH24MISS’) → 20250504 223321`
   3. TO_DATE(문자열, 포맷): 문자형의 데이터를 날짜형으로 변환해주는 함수
      1. `TO_DATE(’20250504’, ‘YYYYMMDD’) → 2025-05-04`

### 5. NULL 관련 함수

1. **NVL(인수1, 인수2)**: 인수1의 값이 NULL인 경우 인수2를 반환하고 아닐 경우 인수1을 반환해주는 함수
   1. NVL(SCORE, 0) → 점수 데이터가 NULL일 경우 0을 반환하고, NULL이 아닐 경우 점수 반환
2. **NULLIF(인수1, 인수2)**: 인수1과 인수2가 같으면 NULL을 반환하고 같지 않으면 인수1을 반환
   1. NULLIF(SCORE, 0) → SCORE가 0일 경우 NULL, 아닐 경우 SCORE 반환
3. **SOALESCE(인수1, 인수2, 인수3 …)**: NULL이 아닌 최초의 인수를 반환해주는 함수
4. **NVL2(인수1, 인수2, 인수3)**: 인수1이 NULL이 아닌 경우 인수2를 반환하고 NULL인 경우 인수3을 반환
   1. NVL2(REVIEW, ‘리뷰있음’, ‘리뷰없음’) → 리뷰 존재하면 ‘리뷰있음’ 리뷰 NULL이면 ‘리뷰없음’ 반환

### 6. CASE

조건 처리 함수

CASE WHEN ~~ END로 구성됨. Oracle에선 DECODE도 같은 기능이다.

## 4. WHERE 절

INSERT를 제외한 DML문을 수행할 때 원하는 데이터만 골라 수행할 수 있도록 해주는 구문.

- `SELECT COL1, COL2 FROM TABLE WHERE 조건`
- `UPDATE TABLE SET 컬럼명 = 새로운 데이터 WHERE 조건`
- `DELETE FROM TABLE WHERE 조건`

### 1. 비교 연산자

`=, <, <=, >=, >`

### 2. 부정 비교 연산자

| 연산자       | 의미      | 예시               |
| ------------ | --------- | ------------------ |
| !=           | 같지 않음 | where col != 10    |
| ^=           | 같지 않음 | where col ^= 10    |
| <>           | 같지 않음 | where col <> 10    |
| not 컬럼명 = | 같지 않음 | where not col = 10 |
| not 컬럼명 > | 같지 않음 | where not col > 10 |

<aside>
⚠️

논리 연산자는 SQL에 명시된 순서와 관계없이 () → NOT → AND → OR 순으로 처리된다.

</aside>

<aside>
⚠️

조건식에서 컬럼명은 일반적으로 좌측에 위치하지만 우측에 위치해도 정상적으로 동작한다.

</aside>

### 3. SQL 연산자

| 연산자             | 의미                              | 예시                       |
| ------------------ | --------------------------------- | -------------------------- |
| BETWEEN A AND B    | A와 B의 사이(A,B포함)             | where col between 1 and 10 |
| LIKE ‘비교 문자열’ | %는 문자열, \_는 하나의 문자 의미 | where col like ‘방탄%’     |

where col like ‘%소년단’
where col like ‘방\_소%’ |
| IN (LIST) | LIST 중 하나와 일치 | where col in (1, 3, 5) |
| IS NULL | NULL 값 | where col is null |

- LIKE에서 _와 %의 문자를 그대로 쓰려면 `_@` `#%` 을 쓰면 사용할 수 있다.

### 4. 부정 SQL 연산자

| 연산자              | 의미                          | 예시                           |
| ------------------- | ----------------------------- | ------------------------------ |
| NOT BETWEEN A AND B | A와 B 사이가 아님(A,B 미포함) | where col not between 1 and 10 |
| NOT IN (LIST)       | LIST 중에 일치하는 것이 없음  | where col not in (1,3,5)       |
| IS NOT NULL         | NULL 값이 아님                | where col is not null          |

### 5. 논리 연산자

**논리 연산자는 SQL에 명시된 순서와 관계없이 `() → NOT → AND → OR` 순으로 처리된다.**

| 연산자 | 의미                            | 예시                       |
| ------ | ------------------------------- | -------------------------- |
| AND    | 모든 조건이 TRUE여야 함         | where col > 1 and col < 10 |
| OR     | 하나 이상의 조건이 TRUE여야 함  | where col = 1 or col = 10  |
| NOT    | TRUE면 FALSE이고 FALSE이면 TRUE | where not col > 10         |

## 5. GROUP BY, HAVING 절

### 1. GROUP BY

기준이 되는 컬럼을 넣어서 데이터를 그룹별로 묶을 수 있다.

### 2. 집계 함수

데이터를 그룹별로 나누면 그룹별로 집계 데이터를 도출하는 것이 가능하다.

| COUNT(\*)            | 전체 row를 count하여 반환                               |
| -------------------- | ------------------------------------------------------- |
| COUNT(컬럼)          | 컬럼값이 null인 row를 제외하고 count하여 반환           |
| COUNT(DISTINCT 컬럼) | 컬럼값이 null이 아닌 row에서 중복을 제거하여 count 반환 |
| SUM(컬럼)            | 컬럼값들의 합계                                         |
| AVG(컬럼)            | 컬럼값들의 평균                                         |
| MIN(컬럼)            | 컬럼럼값들의 최솟값                                     |
| MAX(컬럼)            | 컬럼값들의 최댓값                                       |

### 3. HAVING

GROUP BY의 조건절이다. 마치 where를 사용하는 것과 같다.

**⭐️⭐️⭐️⭐️ SELECT문의 논리적 수행 순서**

```sql
SELECT     5
FROM       1
WHERE      2
GROUP BY   3
HAVING     4
ORDER BY   6
```

<aside>
⚠️

집계 함수에 대한 조건절은 HAVING 절을 이용하여 작성해야 하고, 논리적으로 HAVING절은 SELECT 절보다 먼저 수행되기 때문에 SELECT 절에서 정의한 Alias를 사용할 수 없다.

</aside>

- 118p 문제

  정답 3

![image.png](attachment:c00a4791-e280-404e-8ba6-7508ffc568b2:image.png)

## 6. ORDER BY 절

데이터를 정렬함. 논리적으로 맨 마지막에 수행된다.

- ASC(Ascending): 오름차순
- DESC(Descending): 내림차순
- - 옵션 생략 시 ASC가 기본값이 된다.

<aside>
⚠️

정렬의 기준이 되는 컬럼에 NULL 데이터가 있는 경우, 데이터베이스 종류에 따라 정렬 위치가 다르다.

Oracle의 경우 NULL을 최댓값으로 취급해서 오름차순의 경우 맨 마지막에 위치한다.(SQL Server는 반대). 만약 순서를 변경하고 싶다면 ORDER BY 절에 NULLS FIRST, NULLS LAST 옵션을 써서 NULL의 정렬상 순서를 변경할 수 있다.

</aside>

## 7. JOIN

### 1. JOIN이란?

각기 다른 테이블을 한번에 보여줄 때 사용.

### 2. EQUI JOIN

Equal(=) 조건으로 JOIN 하는 것.

### 3. Non EQUI JOIN

Equal(=) 조건이 아닌 다른 조건(between, >, >=, <. <=)으로 JOIN 하는 것.

### 4. 3개 이상 TABLE JOIN

3개 이상의 테이블도 JOIN이 가능하다.

```sql
SELECT A.PRODUCT_NAME, B.MEMBER_ID, B.CONTENT, C.EVENT_NAME
FROM PRODUCT A,
		 PRODUCT_REVIEW B,
		 EVENT C
WHERE A.PRODUCT_CODE = B.PRODUCT_CODE
  AND B.REG_DATE BETWEEN C.START_DATE ADN C.END_DATE;
```

<aside>
⚠️

JOIN되는 두 테이블에 모두 존재하는 컬럼의 경우, 컬럼명 앞에 반드시 테이블명 또는 Alias를 명시해주어야 한다.

</aside>

### 5. OUTER JOIN

(+) 기호를 붙여주면 LEFT JOIN, RIGHT JOIN 처럼 출력할 수 있다.

(+)는 **데이터가 없을 수 있는 쪽(=NULL이 허용될 테이블)** 에 붙인다.

## 8. STANDARD JOIN

벤더마다 SQL 문법이 다를 경우 호환성 이슈가 발생해 ANSI SQL을 지정하게 되었다.(표준 조인)

### 1. INNER JOIN

JOIN 조건이 충족하는 데이터만 출력

```sql
SELECT
FROM
INNER JOIN
ON
```

### 2. OUTER JOIN

JOIN 조건에 충족하는 데이터가 아니어도 출력될 수 있는 방식

1. `LEFT OUTER JOIN`: 왼쪽 테이블 데이터는 무조건 출력되는 방식(JOIN될 값이 없어도 표시된다)
2. `RIGHT OUTER JOIN`: 오른쪽 테이블 데이터는 무조건 출력되는 방식(없는 값은 NULL로 표기된다)
3. `FULL OUTER JOIN`: 왼쪽, 오른쪽 테이블의 데이터가 모두 출력되는 방식

   <aside>
   ⚠️

   FULL OUTER JOIN은 (+) 로 는 할 수 없다. (+) 은 left, right 한쪽만 가능함!!

   </aside>

### 3. NATURAL JOIN

A 테이블과 B 테이블에서 같은 이름을 가진 컬럼들이 모두 동일한 데이터를 가지고 있을 경우 JOIN이 되는 방식. SQL Server(MSSQL)에서는 지원하지 않는다.

- 기본적으로 모든 컬럼이 같아야 한다. 하지만, Oracle에서는 USING 조건절을 이용하여 같은 이름을 가진 컬럼 중 원하는 컬럼만 JOIN에 이용할 수 있다.(해당 컬럼만 같으면 row 출력됨)

```sql
SELECT SUM(COL1)
  FROM SAMPLE1 A JOIN SAMPLE2 B
  USING (COL1); // col1만 같으면 다 출력함.(Oracle이라고 가정)
```

<aside>
⚠️

join에서 using 절을 사용하는 경우 using 절로 정의된 컬럼 앞에는 별도의 테이블명이나 alias를 표기할 수 없다
—

USING (공통컬럼)은 **두 테이블에 모두 존재하는 동일한 이름의 컬럼을 기준으로 JOIN**합니다. 그리고 JOIN 이후 **그 공통 컬럼은 결과 테이블에서 하나로 통합되기 때문에**, 더 이상 **어느 테이블 소속인지 명시할 필요도 없고, 명시하면 문법 오류가 납니다.**

</aside>

```sql
-- ✅ 맞음
SELECT emp_id, name
FROM EMPLOYEE E
JOIN DEPARTMENT D
USING (dept_id);

-- ❌ 틀림
SELECT E.dept_id  -- ❌ 오류 발생!
FROM EMPLOYEE E
JOIN DEPARTMENT D
USING (dept_id);

-- 💡 ON절 사용 시엔 가능
SELECT E.dept_id, D.dept_id  -- 가능
FROM EMPLOYEE E
JOIN DEPARTMENT D
ON E.dept_id = D.dept_id;
```

| **JOIN 방식** | **컬럼명 사용 방식**             |
| ------------- | -------------------------------- |
| USING 절      | 공통 컬럼에는 테이블명/alias ❌  |
| ON 절         | 컬럼마다 alias 붙여 사용 가능 ✅ |

<aside>
⚠️

NATURAL JOIN에는 ON 절을 쓸 수 없다.

</aside>

### 4. CROSS JOIN

A 테이블과 B 테이블 사이에 JOIN 조건이 없는 경우, 조합할 수 있는 모든 경우를 출력하는 방식이다. Cartesian Product라고 부르기도 한다.

```sql
SELECT A.NAME,
       A.JOB,
       B.DRINK_CODE,
       B.DRINK_NAME
  FROM ENTERTAINER A CROSS JOIN DRINK B;
```

- `p 147 문제`

  ![image.png](attachment:7bab7dd0-411d-4186-9b45-7dfb627be50d:image.png)
