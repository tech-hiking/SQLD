# PART2. SQL 기본 및 활용

## CHAPTER2. SQL 활용

### 1. 서브쿼리 (Subquery)

- 하나의 쿼리 안에 존재하는 또 다른 쿼리
- ORDER BY 절, INSERT문의 VALUE절 등에 사용 가능

```sql
SELECT A,(SELECT B FROM T) -- EX)스칼라 서브쿼리
FROM TB
```

#### (1) 스칼라 서브쿼리 (Scalar Subquery)

- 주로 SELECT절에 위치
- 컬럼이 올 수 있는 대부분의 위치에 사용 가능 (SELECT절, UPDATE문의 SET절, ORDER BY절), FROM절에는 X
- 하나의 값만 반환해야 함 (아니면 ERR)

#### (2) 인라인 뷰 (Inline View)

- 테이블명이 올 수 있는 위치에 사용 가능 (EX) FROM절

#### (3) 중첩 서브쿼리 (Nested Subquery)

- WHERE, HAVING 절에 사용 가능
- 메인 쿼리와의 관계에 따라 연관/비연관 서브쿼리로 분류

```sql
--연관 서브쿼리
SELECT NAME, JOB
 FROM ENTERTAIINER
WHERE AGENCY_CODE=(SELECT AGENCY_CODE FROM AGENCY WHERE AGENCY_NAME="A")
--비연관 서브쿼리
SELECT NAME, JOB
 FROM ENTERTAIINER E
WHERE AGENCY_CODE=(SELECT AGENCY_CODE FROM AGENCY A WHERE E.CODE=A.CODE)
```

- 반환 데이터 형태에 따라 단일행/다중행/다중컬럼 서브쿼리로 분류

  - 단일 행 : 1건 이하의 데이터 , 단일 행 비교연산자와 사용 (=,<,>,<=,>=,<>)
  - 다중 행 : 여러건의 데이터, 다중 행 비교연산자와 사용 (IN,ALL,ANY,SOME,EXISTS)
  - 다중 컬럼 : 여러 컬럼의 데이터 반환

    ```sql
    SELECT NAME, JOB
    FROM ENTERTAINER
    WHERE (AGENCY_CODE, JOB) IN (
    SELECT AGENCY_CODE, JOB
    FROM AGENCY_JOB_STANDARD
    );
    ```

---

### 2. 뷰 (View)

- 특정 SELECT문에 이름을 붙여 재사용이 가능하도록 저장한 오브젝트
- 실제 데이터를 저장하지 않고 해당 데이터를 조회해오는 SELECT문만 가진 가상 테이블
- 보안성, 독립성, 편리성(가독성을 높임)
- 수정(INSERT,UPDATE,DELETE) 가능 조건 : 단일테이블 기반, 기본키 포함, 집계함수 없는 경우

```sql
-- 생성
CREATE VIEW HIGH_SCORE_STUDENTS AS -- AS : 뒤에 오는 쿼리로 뷰를 만든다, 필수 키워드
SELECT 이름, 점수
FROM 학생
WHERE 점수 >= 90;

-- 사용
SELECT * FROM HIGH_SCORE_STUDENTS

-- 삭제
DROP VIEW HIGH_SCORE_STUDENTS

-- 추가 : 생성할 때 WITH CHECK OPTION를 붙이면 불가능
INSERT INTO HIGH_SCORE_STUDENTS (이름, 점수)
VALUES ('홍길동', 95);

-- 수정 : 조건(WHERE 등)을 위반하면 불가능
UPDATE HIGH_SCORE_STUDENTS
SET 점수 = 97
WHERE 이름 = '홍길동';

-- 내용 삭제
DELETE FROM HIGH_SCORE_STUDENTS
WHERE 이름 = '홍길동';
```

---

### 3. 집합 연산자

- 각 쿼리의 결과 집합을 가지고 연산하는 명령어

- 헤더는 첫 번째 쿼리를 따라감

#### (1) UNION / UNION ALL

- 각 쿼리의 결과를 그대로 합하는 것
- UNION ALL : 중복도 그대로 출력
- UNION : 중복 제거. 내부적으로 중복행을 제거해야해서 성능이 불리할 수 있음

```sql
SELECT * FROM RUNNING
UNION
SELECT * FROM INFINITE
```

#### (2) INTERSECT

- 공통 부분만 중복을 제거하여 출력

#### (3) MINUS / EXCEPT

- 앞에 쿼리 결과에서 뒤에 쿼리 결과를 제거하고 출력

---

### 4. 그룹 함수

데이터를 GROUP BY하여 나타낼 수 있는 데이터를 구하는 함수

#### (1) 집계 함수 (SUM, COUNT, AVG, MAX, MIN 등)

```sql
SELECT 부서, COUNT(*), AVG(급여)
FROM 직원
GROUP BY 부서;
```

#### (2) 소계(총계) 함수 (ROLLUP, CUBE, GROUPING SETS등)

(괄호를 하나의 덩어리로 보기)

- ROLLUP() : 소그룹 간의 소계 및 총계를 계산, 인수 순서에 따라 결과가 달라짐

  - (A) : A 그룹핑, 총합계
  - (A,B) : A+B그룹핑, A그룹핑, 총합계
  - (A,B,C) : A+B+C그룹핑, A+B그룹핑, A그룹핑, 총합계

- CUBE() : 소그룹 간의 소계 및 총계를 다차원적으로 계산, 인수 순서에 상관X

  - (A) : A 그룹핑, 총합계
  - (A,B) : A+B그룹핑, A그룹핑, B그룹핑, 총합계
  - (A,B,C) : A+B+C그룹핑, A+B그룹핑, B+C그룹핑, A+C그룹핑, A그룹핑, B그룹핑, C그룹핑, 총합계

- GROUPING SETS() : 특정 항목에 대한 소계를 계산, 인수 순서에 상관X

  - (A,B) : A 그룹핑, B 그룹핑
  - (A,B,()) : A 그룹핑, B 그룹핑, 총합계
  - (A,ROLLUP(B)) : A 그룹핑, B 그룹핑, 총합계

- GROUPING(기준이 되는 컬럼) : 소계 함수와 함께 쓰이며 소계를 나타내는 ROW를 구분할 수 있게 함

  - 그룹핑에 사용된 값이면 0 , 총계에 처리된 행(기존에 NULL로 나오는 컬럼)은 1
  - CASE문으로 원하는 텍스트로 변경 (Oracle에서는 DECODE)

  ```sql
    SELECT CASE GROUPING(ORDER_DT)
                WHEN 1 THEN 'TOTAL' ELSE ORDER_DT --NULL값이 TOTAL로 변경됨
            END AS ORDER_DT,
            COUNT(*)
        FROM STARBUCKS
    GROUP BY ROLLUP(ORDER_DT)
    ORDER BY ORDER_DT
  ```

---

### 5. 윈도우 함수

#### (1)

---

### 6. Top-N 쿼리

#### (1)

---

### 7. 셀프 조인

#### (1)

---

### 8. 계층 쿼리

#### (1)

---

### 9. PIVOT절, UNPIVOT절

#### (1)

---

### 10. 정규표현식

#### (1)

---

```

```
