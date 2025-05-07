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
