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

- 데이터를 GROUP BY하여 나타낼 수 있는 데이터를 구하는 함수
- 그룹핑 기준 컬럼만 남기고, 그 외의 컬럼은 집계 결과만 남음 (여러 행을 하나로 줄이는 역할)
- 그룹에 들어가지 않는 일반 컬럼들은 SELECT절에 쓸 수 없음

#### (1) 집계 함수 (SUM, COUNT, AVG, MAX, MIN 등)

- GROUP BY+집계함수 : 그룹당 1행만 남음, 전체요약 및 보고용
- HAVING절로 필터링

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

- GROUP BY 없이도 각 행에 대해 집계, 순위, 누적값 등을 계산할 수 있는 함수
- OVER()절을 반드시 함께 사용
- PARTITION BY 컬럼
  - 데이터 집합을 그룹으로 나누는 기준 컬럼
  - GROUP BY처럼 그룹을 만들지만, 행을 줄이지 않고 유지한 채 계산 수행 (누적 계산 가능)
  - OVER 내에서만 사용 가능
  - 생략한다면 전체 기준으로 순위를 매김

```sql
함수명() OVER (
  PARTITION BY 그룹기준
  ORDER BY 정렬기준
)
```

#### (1) 순위 함수

순위를 매기는 함수

- RANK() : 같은 순위가 존재하면 그 만큼 다음 순위를 건너뜀 (1,2,2,4)
- DENSE_RANK() : 같은 순위가 존재하더라도 다음 순위를 건너뛰지 않고 이어서 매김 (1,2,2,3)
- ROW_NUMBER() : 동일한 값이라도 각기 다른 순위를 부여함

#### (2) 집계 함수

모든 행이 남기 때문에 누적합/비교/랭킹등 상세 분석용에 사용, WHERE또는 조건없이 사용 가능

- SUM(숫자)

  ```sql
  SELECT STUDENT_NAME,SUBJECT,SCORE,SUM(SCORE) OVER(PARTITION BY STUDENT_NAME) FROM SQLD;

  --누적합(Oracle)
  SUM(SCORE) OVER(PARTITION BY STUDENT_NAME ORDER BY SUBJECT DESC RANGE UNBOUNDED PRECEDING) -- 학생별 누적합이 나타남

  SUM(SCORE) OVER(ORDER BY SCORE DESC) -- 하나일 경우 동일한 누적합이 나옴
  ```

- MAX() / MIN()

  ```sql
  SELECT *
  FROM (
    SELECT *, MAX(점수) OVER(PARTITION BY 과목) AS 과목별최대점수 -- FROM절이 처음으로 실행되기 때문에 WHERE에서 함수 결과를 이용하기 위해 FROM절에 위치
    FROM 성적
  ) AS T
  WHERE 점수 = 과목별최대점수;
  ```

- AVG()
- COUNT()
- 윈도우 함수 사용 옵션(데이터 범위 지정 옵션)

  - UNBOUNDED PRECEDING : 위쪽 끝 행
  - UNBOUNDED FOLLOWING : 아래쪽 끝 행
  - CURRENT ROW : 현재 행
  - n PRECEDING : 현재 행에서 위로 n만큼 이동
  - n FOLLOWING : 현재 행에서 아래로 n만큼 이동

  ```sql
  -- ROW기반 : 행 자체가 기준
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW -- 처음부터 현재(=RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

  ROWS BETWEEN N PRECEDING AND N FOLLOWING -- 본인과 같거나 N차이나는 행

  ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING

  -- RANGE기반 : 행이 가지고 있는 데이터 값이 기준
  RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

  RANGE BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING

  RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
  ```

#### (3) 행 순서 함수

SQL SEVER(MSSQL)에서는 지원 X

- FIRST_VALUE() : 파티션별(PARTITION BY로 나눈 그룹) 가장 선두에 위치한 데이터
- LAST_VALUE() : 파티션별 가장 끝에 위치한 데이터
  - WINDOWING절의 default는 RANGE UNBOUNDED PRECEDING : 제일 마지막이 아닌 자기 값이 나옴
  - RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING 옵션을 추가해야 전체에서 마지막 값이 나옴
- LAG(컬럼,숫자(기본1)) / LEAD() : 파티션별로 특정 수 만큼 앞선(뒤에 있는) 데이터

#### (4) 비율 함수

SQL SEVER(MSSQL)에서는 지원 X

- RATIO_TO_REPORT(A) : 파티션별 합계에서 차지하는 비율, A/SUM(A)와 같음
- PERCENT_RANK(A) : 해당 파티션의 맨위 끝 행을 0, 맨아래를 1로 두고 현재 행이 위치하는 백분위 순위 값, (RANK()-1)/(COUNT(\*)-1)과 같음
- CUME_DIST(A) : 해당 파티션에서의 누적 백분율(0~1 사이),COUNT(ORDER BY A)/COUNT(\*) 와 같음
- NTILE(A) : 주어진 수만큼 행들을 N등분한 후 현재 행에 해당하는 등급을 구함, 행이 남으면 맨 앞 그룹부터 하나씩 더 채움

---

### 6. Top-N 쿼리

상위N개를 추출하기 위한 쿼리

#### (1) ROWNUM

- Oracle에서 슈도컬럼(Pseudo Column)임 = 실제로 존재하지 않는 가짜 컬럼
- 엑셀의 자동번호와 비슷한 목적 (SELECT ROWNUM), 임의의 번호 => 정렬 후 사용해야 TOP-N의 기능을 수행
- 건너뛰기(=5와 같은 WHERE절) 불가능

```sql
SELECT ROWNUM,이름,국어,영어
  FROM (
    SELECT 이름, 국어, 영어
      FROM EXAM_SCORE
      ORDER BY 국어 DESC, 영어 DESC -- ORDER BY가 밖에있으면 WHERE보다 늦게 실행되기 때문에 잘못된 결과가 나옴
  )
WHERE ROWNUM<=5;
```

#### (2) 윈도우 함수의 순위함수

- (ex) ROW_NUMBER()실행 후 WHERE RNUM<=5 로 사용 가능

---

### 7. 셀프 조인

- 나 자신과의 조인 ex) 한 테이블에 직원, 상사 정보가 다 있을 때 각 직원의 상사를 찾기 위한 연결
- 같은 테이블이 두번 이상 등장하기 때문에 ALIAS사용
- 계층구조, 비교, 관계 매핑에 자주 사용
- 카테고리 DEPTH가 깊어질수록 셀프조인이 반복됨 => 계층 쿼리로 간단하게 작성 가능

```sql
SELECT A.이름 AS 직원, B.이름 AS 상사
FROM EMP A
LEFT JOIN EMP B
  ON A.상사사번 = B.사번;
```

---

### 8. 계층 쿼리

테이블에 계층 구조를 이루는 컬럼이 존재할 경우 계층 쿼리를 이용해서 데이터를 출력

#### (1) Oracle 스타일

- CONNECT BY : 부모-자식 관계 조건 지정, 루트로부터 자식 노드를 생성해주는 절, 조건에 만족한 데이터가 없을 때까지 노드 생성
  - CONNECT BY PRIOR 부모컬럼 = 자식컬럼 : TOP->BOTTOM
  - CONNECT BY 자식컬럼 = PRIOR 부모컬럼 : BOTTOM->TOP
- START WITH : 최상위(루트)노드 지정, 경로가 시작되는 루트노드 생성
- LEVEL : 계층 깊이 자동 생성, 현재 DEPTH를 반환, 루트는 1
- SYS_CONNECT_BY_PATH(컬럼,'구분자') : 루트부터 현재 노드까지의 경로를 문자열로 반환
- PRIOR : 바로 앞에 있는 부모 노드의 값을 반환, 부모-자식 관계의 방향을 지정

```sql
SELECT
  LEVEL AS 계층, -- 현재 행이 몇단곈지 자동 계산
  ENAME AS 직원명,
  MGR AS 상사번호,
  SYS_CONNECT_BY_PATH(ENAME, ' > ') AS 경로
FROM EMP
START WITH MGR IS NULL -- 루트(사장)부터 시작
CONNECT BY PRIOR EMPNO = MGR; -- 상사의 EMPNO=자식의 MGR, 부모->자식 방향으로
```

- CONNECT_BY_ROOT 컬럼 : 어떤 루트에서 시작됐는지 보여줌
- CONNECT_BY_ISLEAF 컬럼 : 현재 행이 마지막행(리프노트)이면 1 아니면 0
- ORDER을 할 경우 계층과 상관없이 정렬될 수 있으니 ORDER SIBLING BY절 사용

---

### 9. PIVOT절, UNPIVOT절 (Oracle전용 구문)

#### (1) PIVOT절

- 행이 열로 변환(가로로 긴 형태로 변환됨)하면서 데이터를 집계
- FOR : PIVOT할 열, 값이 헤더로 올라오게 될 컬럼을 지정
- IN : 실제로 PIVOT할 열 값, 헤더에 표시할 컬럼 값
- IN절에 붙은 ALIAS\_집계함수의 ALIAS (ex) IT_SAL

```sql
SELECT *
FROM (
  SELECT 이름, 과목, 점수
  FROM 성적
)
PIVOT (
  MAX(점수)
  FOR 과목 IN ('수학' AS 수학, '영어' AS 영어)
);

--CASE WHEN (Oracle이 아닐 경우)
SELECT
  이름,
  MAX(CASE WHEN 과목 = '수학' THEN 점수 END) AS 수학,
  MAX(CASE WHEN 과목 = '영어' THEN 점수 END) AS 영어
FROM 성적
GROUP BY 이름;
```

#### (2) UNPIVOT절

- PIVOT과 반대 작업
- 집계된 데이터를 열에서 행으로 변환

```sql
SELECT *
FROM (
  SELECT 이름, 수학, 영어
  FROM 성적_피벗뷰  -- 또는 위 PIVOT 결과 쿼리
)
UNPIVOT (
  점수 FOR 과목 IN (수학, 영어)
);
```

- PIVOT/UNPIVOT 모두 INCLUDE NULL옵션 사용 가능, 기본값은 NULL제외

---

### 10. 정규표현식

특정 규칙에 맞는 문자열 패턴을 정의하는 식

#### (1) REGEXP_SUBSTR(문자열,패턴)

- 문자열에서 특정 패턴에 맞는 부분 추출, 첫번째 일치하는 문자 반환
- .(한문자), |(OR), \(뒤에오는 문자가 일반문자로 취급),
- ^A : A로 시작
- A$ : A로 끝
- AB? : A, AB와 일치, 선행문자(B)가 0개 또는 1개
- A\* : 선행문자가 0개 이상
- A+ : 선행문자가 1개 이상
- [] : 대괄호 문자중 하나와 일치
- [-] : [0-9]는 숫자 0-9까지, 연속 문자의 범위 지정,
- [^sq]l : sl/ql이 아닌 l로 끝난 두글자 문자열, 대괄호 안의 문자들을 제외한 나머지 문자중 하나와 일치
- () : 소괄호로 묶인 표현식을 한 단위로 취급, (AB)+는 AB가 반복됨

```sql
REGEXT_SUBSTR("1234567890",'(123)(4(56)(78))',1,1,'i',1) -- 123출력

REGEXP_SUBSTR(
  문자열,           -- 대상 문자열
  정규표현식,       -- 패턴
  시작 위치,        -- 몇 번째 문자부터 검사할지
  일치 횟수,        -- 몇 번째 일치 항목을 찾을지
  플래그,           -- 'i' = 대소문자 구분 없음 등
  그룹 번호          -- (괄호) 몇 번째 그룹을 추출할지
)
```

- (123)(4(56)(78)) : 그룹을 나타냄 (123) / (4(56)(78)) / (56) / (78)
- POSIX문자 클래스를 사용하여 문자그룹을 간단히 표현

  | 클래스 이름  | 의미                         | 설명 예시                  |
  | ------------ | ---------------------------- | -------------------------- |
  | `[:alnum:]`  | 알파벳 + 숫자 문자           | `[A-Za-z0-9]`과 같음       |
  | `[:alpha:]`  | 알파벳 문자                  | `[A-Za-z]`                 |
  | `[:digit:]`  | 숫자                         | `[0-9]`                    |
  | `[:lower:]`  | 소문자                       | `[a-z]`                    |
  | `[:upper:]`  | 대문자                       | `[A-Z]`                    |
  | `[:blank:]`  | 공백 문자(스페이스, 탭)      | `' '` 또는 `\t`            |
  | `[:space:]`  | 모든 공백 문자               | 스페이스, 탭, 개행 등      |
  | `[:xdigit:]` | 16진수 문자                  | `[0-9A-Fa-f]`              |
  | `[:punct:]`  | 문장 부호 (.,!? 등)          | 기호 문자 전반             |
  | `[:cntrl:]`  | 제어 문자 (Ctrl + ...)       | 탭, 줄바꿈, 백스페이스 등  |
  | `[:print:]`  | 출력 가능한 모든 문자        | 공백 포함, 제어문자는 제외 |
  | `[:graph:]`  | 출력 가능한 문자 (공백 제외) | 알파벳, 숫자, 기호 등      |

---

- 이메일 패턴
  - [[:alnum:].\_%+-] : 알파벳 대소문자, 숫자, 특수문자(. % + -)가 포함된 하나 이상의 문자열
  - @[[:alnum:]-] : 이메일 도메인 부분, 알파벳 대소문자,숫자 또는 하이픈이 포함된 하나 이상의 문자열
  - \.[[:alpha:].]{2,} : 최상위 도메인, 마침표뒤에 2개이상의 알파벳 또는 마침표가 나오는 패턴

```sql
SELECT REGEXP_SUBSTR(
    'EMAIL: myemail@example.com',
    '[[:alnum:]._%+-]+@[[:alnum:]-]+\.[[:alpha:].]{2,}'
) AS EMAIL
FROM DUAL
```

- ## 전화번호 패턴
  - 010으로 시작
  - [-]? : 하이픈은 0개 또는 1개
  - [0-9]{3,4} : 숫자 3자리 또는 4자리
  - [0-9]{4} : 숫자 4자리

```sql
 '010[-]?[0-9]{3,4}[-]?[0-9]{4}'
```

#### (2) REGEXP_REPLACE(문자열,패턴,대체문자)

문자열내에서 패턴과 일치하는 부분을 찾아 다른 문자열로 대체

#### (3) REGEXP_INSTR(문자열,패턴[,시작위치,몇번째 문자열,0(시작점)/1(끝나는 다음 위치),c(대소문자구분)/i(대소문자 구분X)])

문자열에서 정규표현식 패턴과 일치하는 부분의 위치 반환

#### (4) REGEXP_COUNT(문자열,패턴[,시작위치,몇번째 문자열,0(시작점)/1(끝나는 다음 위치),c(대소문자구분)/i(대소문자 구분X)])

문자열에서 일치하는 부분이 몇번 나타나는지 계산

#### (5) REGEXP_LIKE(문자열,패턴)

문자열 패턴과 일치하는지 여부를 확인하는 조건식으로 데이터 필터링에 사용

```sql
WHERE REGEXP_LIKE(FRIST_NAME,'^Ste(v|p)en$') -- Steven or Stephen
```
