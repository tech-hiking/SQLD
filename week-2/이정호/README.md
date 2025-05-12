# PART 2 | SQL 기본 및 활용

## CHAPTER 2 | SQL 활용 (171~305p)

### 1) 서브쿼리
- SELECT 절: 스칼라 서브쿼리
- FROM 절: 인라인 뷰
- WHERE/HAVING 절: 중첩 서브쿼리
- 스칼라 서브쿼리
  - 주로 SELECT 절에 위치하나 컬럼이 올 수 있는 대부분 위치에 사용할수 있다. 컬럼 대신 사용되므로 반드시 하나의 값만을 반환해야 하며 그렇지 않은 경우 에러를 발생.
- 인라인 뷰
  - FROM 절 등 테이블명이 올 수 있는 위치에 사용 가능하다.
- 중첩 서브쿼리
  - WHERE/HAVING 절에 사용할 수 있다.
  - 비연관 서브쿼리: 메인쿼리와 관계를 맺고 있지 않음
    - 서브쿼리 내에 메인쿼리의 컬럼이 존재하지 않는다.
  - 연관 서브쿼리: 메인쿼리와 관계를 맺고 있음
    - 서브쿼리 내에 메인쿼리의 컬럼이 존재
  - 반환하는 데이터 형태
    - 단일 행 서브쿼리: 1건 이하의 데이터를 반환(=, <, >, <=. >=, <> 연산자와 함께 사용)
    - 다중 행 서브쿼리: 여러건 데이터를 반환(IN, ALL, ANY, SOME, EXISTS 연산자와 함께 사용)
    - 다중 컬럼 서브쿼리: 여러 컬럼의 데이터를 반환


### 2) 뷰
- 특정 SELECT 문에 이름을 붙여서 재사용 가능하도록 저장해놓은 오브젝트이다. 테이블처럼 사용할수 있으며 실제 데이터를 저장하지는 않고 해당 데이터를 조회하는 SELECT 문만 가지고 있다.
- 보안성의 특징을 갖는다, 보안이 필요한 컬럼을 가진 테이블일 경우 해당 컬럼을 제외한 별도의 뷰 생성할 수 있다.


### 3) 집합 연산자
- 각 쿼리의 결과 집합을 가지고 연산을 하는 명령어
- UNION ALL: 합집합. 중복된 행도 그대로 출력
- UNION: 합집합. 중복된 행도 한 줄로 출력
  - 결과값에 중복된 행이 없을 때는 내부적으로 중복된 행 제거 과정을 거치므로 UNION ALL보다 성능상 불리할 수 있다.
- INTERSECT: 교집합. 중복된 행도 한 줄로 출력
  - 교집합의 헤더값은 첫 번째 쿼리를 따라간다.(188p)
- MINUS/EXCEPT: 차집합. 중복된 행도 한 줄로 출력


### 4) 그룹 함수
- 데이터를 GROUP BY하여 나타낼 수 있는 데이터를 구하는 함수이다.
- 집계 함수: COUNT, SUM, AVG, MAX, MIN 등
- 소계(총계) 함수: ROLLUP, CUBE, GROUPING SETS 등
- ROLLUP: 소그룹 간의 소계 및 총계를 계산하는 함수이다.
  - ROLLUP(A, B, C)
    1. A, B, C로 그룹핑
    2. A, B로 그룹핑
    3. A로 그룹핑
    4. 총합계
    ** 내부 인수를 소괄호로 묶었을 때도 동일하게 A, B, C 처럼 취급(194p)
- CUBE: 소그룹 간의 소계 및 총계를 **다차원적으로** 계산할 수 있는 함수이다. 조합 할 수 있는 모든 그룹에 대한 소계를 집계한다.
  - CUBE(A, B, C)
    1. A, B, C로 그룹핑
    2. A, B로 그룹핑
    3. A, C로 그룹핑
    4. B, C로 그룹핑
    5. A로 그룹핑
    6. B로 그룹핑
    7. C로 그룹핑
    8. 총합계
  ** 인수의 순서가 바뀌어도 같은 결과를 출력
- GROUPING SETS: 특정 항목에 대한 소계를 계산하는 함수이다. 인자값으로 ROLLUP/CUBE를 사용할 수도 있다.
  - GROUPING SETS(A, B)
    1. A로 그룹핑
    2. B로 그룹핑
  - GROUPING SETS(A, B, ())
    1. A로 그룹핑
    2. B로 그룹핑
    3. 총합계
  - GROUPING SETS(A, ROLLUP(B))
    1. A로 그룹핑
    2. B로 그룹핑
    3. 총합계
  - GROUPING SETS(A, ROLLUP(B, C))
    1. A로 그룹핑
    2. B, C로 그룹핑
    3. B로 그룹핑
    4. 총합계
  - GROUPING SETS(A, B, ROLLUP(C))
    1. A로 그룹핑
    2. B로 그룹핑
    3. C로 그룹핑
    4. 총합계
  ** 각 인수별로 결과를 나열할뿐 조합은 하지 않음, 인수의 순서가 바뀌어도 같은 결과를 출력
- GROUPING: ROLLUP/CUBE/GROUPING SETS 등과 함께 쓰이며 소계를 나타내는 Row를 구분할 수 있게 해준다. 원하는 뒤치에 원하는 텍스트를 출력할 수 있다.
  - 소계가 계산된 Row에서는 GROUPING 함수의 결과값이 1이 되고 나머지 Row에서는 0이 된다. 이를 이용하여 CASE등으로 원하는 텍스트 출력 가능.


### 5) 윈도우 함수
- OVER 키워드와 함께 사용되며 역할에 따라 다음과 같이 나눌 수 있다.
  - 순위 함수: RANK, DENSE_RANK, ROW_NUMBER
  - 집계 함수: SUM, MAX, MIN, AVG, COUNT
  - 행 순서 함수: FIRST_VALUE, LAST_VALUE, LAG, LEAD
  - 비율 함수: CUME_DIST, PERCENT_RANK, NTILE, RATIO_TO_REPORT
- 순위 함수
  - RANK: 순위를 매기면서 같은 순위가 존재하면 다음 순위를 건너 뛴다.
    - 1, 2, 2, 4
  - DENSE_RANK: 순위를 매기면서 같은 순위가 존재하더라도 다음 순위를 건너뛰지 않고 이어서 매긴다. (dense: 밀집한)
    - 1, 2, 2, 3
  - ROW_NUMBER: 순위를 매기면서 동일한 값이라도 각기 다른 순위를 부여한다.
    - 1, 2, 3, 4
- 집계 함수
  - SUM: 데이터의 합계를 구하는 함수이다. 인자값으로는 숫자형만 올 수 있다.
    - 누적 값 구하기
      - Oracle의 경우 OVER 절 내에 ORDER BY 사용해서 가능하다.
        - SUM 하는 컬럼을 OVER 절 ORDER BY 절에서 명시해주게 되면 RANGE UNBOUNDED PRECEDING 구문 없이도 누적합이 집계된다.
          - SUM(SCORE) OVER(ORDER BY SCORE DESC) AS SUM_SCORE
      - 233p하단 예시문제: 동일 값이 기준일시 누적 값은 합산으로???
  - MAX, MIN, AVG
  - WINDOWING 절을 이용하여 집계하려는 데이터의 범위를 지정할 수 있다.(241p 하단)
    - UNBOUNDED PRECEDING: 위쪽 끝 행
    - UNBOUNDED FOLLOWING: 아래쪽 끝 행
    - CURRENT ROW: 현재 행
    - n PRECENDING: 현재 행에서 위로 n만큼 이동
    - n FOLLOWING: 현재 행에서 아래로 n만큼 이동
    - ROWS: 행 자체가 기준이 된다
    - RANGE: 행이 가지고 있는 데이터 값이 기준이 된다.
      - RANGE BETWEEN 10 PRECEDING AND CURRENT ROW: 현재 행이 가지고 있는 값보다 10만큼 적은 행부터 현재 행까지, RANGE 10 PRECEDING과 같은 의미
  - COUNT
    - 과목별로 본인보다 점수가 높거나 같은 건수를 카운트 하는 쿼리의 OVER 예시
      - OVER(PARTITION BY SUBJECT ORDER BY SOCRE DESC RANGE UNBOUNDED PRECEDING)
    - 과목별로 본인 점수와 5점 이하로 차이가 나거나 점수가 같은 건수를 카운트 하는 쿼리의 OVER 예시
      - OVER(PARTITION BY SUBJECT ORDER BY SCORE DESC RANGE BETWEEN 5 PRECEDING AND 5 FOLLOWING)
- 행 순서 함수(SQL Server(MYSQL)에서는 지원하지 않는다.)
  - FIRST_VALUE: 파티션별 가장 선두에 위치한 데이터를 구하는 함수. 
  - LAST_VALUE: 파티션별 가장 끝에 위치한 데이터를 구하는 함수.
    - WINDOWING 절의 default가 RANGE UNBOUNDED PRECEDING이어서 파티션의 범위가 맨 위 끝 행부터 현재행까지로 지정되므로 원하는 결과 출력을 위해서는 다음과 같이 명시해주어야 한다.(MIN 등도 마찬가지)
      - OVER(ORDER BY SCORE RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
  - LAG: 파티션별로 특정 수만큼 앞선 데이터를 구하는 함수.
    - 자기 row는 포함하지 않음
    - 두번 째 인자값을 생략하면 default는 1이다.
    - LAG(SCORE, 2) OVER ~: 과목 별로 본인보다 2만큼 앞에 있는 점수를 구한다.
  - LEAD: 파티션별로 특정 수만큼 뒤에 있는 데이터를 구하는 함수.
    - 자기 row는 포함하지 않음
    - 두번 째 인자값을 생략하면 default는 1이다.
- 비율 함수
  - RATIO_TO_REPORT: 파티션별 합계에서 차지하는 비율을 구하는 함수(SQL Server(MYSQL)에서는 지원하지 않는다.)
    - SCORE/SUM 값과 동일하다.
    - 과목별 SCORE 합계에서 차지하는 비율 구하는 쿼리의 부분 예시: RATIO_TO_REPORT(SCORE) OVER(PARTITION BY SUBJECT) AS RATIO_TO_REPORT
  - PERCENT_RANK: 해당 파티션의 맨 위 끝 행을 0, 맨 아래 끝 행을 1로 놓고 현재 행이 위치하는 백분휘 순위 값을 구하는 함수이다.(SQL Server(MYSQL)에서는 지원하지 않는다.)
    - (RANK-1)/(COUNT-1) 값과 동일하다.
    - 과목별로 나눈 파티션에서 해당 SCORE가 차지하는 백분위 순위를 구하는 쿼리의 부분 예시: PERCENT_RANK() OVER(PARTITION BY SUBJECT ORDER BY SCORE)
  - CUME_DIST: 해당 파티션에서의 누적 백분율을 구하는 함수이다. 결과값을 0보다 크고 1보다 작거나 같은 값을 가진다.(SQL Server(MYSQL)에서는 지원하지 않는다.)
    - COUNT/TOTAL_COUNT 값과 동일하다.
  - NTILE: 주어진 수만큼 행들을 n등분한 후 현재 행에 해당하는 등급을 구하는 함수이다.
    - 할당 행이 남았을 경우 맨 앞의 그룹부터 하나씩 더 채워진다.(예시 267p)


### 6) Top-N 쿼리
- N위까지 추출한다.
- ROWNNUM
  - 슈도 컬럼으로 엑셀에서 자동번호를 매기는 것과 유사
  - ROWNNUM은 항상 < 조건이나 <= 조건으로 사용해야 한다.
  - ORDER BY 절이 WHERE 절보다 나중에 수행되기 때문에 유의한다.(271p)
- 윈도우 함수의 순위 함수
  - RANK 등 활용한다.


### 7) 셀프 조인
- 나 자신과의 조인으로 FROM 절에 같은 테이블이 두 번 이상 등장하기 때문에 ALIAS를 반드시 표기해주어야 한다.


### 8) 계층 쿼리
- 테이블에 계층 구조를 이루는 컬럼이 존재할 경우 계층 쿼리를 이용해서 데이터를 출력할 수 있다. 셀프조인으로 작성한 쿼리를 계층 쿼리로 간단히 작성 가능하다.
- LEVEL: 현재의 DEPTH를 반환한다. 루트 노드는 1이 된다.
- SYS_CONNECT_BY_PATH(컬럼, 구분자): 루트 노트부터 현재 노드까지의 경로를 출력해주는 함수이다.
- START WITH: 경로가 시작되는 루트 노드를 생성해주는 절이다.
- CONNECT BY: 루트로부터 자식 노드를 생성해주는 절이다. 조건에 만족하는 데이터가 없을 때까지 노드를 생성한다.
- PRIOR: 바로 앞에 있는 부모 노드의 값을 반환한다.
- CONNECT_BY_ROOT 컬럼: 루트 노드의 주어진 컬럼 값을 반환한다.
- CONNECT_BY_ISLEAF: 가장 하위 노드인 경우 1을 반환하고 그 외에는 0을 반환한다.
- 순방향 쿼리 예시
  - START WITH PARENT_CATEGORY IS NULL
  - CONNECT BY PRIOR CATEGORY_NAME = PARENT_CATEGORY;
- 역방향 쿼리 예시
  - START WITH CATEGORY_TYPE = '소'
  - CONNECT BY CATEGORY_NAME = PRIOR PARENT_CATEGORY;
- 정렬은 ORDER SIBLINGS BY 절을 이용하여 같은 레벨끼리 정렬되도록 한다.


### 9) PIVOT 절과 UNPIVOT 절
- PIVOT 절
  - 회전하다라는 의미로 행이 열로 변환된다.(대부분 가로로 길쭉한 형태로)
  - PIVOT 절의 구성 요소
    - 집계 함수: 결과 데이터에 표시할 집계 데이터를 정의
    - FOR 절: PIVOT 할 컬럼을 지정
    - IN 절: PIVOT 할 컬럼 값을 지정
  - PIVOT 구문에 Alias를 지정 가능
    - PIVOT (SUM(SALARY) AS SAL FOR DEPT_NAME IN ('인사팀' AS HR, ...))
    - 집계 함수에 붙은 Alias는 헤더명 끝에 언더바와 함꼐 표시가 되고 IN 절에 붙은 Alias는 헤더명 앞쪽에 표시가 된다.
      - ex) HR_SAL
  - 집계할 데이터가 없는 경우 값은 NULL로 출력된다.
  - 290p 예시
- UNPIVOT 절
  - 열 데이터를 다시 행으로 변환시키는 역할을 한다.
  - SELCT절의 피벗된 테이블에서 확인할 수 없는 컬럼은 실제 피벗된 테이블에 존재하는 것이 아니라 UNPIVOT 절에서 만들어 낸 것이다.
  - UNPIVOT 절의 구성 요소
    - UNPIVOT 컬럼: UNPIVOT 된 값이 들어갈 컬럼을 지정
    - FOR 절: UNPIVOT 된 값에 대한 설명이 들어갈 컬럼을 지정
    - IN 절: FOR 절에서 생성한 컬럼에 표시될 데이터 값을 지정
  - INCLUDE NULLS 옵션으로 피벗 테이블의 NULL 데이터도 함께 출력할 수 있다.
- PIVOT/UNPIVOT 비교 TIP(292p)
  - PIVOT: 인라인 뷰, 집계 함수, 데이터를 나열한 IN 절
  - UNPIVOT: 결과 데이터에 출력될 헤더명 네이밍, IN 절에는 기존 컬럼명을 나열


### 10) 정규표현식
- 특정 규칙에 맞는 문자열 패턴을 정의하는 식이다.
- 정규 표현식의 기본연산자 - 296p
- 정규 표현식의 괄호 연산자 - 297p
- 정규 표현식의 POSIX 문자 클래스 - 298p
- REGEXP_SUBSTR 함수: 문자열에서 특정 패턴에 맞는 부분을 추출하는 함수이다.
  - 해당하는 패턴이 없으면 NULL이 출력된다.
  - ('SQL', 'S|L'): 첫 번쨰로 일치하는 문자인 S가 출력된다.
  - 원화표시를 | 앞에 쓰면 자체를 기호로 인식한다.
  - 총 6개 매개변수 까지 사용 가능할 때 뒤쪽에 있는 네 개의 매개 변수는 차례대로
    - 검색 시작 위치 / 첫 번째로 일치하는 패턴 / 대소문자 구분 없이 검색 / n번쨰 그룹과 일치
- REGEXP_REPLACE 함수: 문자열 내에서 정규표현식 패턴과 일치하는 부분을 찾아 이를 지정한 다른 문자열로 대체하는 ㅎ마수이다.
- REGEXP_INSTR 함수: 문자열에서 정규표현식 패턴과 일치하는 부분의 위치를 반환하는 함수이다.
- REGEXP_COUNT 함수: 문자열내에서 정규표현식 패턴과 일치하는 부분이 몇 번 나타나는지를 계산하는 함수이다.
- REGEXP_LIKE 조건: 정규표현식을 사용하여 문자열 패턴과 일치하는지 여부를 확인하는 조건식이다.
  - LIKE와 비슷한 기능을 하지만 보다 강력하고 복잡한 패턴 매칭을 지원한다고 이해하면 됨

