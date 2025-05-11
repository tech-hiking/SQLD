# 정리 노트

## SQL 의미 (79P)

- Structured Query Language
- **"구조"**를 떠올리자
- 즉 질의를 표현하는 언어인데, **표준으로 만든 구조**를 따라야 한다.

## 산술 연산자 (81P)

- `NUMBER` / `DATE` 자료형에 사용할 수 있다.
- 다른 컬럼끼리 연산에 `NULL`이 포함되어 있으면 결과값은 `NULL`이다.

### Q. 어떻게 `DATE` 자료형을 지니길래 산술 연산에 이용할 수 있을까?

산술연산이 가능하다는 건 내부적으로 숫자로 변환하기 때문이다.

#### 1. Oracle

Oracle은 `DATE` 타입은 7바이트이며, C 구조체 패딩으로 8바이트로 실제로 저장된다.

- Byte 1: Epoch Number
- Byte 2: Year offset from epoch base
- Byte 3: Month
- Byte 4: Day
- Byte 5: Hour (24 hour time)
- Byte 6: Minute
- Byte 7: Second

```sql
SQL> select sysdate, dump(sysdate) from dual;

SYSDATE              DUMP(SYSDATE)
-------------------- -------------------------------------
21-jul-2011 16:58:35 Typ=13 Len=8: 7,219,7,21,16,58,35,0
```

- epoch base = 256 \* 7 = 1792
- epoch base + offset = 1792 + 219 = 2011

`DATE` 타입의 산술 연산은 내부적으로 DAY 증감으로 처리한다.

```sql
select SYSDATE-1 as 하루전 FROM DUAL;
select SYSDATE+1 as 내일 FROM DUAL;
select to_char(sysdate-1,'yyyy-mm-dd') as 하루전 from dual;
select to_char(sysdate,'yyyy-mm-dd') as 오늘 from dual;
select to_char(sysdate+1,'yyyy-mm-dd') as 내일 from dual;

select to_char(sysdate-1,'yyyy-mm-dd') as 하루전
, to_char(sysdate,'yyyy-mm-dd') as 오늘
, to_char(sysdate+1,'yyyy-mm-dd') as 내일 from dual;
```

#### 2. MySQL

MySQL 구조체는 다음과 같다. ([mysql_time.h#L82](https://github.com/mysql/mysql-server/blob/ff05628a530696bc6851ba6540ac250c7a059aa7/include/mysql_time.h#L82))

```c
/*
  Structure which is used to represent datetime values inside MySQL.

  We assume that values in this structure are normalized, i.e. year <= 9999,
  month <= 12, day <= 31, hour <= 23, hour <= 59, hour <= 59. Many functions
  in server such as my_system_gmt_sec() or make_time() family of functions
  rely on this (actually now usage of make_*() family relies on a bit weaker
  restriction). Also functions that produce MYSQL_TIME as result ensure this.
  There is one exception to this rule though if this structure holds time
  value (time_type == MYSQL_TIMESTAMP_TIME) days and hour member can hold
  bigger values.
*/
typedef struct MYSQL_TIME {
  unsigned int year, month, day, hour, minute, second;
  unsigned long second_part; /**< microseconds */
  bool neg;
  enum enum_mysql_timestamp_type time_type;
  /// The time zone displacement, specified in seconds.
  int time_zone_displacement;
} MYSQL_TIME;
```

#### 3. PostgreSQL

PostgreSQL의 `Timestamp`는 단순 **8바이트 정수형**으로 표현한다.

```c
/*
 * Timestamp represents absolute time.
 *
 * Interval represents delta time. Keep track of months (and years), days,
 * and hours/minutes/seconds separately since the elapsed time spanned is
 * unknown until instantiated relative to an absolute time.
 *
 * Note that Postgres uses "time interval" to mean a bounded interval,
 * consisting of a beginning and ending time, not a time span - thomas 97/03/20
 *
 * Timestamps, as well as the h/m/s fields of intervals, are stored as
 * int64 values with units of microseconds.  (Once upon a time they were
 * double values with units of seconds.)
 *
 * TimeOffset and fsec_t are convenience typedefs for temporary variables.
 * Do not use fsec_t in values stored on-disk.
 * Also, fsec_t is only meant for *fractional* seconds; beware of overflow
 * if the value you need to store could be many seconds.
 */

typedef int64 Timestamp;
typedef int64 TimestampTz;
typedef int64 TimeOffset;
typedef int32 fsec_t;			/* fractional seconds (in microseconds) */
```

## 합성 연산자 (83P)

문자와 문자를 연결할 때 `||` 으로 사용한다.  
`CONCAT()` 함수만 알고 있었다.

```sql
SELECT 'S' || 'Q' || 'L' || '개' || '발' || '자' AS SQLD
  FROM DUAL;

-- SQL개발자
```

## 문자 함수 > `LTRIM` / `RTRIM` (84P)

교재 설명은 아래와 같다.
RDBMS 벤더별로 동작이 다르다.

- `LTRIM(문자열 [, 특정 문자])` - []는 옵션
- `RTRIM(문자열 [, 특정 문자])` - []는 옵션

### 1. Oracle / MSSQL

Oracle / MSSQL은 특정 문자를 집합으로 바라본다는 점을 기억할 것! ([오라클 공식 문서](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/RTRIM.html))

![alt text](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/img/rtrim.gif)

```
RTRIM removes from the right end of char all of the characters that appear in set. This function is useful for formatting the output of a query.
```

- `문자열`에 왼쪽(`LTRIM`) 또는 오른쪽(`RTRIM`)부터 한 글자씩 `특정 문자 집합`에 속하면 삭제
- 반복하다 처음으로 `특정 문자 집합`에 속하지 않을 시 함수 동작을 끝낸다.

---

### 2. MySQL (Server)

- MySQL은 두 번째 인자가 없으며, `SELECT TRIM(TRAILING 'LE' FROM 'SQL');` 은 `SQL` 이다.
- Oracle 처럼 특정 문자를 집합처럼 취급하지 않고 시퀀스로 비교한다.
- 아래는 그 [소스 코드](https://github.com/mysql/mysql-server/blob/ff05628a530696bc6851ba6540ac250c7a059aa7/sql/item_strfunc.cc#L1803)다.

```c
// SQL 파서가 `Item_func_trim` C++ 클래스 인스턴스를 생성


String *Item_func_trim::val_str(String *str) {
    ...

    String *remove_str = &remove;  // Default value.

    ...

    const size_t remove_length = remove_str->length();
    if (remove_length == 0 || remove_length > res->length()) return res;

    const char *ptr = res->ptr();
    const char *end = ptr + res->length();
    const char *const r_ptr = remove_str->ptr();

    if (use_mb(res->charset())) {
        // 멀티바이트 처리 로직
        if (m_trim_leading) {
            while (ptr + remove_length <= end) {
                uint num_bytes = 0;
                while (num_bytes < remove_length) {
                    uint len;
                    if ((len = my_ismbchar(res->charset(), ptr + num_bytes, end)))
                        num_bytes += len;
                    else
                        ++num_bytes;
                }
                if (num_bytes != remove_length) break;
                if (memcmp(ptr, r_ptr, remove_length)) break;
                ptr += remove_length;
            }
        }
        if (m_trim_trailing) {
            ...
        }
        ...
    } else {
        // 싱글바이트 처리 로직 (ex. ASCII)
        if (m_trim_leading) {
            while (ptr + remove_length <= end && !memcmp(ptr, r_ptr, remove_length))
                ptr += remove_length;
        }
        if (m_trim_trailing) {
            while (ptr + remove_length <= end &&
                    !memcmp(end - remove_length, r_ptr, remove_length))
                end -= remove_length;
        }
    }

    if (ptr == res->ptr() && end == ptr + res->length()) return res;
    tmp_value.set(*res, static_cast<uint>(ptr - res->ptr()),
                    static_cast<uint>(end - ptr));
    return &tmp_value;
}
```

- MySQL Server 에서는 `memcmp` 비교로 문자열 시퀀스 전체를 비교한다.

### 3. PostgreSQL

- PostgreSQL은 Oracle 과 마찬가지로 두 번째 문자열을 **집합으로 처리**한다.
- [PostgreSQL 코드](https://github.com/postgres/postgres/blob/caa76b91a60681dff0bf193b64d4dcdc1014e036/src/backend/utils/adt/oracle_compat.c#L394)는 아래와 같다.

```cpp
/*
 * Common implementation for btrim, ltrim, rtrim
 */
static text *
dotrim(const char *string, int stringlen,
	   const char *set, int setlen,
	   bool doltrim, bool dortrim)
{
	int			i;

	/* Nothing to do if either string or set is empty */
	if (stringlen > 0 && setlen > 0)
	{
		if (pg_database_encoding_max_length() > 1)
		{
			/*
			 * In the multibyte-encoding case, build arrays of pointers to
			 * character starts, so that we can avoid inefficient checks in
			 * the inner loops.
			 */
             ...

             // 각 바이트마다 싱글 바이트 / 멀티 바이트 인코딩 분기 처리
        }
        else {
            /*
			 * In the single-byte-encoding case, we don't need such overhead.
			 */
			if (doltrim)
			{
				while (stringlen > 0)
				{
					char		str_ch = *string;

					for (i = 0; i < setlen; i++)
					{
						if (str_ch == set[i])
							break;
					}
					if (i >= setlen)
						break;	/* no match here */
					string++;
					stringlen--;
				}
			}

			if (dortrim)
			{
				while (stringlen > 0)
				{
					char		str_ch = string[stringlen - 1];

					for (i = 0; i < setlen; i++)
					{
						if (str_ch == set[i])
							break;
					}
					if (i >= setlen)
						break;	/* no match here */
					stringlen--;
				}
			}
        }
    }

    /* Return selected portion of string */
	return cstring_to_text_with_len(string, stringlen);
```

- `TRIM([위치] [특정 문자] [FROM] 문자열)` - []는 옵션

  - `TRIM(LEADING '블' FROM '블랙핑크')` -> `랙핑크`

## 문자 함수 > `SUBSTR(문자열, 시작점 [, 길이])` (87P)

- 시작점은 인덱스(0부터 시작)가 아니다.
- 인간 친화적인 위치(1번째, 2번째, ...)로 기억하자.

## 문자 함수 > `SIGN(수)`

- 음수 : -1
- 0 : 0
- 양수 : 1

## 숫자 함수 > `MOD(수1, 수2)` (95P)

- MOD(15, 17) -> 몫 2, 나머지 1
- MOD(15, -4) -> 몫 -3, 나머지 3
- MOD(-15, 0) -> 나머지 -15 (수2가 0이기에 수1 그대로 반환)
  - `Oracle` 기준
  - `MySQL` / `MSSQL` 기준 `NULL` 반환
  - `PostgreSQL` 기준 오류 반환 (`ERROR: division by zero`)
- MOD(-15, -4) -> 몫 3, 나머지 -3

- 몫의 부호는 분수 처리한다고 취급하며, 한 쪽이 마이너스면 몫도 마이너스다. 그 외는 플러스다.
- 나머지는 역산하여 구한다고 생각하면 편하다.
- -15, -4 -> -4 \* 3 + (나머지) = -15 이기에, 나머지는 -3 이다.
- 15, -4 -> -4 \* -3 + (나머지) = 15 이기에, 나머지는 +3 이다.

## 날짜 함수 > `ADD_MONTHS` (96P)

- 이전 달이나 다음 달에 기준 날짜의 일자가 없을 시 해당 월의 마지막 일자가 반환

- ADD_MONTHS(TO_DATE('2021-12-31', 'YYYY-MM-DD'), -1) -> 2021-11-30
- ADD_MONTHS(TO_DATE('2021-12-31', 'YYYY-MM-DD'), 1) -> 2022-01-31

## NULL 관련 함수 (99P)

1. `NVL(인수1, 인수2)` : 인수1 NULL -> 인수2 or 인수1
2. `NULLIF(인수1, 인수2)` : 인수1 = 인수2 -> NULL or 인수1

- 161P

3. `COALESCE(인수1, 인수2, 인수3, ...)` : `NULL` 이 아닌 최초의 인수
4. `NVL2(인수1, 인수2, 인수3)` : 인수1 NULL (X) -> 인수2, 인수 1 NULL (O) -> 인수3

## CASE (102P)

```
CASE WHEN SUBWAY_LINE = '1' THEN 'BLUE'
    ...
    [ELSE 'GRAY']
END
```

- 별도 `ELSE` 값이 없을 시 `NULL`이 된다. (문제 출제)

- Oracle의 `DECODE` 함수가 있다.
- `DECODE` 함수 인자가 짝수가 아니라, 마지막에 하나가 더 있을 시 `ELSE` 값이다.

## SQL 연산자 (108P)

- LIKE '비교 문자열' : `where col like '%#%%' escape '#'` `%` 문자를 넣어야할 때, escape 지정을 해야 한다.

## SELECT 문 논리적 수행 순서 (117P)

- SELECT - 5
- FROM - 1
- WHERE - 2
- GROUP BY - 3
- HAVING - 4
- ORDER BY - 6

SELECT 절의 집계 함수 Alias를 GROUP BY, HAVING에서 사용할 수는 없다. (SELECT가 후순위이기 때문에 앞단계에서 인식할 수 없기 때문)

- `Oracle` 에서는 `NULL` 값을 최대값으로 인식한다. (123P)

## 조인

- Oracle 에서는 모든 행이 출력되는 테이블의 반대편 테이블의 옆에 `(+)` 기호를 붙여준다. (128P)
- Where 절에서 Join 칼럼 옆에 (+)를 붙이면 OUTER JOIN이 된다. (130P)
  - 좌변이나 우변 중 하나에만 사용해야 한다. (150P)
- EQUI 조인에서 조인 칼럼 대체 가능 (130P) / Non EQUI 조인에서 서로 다른 값을 가질 수 있으므로 대체 불가능 (126P)
- Natural Join 은 같은 이름을 가진 칼럼들이 모두 동일한 데이터를 가지고 있을 시 Join이 된다. (142P)
  - Oracle 은 USING 조건절로 원하는 칼럼을 선택할 수 있다.
  - **단 USING 절로 정의된 칼럼 앞에는 별도의 ALIAS나 테이블명을 붙이지 않아야 한다. (중요, 문제)** (150P, 155P)
  - ON 절을 쓸 수 없다. (150P)
- 별도의 조인 조건이 없을 시 CROSS JOIN (147P)

## 적중 문제

- `NULL` 과의 비교 연산은 언제나 `False` 이다. (152P)
- `ALIAS` 를 지정하지 않으면, 칼럼명은 대문자로 출력된다. (166P)
- ORDER BY 절에 SELECT 절에 기술된 칼럼의 순서를 숫자로 명시해줄 수 있다. (167P)
- `AVG` / `MAX` 함수는 각각 `NULL` 값을 제외하고 집계한다. (168P)
- 테이블 전체가 한 개의 그룹이 디는 경우 `HAVING`만 단독으로 사용할 수 있다. (169P)

- 30번 문제. UNION 처럼 각 Join 조건을 Row 단위로 합침. 자세히는 모르겠음
