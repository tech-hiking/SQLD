## 1. 서브쿼리

- SELECT 절 : 스칼라 서브쿼리
- FROM 절 : 인라인 뷰
- WHERE 절 / HAVING 절 : 중첩 서브쿼리

- 비연관 서브쿼리 (Uncorrelated Subquery) : 메인 쿼리와 관계를 맺고 있지 않음
- 연관 서브쿼리 (Correlated Subquery) : 메인 쿼리와 관계를 맺고 있음

- 중첩 서브쿼리는 반환하는 데이터 형태 분류
  - **단일 행 서브쿼리**
    - 서브쿼리가 1건 이하 데이터를 반환
    - 단일 행 비교 연산자와 함께 사용
    - `=, <, >, <=, >=, <>`
  - **다중 행 서브쿼리**
    - 서브쿼리가 여러 건의 데이터를 반환
    - 다중 행 비교 연산자와 함께 사용
    - `IN, ALL, ANY, SOME, EXISTS`
  - **다중 칼럼 서브쿼리**
    - 서브쿼리가 여러 칼럼의 데이터를 반환

### Q. 스칼라 서브쿼리는 언제 쓰면 좋을까?

- 단일 행을 찾으므로 Join 과 같은 효과를 볼 수 있다.
- 부모 쿼리의 모든 행마다 외부 테이블을 조회해 비교한다.
- 경우에 따라 빠를 수도, 느릴 수도 있다는데 왜 그럴까?

쿼리가 실행되어 테이블을 반환하는 과정을 간략하면 다음과 같다.

- **쿼리 파싱 (Parsing)**
  - SQL 구문 분석 → AST(Abstract Syntax Tree) 생성
- **쿼리 최적화 및 실행 계획 수립 (Planning/Optimization)**
  - 통계 정보, 인덱스 존재 여부, 비용 추정 기반으로
    **최적의 실행 계획(Execution Plan)** 생성
  - 어떤 인덱스를 쓸지, 어떤 조인 전략을 쓸지 결정됨
- **쿼리 실행 (Execution)**
  - 실행 계획에 따라 데이터 블록 / 인덱스 블록 요청
- **스토리지 레이어에서 블록 단위 I/O**
  - **필요한 데이터 or 인덱스 블록을 디스크에서 읽음**
  - 먼저 **Buffer Pool(메모리)** 에서 찾고, 없으면 디스크에서 로딩
  - 읽은 블록은 Buffer Pool에 캐싱
- **임시 결과 생성**
  - **정렬, 그룹핑, 조인, 서브쿼리, 윈도우 함수** 등으로 인해
    중간 연산 결과를 메모리에 올려두거나,
  - 메모리 초과 시 **Temp Tablespace 등 디스크로 스필(Disk Spill) (스와핑 유사)**
- **결과 반환**
  - 처리된 결과를 **쿼리 요청 세션에 반환**

스칼라 서브쿼리 또는 조인의 성능 병목은 결국 **스토리지 레이어의 I/O 비용**과 **임시 결과 생성 및 연산에 소모되는 CPU 비용**에 의해 결정된다.
→ 따라서 단순히 "스칼라 서브쿼리를 썼다 vs 안 썼다" 만으로 성능을 일반화해 판단해선 안 된다.

예를 들어, 1억 개의 로우가 있는 테이블에 대해 **스칼라 서브쿼리를 수행**한다고 가정하자.

이때 서브쿼리 내부에서 사용하는 컬럼이 **적절한 인덱스로 커버되지 않았다면**, 각 로우마다 디스크에서 블록을 읽어야 할 수 있다.

- 이 작업이 **랜덤 액세스 패턴**을 유발할 경우,
  버퍼 풀(Cache) 미스율이 급격히 상승하고,
  결과적으로 **디스크 I/O 병목이 발생**한다.
- 더불어, 스칼라 서브쿼리는 **로우당 1회 반복 평가**되므로,
  CPU 비용까지 누적되어 **병렬처리되지 않으면 병목이 가속화**된다.

또한, 서브쿼리 내에서 **JOIN 또는 집계 연산**이 포함되거나,

**디스크 스필이 발생할 정도의 임시 테이블이 생성**된다면

**CPU, 메모리, 디스크 I/O 세 축 모두에서 병목이 발생**할 수 있다.

반대로 다음과 같은 상황이라면 스칼라 서브쿼리는 성능에 거의 영향을 주지 않을 수 있다.

1. **서브쿼리의 대상 테이블이 작다**

- 테이블의 **row 수가 수천~수만 수준 이하**이고,
- 인덱스를 타지 않더라도 **풀스캔 비용이 감당 가능한 수준**이면 부담이 적다.

2. **서브쿼리의 결과가 거의 동일하거나, 캐시 히트율이 높다**

- 예를 들어, 동일한 파라미터(외부 컬럼)에 대해 **결과가 반복되는 경우**,
  옵티마이저나 DB 엔진이 **결과를 메모리에 캐싱**하거나 **중복 실행을 회피**할 수 있다.
- PostgreSQL이나 MySQL 8.x 이상, Oracle 등은 **semi-join 변환, 쿼리 플래튼(flattening)** 같은 최적화도 시도한다.
  - 프로그래밍 언어 컴파일러가 컴파일 타임에 결정할 수 있는 인라인 함수 반환을 함수를 호출하는 대신 값으로 전개하는 기능과 유사

3. **버퍼풀(Cache Pool)이 충분하다**

- 자주 접근되는 인덱스/데이터 블록이 버퍼풀에 유지된다면, 디스크 I/O는 거의 발생하지 않는다.
- 이 경우 I/O 병목이 제거되어 **CPU 비용만으로도 충분히 감당** 가능해진다.

4. **서브쿼리 자체의 연산 복잡도가 낮다**

- 단순한 `SELECT column FROM table WHERE id = ?` 수준이면 CPU 사용량도 거의 없다.
- 정렬, 집계, JOIN 등이 없는 **단건 조회 수준의 서브쿼리**는 부담이 매우 낮다.

읽어보면 좋을 참고 아티클

1. [기억보다 기록을 : MySQL where in (서브쿼리) vs 조인 조회 성능 비교 (5.5 vs 5.6)](https://jojoldu.tistory.com/520)

## 2. 뷰

- 뷰는 투명성을 갖지 않는다. 사용자는 내부적으로 뷰를 생성하는 SQL을 볼 수 없기 때문이다.

```sql
CREATE OR REPLACE VIEW DEPT_MEBMER AS
	SELECT A.DEPARTMENT_ID,
		A.DEPARTMENT_NAME,
		B.FIRST_NAME,
		B.LAST_NAME
	FROM DEPARTMENTS A
	LEFT OUTER JOIN EMPLOYEES B
		ON A.DEPARTMENT_ID = B.DEPARTMENT_ID;
```

### Q. 뷰는 실제 데이터를 갖고 있을까?

- 뷰는 가상 테이블이다.
- 실제 데이터를 메모리(버퍼풀)에 저장하고 있지 않는다.
- 쿼리문만 갖고 있는 논리적 객체이자 **메타데이터**다.
- 뷰를 조회할 때 쿼리가 실행되고 그 결과는 임시 테이블처럼 메모리에 로딩한다.

구체화 뷰(Materialized View)와의 차이점

- 뷰와 다르게 결과 데이터를 **디스크에 저장**!
- 조회할 때마다 쿼리를 실행하는 게 아니라, 저장된 데이터를 읽기만 한다.
- **데이터가 바뀌면 수동으로 갱신해 주어야 함!**

| 구분             | View               | Materialized View         |
| ---------------- | ------------------ | ------------------------- |
| 실시간 쿼리 수행 | ✅ 예              | ❌ 아니오 (저장된 데이터) |
| 디스크 저장      | ❌ 없음            | ✅ 있음                   |
| 조회 속도        | 느릴 수 있음       | 빠름 (캐시처럼 동작)      |
| 갱신 필요        | 없음               | 수동 갱신 필요            |
| 주 용도          | 복잡한 쿼리 캡슐화 | 조회 성능 최적화          |

## 3. 집합 연산자

- UNION ALL : 중복된 행도 그대로 출력
- UNION : 중복된 행은 한 줄로 출력
- INTERSECT : 중복된 행은 한 줄로 출력
- MINUS / EXCEPT : 중복된 행은 한 줄로 출력

## 4. 그룹 함수

- 그룹 함수 > 소계(총계) 함수는 생소해 정리했습니다.
  - `ROLLUP` : 전체에서 뒤에서부터 하나씩 소거해 소계
    - `GROUP BY ROLLUP (A,B,C)` : (A,B,C) -> (A, B) -> (A) -> 총합계
  - `CUBE` : 높은 차수(개수)를 앞에서부터 선택해 소계
    - `GROUP BY CUBE(A,B,C)` : |3개 선택 (A,B,C) | -> |2개 선택 (A, B) -> (A, C) -> (B,C) | -> |1개 선택 (A) -> (B) -> (C) | -> (총합계)
  - `GROUPING SETS` : 인수 앞부터 차례대로 소계, `()` 은 총합계
    - `GROUP BY GROUPING SETS (A, ROLLUP(B,C))` : (A) -> (B,C) -> (B) -> 총합계
- 순위 함수
  - `RANK` : 1, 2, 2, 4, 5, 5, 7, ... -> 중복 순위 + 그만큼 다음 순위도 밀려남
  - `DENSE_RANK` : 1, 2, 2, 3, 4, 4, 5, ... -> 중복 순위 + 다음 순위는 그대로 증가
- 문제를 풀며 알게 된 팁
  - 소계 함수 문제 팁 : 하단 행과 `NULL` 칼럼의 패턴 보며 빠르게 유추 가능
    - 소계 함수 첫 번째 칼럼이 마지막 전까지 유지 : `ROLLUP` 확률 높음
    - 소계 함수 첫 번째 칼럼이 마지막 전까지 `NULL` : `CUBE` 확률 높음
    - 소계 함수 인수 순서대로 소계되어 `NULL` 칼럼의 패턴이 규칙적 : `GROUPING SETS` 확률 높음
- 공유하고 싶은 내용
  - `SUM` 파티셔닝 안의 `ORDER BY 칼럼` 이 들어갈 시 누적합 되는 이유는?
    - `WINDOWING 절` 기본값 : `RANGE UNBOUNDED PRECEDING` 이 기본값이기 때문
      - 의미 : 행 데이터를 기준으로 / 이전 행 전부를 포함
      - 이해 : 한 파티션을 Linked List 탐색으로 이해하면 수월
        - `UNBOUNDED PRECEDING` : 파티션 내 이전 행의 포인터를 끝날 때까지 무한히 탐색해 전체 집합을 도출
        - `UNBOUNDED FOLLOWING` : 파티션 내 다음 행의 포인터가 끝날 때까지 무한히 탐색해 전체 집합을 도출
        - `n PRECEDING` : 파티션 내 N개의 이전 행을 탐색해 집합 도출
        - `n FOLLOWING` : 파티션 내 N개의 다음 행을 탐색해 집합 도출

### Q. 소계함수는 내부적으로 어떻게 실행될까?

`UNION ALL` 로 같은 효과를 낼 수 있다.

```sql
SELECT k1, k2, SUM(k3) FROM t GROUP BY ROLLUP(k1, k2)

SELECT k1, k2, SUM(k3) FROM t GROUP BY k1, k2
UNION ALL
SELECT k1, NULL, SUM(k3) FROM t GROUP BY k1
UNION ALL
SELECT NULL, NULL, SUM(k3) FROM t

--

SELECT column1, column2, aggregate_function(column)
FROM table_name
GROUP BY CUBE(column1, column2)

GROUP BY GROUPING SETS(
  (column1, column2),
  (column1),
  (column2),
  ()
)

--

SELECT k1, k2, SUM(k3) FROM t GROUP BY GROUPING SETS((k1, k2), (k1), (k2), ())

SELECT k1, k2, SUM(k3) FROM t GROUP BY k1, k2
UNION ALL
SELECT k1, NULL, SUM(k3) FROM t GROUP BY k1
UNION ALL
SELECT NULL, k2, SUM(k3) FROM t GROUP BY k2
UNION ALL
SELECT NULL, NULL, SUM(k3) FROM t

```

MySQL에선 Rollup Level 을 조절해가며, 계층적으로 그룹 집계를 반복 생성한다.

- **첫 행을 읽어서 집계 시작**
  - 첫 행을 버퍼에 저장하고 그룹 기준을 캐시함.
  - `m_first_row_this_group` ← 현재 그룹
  - `m_first_row_next_group` ← 다음 그룹 후보
- **동일 그룹이면 계속 누적 (`aggregator_add()`)**
  - 다음 행을 읽고 group key가 같으면 누적만 하고 루프
- **그룹이 바뀌면**
  - 다음 그룹의 첫 row를 미리 읽고 `m_first_row_next_group`에 저장
  - 이전 그룹의 집계 결과를 **여러 번 출력**해야 함 → 이게 ROLLUP
  - 그래서 `m_state = OUTPUTTING_ROLLUP_ROWS`로 전환
- **ROLLUP 계층 출력**
  - `SetRollupLevel(...)`을 호출하며, `Item_rollup_sum_switcher`에 레벨별로 집계 결과를 설정
  - 각 레벨은 `NULL`을 집어넣어 상위 집계로 표현
- **ROLLUP이 끝나면 다음 그룹으로**

파일 : [composite_iterators.cc](https://github.com/mysql/mysql-server/blob/ff05628a530696bc6851ba6540ac250c7a059aa7/sql/iterators/composite_iterators.cc#L260)

```sql
int AggregateIterator::Read() {
  switch (m_state) {

	  // 첫 행 읽기

    case READING_FIRST_ROW: {
      // Start the first group, if possible. (If we're not at the first row,
      // we already saw the first row in the new group at the previous Read().)
      int err = m_source->Read();
      if (err == -1) {
        m_seen_eof = true;
        m_state = DONE_OUTPUTTING_ROWS;
        if (m_rollup && m_join->send_group_parts > 0) {
          // No rows in result set, but we must output one grouping row: we
          // just want the final totals row, not subtotals rows according
          // to SQL standard.
          SetRollupLevel(0);
          if (m_output_slice != -1) {
            m_join->set_ref_item_slice(m_output_slice);
          }
          return 0;
        }
        if (m_join->grouped || m_join->group_optimized_away) {
          SetRollupLevel(m_join->send_group_parts);
          return -1;
        } else {
          // If there's no GROUP BY, we need to output a row even if there are
          // no input rows.

          // Calculate aggregate functions for no rows
          for (Item *item : *m_join->get_current_fields()) {
            if (!item->hidden ||
                (item->type() == Item::SUM_FUNC_ITEM &&
                 down_cast<Item_sum *>(item)->aggr_query_block ==
                     m_join->query_block)) {
              item->no_rows_in_result();
            }
          }

          /*
            Mark tables as containing only NULL values for ha_write_row().
            Calculate a set of tables for which NULL values need to
            be restored after sending data.
          */
          if (m_join->clear_fields(&m_save_nullinfo)) {
            return 1;
          }
          for (Item_sum **item = m_join->sum_funcs; *item != nullptr; ++item) {
            (*item)->clear();
          }
          if (m_output_slice != -1) {
            m_join->set_ref_item_slice(m_output_slice);
          }
          return 0;
        }
      }
      if (err != 0) return err;

      // Set the initial value of the group fields.
      (void)update_item_cache_if_changed(m_join->group_fields);

      StoreFromTableBuffers(m_tables, &m_first_row_next_group);

      m_last_unchanged_group_item_idx = 0;
    }
      [[fallthrough]];

	  // 그룹이 바뀔 때

    case LAST_ROW_STARTED_NEW_GROUP:
      SetRollupLevel(m_join->send_group_parts);

      // We don't need m_first_row_this_group for the old group anymore,
      // but we'd like to reuse its buffer, so swap instead of std::move.
      // (Testing for state == READING_FIRST_ROW and avoiding the swap
      // doesn't seem to give any speed gains.)
      swap(m_first_row_this_group, m_first_row_next_group);
      LoadIntoTableBuffers(
          m_tables, pointer_cast<const uchar *>(m_first_row_this_group.ptr()));

      for (Item_sum **item = m_join->sum_funcs; *item != nullptr; ++item) {
        if (m_rollup) {
          if (down_cast<Item_rollup_sum_switcher *>(*item)
                  ->reset_and_add_for_rollup(m_last_unchanged_group_item_idx))
            return true;
        } else {
          if ((*item)->reset_and_add()) return true;
        }
      }

      // Invalidate the cache in EQRefIterators, if needed. The call to
      // LoadIntoTableBuffers() above would usually restore the cache correctly
      // to the values it had just after the previous call to m_source->Read().
      // However, if the row was NULL-complemented, LoadIntoTableBuffers() will
      // have overwritten the cached values with NULLs, and the cache must be
      // invalidated.
      for (AccessPath *lookup : m_single_row_index_lookups) {
        if (lookup->eq_ref().table->has_null_row()) {
          lookup->eq_ref().ref->key_err = true;
        }
      }

			// 현재 그룹이 유지되는 동안 계속 행 읽기 수행

      // Keep reading rows as long as they are part of the existing group.
      for (;;) {
        int err = m_source->Read();
        if (err == 1) return 1;  // Error.

        if (err == -1) {
          m_seen_eof = true;

          // End of input rows; return the last group. (One would think this
          // LoadIntoTableBuffers() call is unneeded, since the last row read
          // would be from the last group, but there may be filters in-between
          // us and whatever put data into the row buffers, and those filters
          // may have caused other rows to be loaded before discarding them.)
          LoadIntoTableBuffers(m_tables, pointer_cast<const uchar *>(
                                             m_first_row_this_group.ptr()));

          if (m_rollup && m_join->send_group_parts > 0) {
            // Also output the final groups, including the total row
            // (with NULLs in all fields).
            SetRollupLevel(m_join->send_group_parts);
            m_last_unchanged_group_item_idx = 0;
            m_state = OUTPUTTING_ROLLUP_ROWS;
          } else {
            SetRollupLevel(m_join->send_group_parts);
            m_state = DONE_OUTPUTTING_ROWS;
          }
          if (m_output_slice != -1) {
            m_join->set_ref_item_slice(m_output_slice);
          }
          return 0;
        }

        int first_changed_idx =
            update_item_cache_if_changed(m_join->group_fields);
        if (first_changed_idx >= 0) {
          // The group changed. Store the new row (we can't really use it yet;
          // next Read() will deal with it), then load back the group values
          // so that we can output a row for the current group.
          // NOTE: This does not save and restore FTS information,
          // so evaluating MATCH() on these rows may give the wrong result.
          // (Storing the row ID and repositioning it with ha_rnd_pos()
          // would, but we can't do the latter without disturbing
          // ongoing scans. See bug #32565923.) For the old join optimizer,
          // we generally solve this by inserting temporary tables or sorts
          // (both of which restore the information correctly); for the
          // hypergraph join optimizer, we add a special streaming step
          // for MATCH columns.
          StoreFromTableBuffers(m_tables, &m_first_row_next_group);
          LoadIntoTableBuffers(m_tables, pointer_cast<const uchar *>(
                                             m_first_row_this_group.ptr()));

          // If we have rollup, we may need to output more than one row.
          // Mark so that the next calls to Read() will return those rows.
          //
          // NOTE: first_changed_idx is the first group value that _changed_,
          // while what we store is the last item that did _not_ change.
          if (m_rollup) {
            m_last_unchanged_group_item_idx = first_changed_idx + 1;
            if (static_cast<unsigned>(first_changed_idx) <
                m_join->send_group_parts - 1) {
              SetRollupLevel(m_join->send_group_parts);
              m_state = OUTPUTTING_ROLLUP_ROWS;
            } else {
              SetRollupLevel(m_join->send_group_parts);
              m_state = LAST_ROW_STARTED_NEW_GROUP;
            }
          } else {
            m_last_unchanged_group_item_idx = 0;
            m_state = LAST_ROW_STARTED_NEW_GROUP;
          }
          if (m_output_slice != -1) {
            m_join->set_ref_item_slice(m_output_slice);
          }
          return 0;
        }

        // Give the new values to all the new aggregate functions.
        for (Item_sum **item = m_join->sum_funcs; *item != nullptr; ++item) {
          if (m_rollup) {
            if (down_cast<Item_rollup_sum_switcher *>(*item)
                    ->aggregator_add_all()) {
              return 1;
            }
          } else {
            if ((*item)->aggregator_add()) {
              return 1;
            }
          }
        }

        // We're still in the same group, so just loop back.
      }

    case OUTPUTTING_ROLLUP_ROWS:
      SetRollupLevel(m_current_rollup_position - 1);

      if (m_current_rollup_position <= m_last_unchanged_group_item_idx) {
        // Done outputting rollup rows; on next Read() call, deal with the new
        // group instead.
        if (m_seen_eof) {
          m_state = DONE_OUTPUTTING_ROWS;
        } else {
          m_state = LAST_ROW_STARTED_NEW_GROUP;
        }
      }

      if (m_output_slice != -1) {
        m_join->set_ref_item_slice(m_output_slice);
      }
      return 0;

    case DONE_OUTPUTTING_ROWS:
      if (m_save_nullinfo != 0) {
        m_join->restore_fields(m_save_nullinfo);
        m_save_nullinfo = 0;
      }
      SetRollupLevel(INT_MAX);  // Higher-level iterators up above should not
                                // activate any rollup.
      return -1;
  }

  assert(false);
  return 1;
}

void AggregateIterator::SetRollupLevel(int level) {
  if (m_rollup && m_current_rollup_position != level) {
    m_current_rollup_position = level;
    for (Item_rollup_group_item *item : m_join->rollup_group_items) {
      item->set_current_rollup_level(level);
    }
    for (Item_rollup_sum_switcher *item : m_join->rollup_sums) {
      item->set_current_rollup_level(level);
    }
  }
}
```

### Q. WINDOWING 절은 어떻게 관리되고 있을까?

그룹별로 서로 다른 파티션을 가지므로, 내부적으로 링크드 리스트 기반이지 않을까 싶었다.
실제 구현은 예상과는 달랐다.

PostgreSQL는 윈도우 파티션을 `WindowAggState` 구조체로 관리한다. ([src/include/nodes/execnodes.h](https://github.com/postgres/postgres/blob/master/src/include/nodes/execnodes.h#L2622C1-L2708C18))

```sql
typedef struct WindowAggState
{

	...
	Tuplestorestate *buffer;	/* stores rows of current partition */
	int64		spooled_rows;	/* total # of rows in buffer */
	int			current_ptr;	/* read pointer # for current row */

	int64		frameheadpos;	/* current frame head position */
	int64		frametailpos;	/* current frame tail position (frame end+1) */

	...

	ExprState  *startOffset;	/* expression for starting bound offset */
	ExprState  *endOffset;		/* expression for ending bound offset */
	Datum		startOffsetValue;	/* result of startOffset evaluation */
	Datum		endOffsetValue; /* result of endOffset evaluation */

	/* these fields are used with RANGE offset PRECEDING/FOLLOWING: */
	FmgrInfo	startInRangeFunc;	/* in_range function for startOffset */
	FmgrInfo	endInRangeFunc; /* in_range function for endOffset */
	Oid			inRangeColl;	/* collation for in_range tests */
	bool		inRangeAsc;		/* use ASC sort order for in_range tests? */
	bool		inRangeNullsFirst;	/* nulls sort first for in_range tests? */

	...

	bool		framehead_valid;	/* true if frameheadpos is known up to
									 * date for current row */
	bool		frametail_valid;	/* true if frametailpos is known up to
									 * date for current row */
	...

} WindowAggState;
```

- 윈도우 파티션 마다 `Tuplestorestate` 라는 슬라이딩 버퍼 큐에 행을 저장한다.

  - 디스크 기반 스풀링 큐로 동작
  - 메모리에 담긴 파티션 행들을 저장함 (메모리 용량이 넘쳐 필요 시 디스크에 spill 가능)
  - 각 row는 인덱스 기반으로 접근함 → **`currentpos`, `frameheadpos`, `frametailpos` 등은 이 인덱스에 대한 포인터**
    - 슬라이딩 윈도우 구간을 지정

- 링크드 리스트보다는 배열과 유사한 형태
  - **인덱스 포인터 + Tuplestore 버퍼** 구조
  - **슬라이딩 윈도우 방식**으로 N preceding / following 을 효율적으로 탐색
- **row 접근 순서 보장 + 범위 유지 + 디스크 I/O 최소화**를 목표로 설계됨

만약 카디널리티가 높은 칼럼을 선택한다면 어떻게 될까?

- 각 파티션마다 들어갈 행 수가 적어져, 파티션마다 처리 속도는 빠르다.
- 그러나 `WindowAggState` 구조체가 더 많이 생성되어 메모리 사용량이 증가하고, 상태를 추적하고 관리하고 집계 연산을 수행하는 자원 소모 비용이 커진다.
  - 다음 파티션을 만나면 버퍼를 비우고 집계를 초기화 하는 등의 컨텍스트 스위칭 CPU 비용도 있음
- 따라서 많은 파티션이 있을 수록 메모리 상에 처리할 수 없고, 디스크에 Spill 할 가능성이 높아진다.
- 디스크에 Spill 한다는 건 `WindowAggState` 중간 처리 결과를 Disk I/O 로 저장하고, 불러오는 비용이 생긴다.

반대로 카디널리티가 적은 칼럼을 선택한다면 어떻게 될까?

- 적은 수의 `WindowAggState` 구조체 사용으로 파티션 처리 효율이 높아질 수 있다.

Window 함수에 가장 크게 성능에 영향을 끼치는 건, 수행되는 행 수에 달려있다.

- 행 수 : I/O 양, 디스크에서 읽어야 하는 물리적인 행 수
- 필터링 효율 : WHERE / JOIN 절에서 얼마나 많은 행이 걸러지는지
- 정렬 / 집계 대상 행 수 : 정렬, 집계 대상 행이 많으면 CPU / 메모리 비용 증가
- 중간 결과 재사용 여부 : 메모리 버퍼풀에 저장된 서브쿼리 / CTE(WITH) / Temp Table 로 재사용이 얼마나 가능한지
  - PostgreSQL 기준 CTE 가 내부 서브쿼리로 인라인될 수 있음 ( 사용할 때마다 재계산 )

## 5. WINDOWING 절 정리

- 기본값 : RANGE 방식의 `UNBOUNDED PRECEDING`
- BETWEEN A AND B : A 와 B 사이 ( Inclusive)
- UNBOUNDED [PRECEDING] / UNBOUNDED FOLLOWING : 파티션의 처음과 끝
- N PRECEDING / N FOLLOWING : 이전 행 N개, 이후 행 N개 이동
- CURRENT ROW : 현재 행

- ROW : 행 자체가 기준
- RANGE : 행의 데이터 값이 기준
  - `RANGE BETWEEN 10 PRECEDING AND CURRENT ROW` : **_현재 행의 값보다 10만큼 적은 행_** 부터 현재 행까지

## 6. LAG / LEAD

- LAG : 특정 수만큼 앞의 데이터를 구함
- LEAD : 특정 수만큼 뒤의 데이터를 구함

## 7. 비율 함수

- `RATIO_TO_REPORT` : 합계에서 차지하는 비율

  - `SCORE / SUM(SCORE)` 와 동일

- `PERCENT_RANK` : 파티션의 맨 위를 0, 맨 아래를 1로 놓고 현재 행의 백분위 순위 값

  - `(RANK - 1) / (COUNT - 1)`
  - `( RANK() OVER (ORDER BY SCORE) - 1 ) / ( COUNT(*) OVER() - 1)`

- `CUME_DIST` (Cumulative Distribution, 누적 분포 함수) : 현재 파티션에서의 누적 백분율

  - 0보다 크고 1보다 작거나 같은값을 가진다.
  - `COUNT / TOTAL_COUNT`
  - `COUNT(*) OVER (ORDER BY SCORE) / COUNT(*) OVER()`

- `NTILE` : 주어진 수만큼 행들을 n등분한 후에 현재 행에 해당하는 등급을 구함
  - 할당할 행이 남으면, 맨 앞 그룹부터 하나씩 더 채워진다.
  - 10개의 행을 `NTIME(3) OVER(PARTITION BY SUBJECT ORDER BY SCORE DESC)`
  - `1 1 2 2 3` (5개 파티션행)
  - 5개의 행을 2개로 나눈다면? `1 1 1 2 2`

## 8. Top-N 쿼리

- SELECT 절의 순서(5번째)는 `ROWNUM`은 WHERE 절 순서(2번째)보다 뒤이기 때문에, 동등 조건은 사용할 수 없다.
- 또한 `ORDER` 절이 마지막 순서이기에, `ROWNUM`이 붙여진 이후에 정렬을 하게 될 시 깨질 수 있다.
  - -> 정렬 전 데이터로 무작위 순서 ROWNUM 을 붙인 꼴이 된다.
- 따라서 FROM 절의 정렬 적용한 서브 쿼리로 넣고, 해당 서브 쿼리의 `ROWNUM` 으로 필터링한다.

## 9. 셀프 조인 / 계층 쿼리

같은 테이블을 조인한다. ALIAS를 표기해 중복 칼럼을 방지한다.
주로 계층 구조를 따르는 테이블 쿼리에 용이하다.

- (276P. 문제) 특정 직원(EMPLOYEE)이 매니저 직원(MANAGER)이니, A 테이블로 매니저를 표현해야 한다.

( 오라클 기준 )

- `LEVEL` : 현재 DEPTH / 루트 노드는 1
- `SYS_CONNECT_BY_PATH (컬럼, 구분자)` : 루트 노드부터 현재 노드까지의 경로를 출력
  - MySQL 기준 `GROUP_CONCAT`
- `START WITH` : 경로가 시작되는 루트 노드를 생성해주는 절
- `CONNECT BY` : 루트로부터 자식 노드를 생성해주는 절 / 조건에 만족하는 데이터가 없을 때 까지 노드를 생성
- `PRIOR` ; 바로 앞에 있는 부모 노드의 값을 반환

- `CONNECT_BY_ROOT` : 루트 노드의 주어진 칼럼 값을 반환
- `CONNECT_BY_ISLEAFT` : 가장 하위 노드면 1 / 그 외는 0 반환

계층별 정렬은 `ORDER SIBLINGS BY ~` 절을 사용한다.

## 10. PIVOT / UNPIVOT

- PIVOT : 원본 데이터를 행에서 열로 변환해 데이터를 집계함
- UNPIVOT : 집계된 데이터를 열에서 행으로 변환함

```sql
-- Pivot
SELECT *
    FROM (SELECT GRADE, DEPT_NAME, SALARY FROM EMP_INFO)
    PIVOT (SUM(SALARY) FOR DEPT_NAME IN ('인사팀', 'IT개발팀', 'AI연구팀', '클라우드팀'))
    ORDER BY GRADE;

-- Unpivot
SELECT GRADE, DEPT_NAME, SALARY
    FROM EMP_INFO_PIVOT
    UNPIVOT (SALARY FOR DEPT_NAME IN (HR_SAL, IT_SAL, AI_SAL, CLOUD_SAL))
    ORDER BY GRADE;
```

- `PIVOT (SUM(SALARY) FOR DEPT_NAME IN ('인사팀', 'IT개발팀', 'AI연구팀', '클라우드팀'))`

  - SUM(~) : 집계 데이터
  - FOR 절 : PIVOT 할 칼럼
  - IN 절 : 헤더에 표시할 PIVOT 데이터

- Pivot : `피벗값(Aliasing 가능)_피벗칼럼(Aliasing 가능)` 으로 변경
- Unpivot : 위 `XXX_YYY` 를 인식해 데이터로 만들어줌

## 11. 정규식

기본 연산자

- `.` : 임의의 한 문자
- `|` : or
- `\` : 이스케이핑 -> 일반 문자로 취급
- `^` : 문자열의 시작
- `$` : 문자열의 끝
- `?` : 선행 문자 0개 / 1개
- `*` : 선행 문자 0개 이상
- `+` : 선행 문자 1개 이상

패턴 연산자

- `[]` : 대괄호 안 문자 중 하나와 일치
- `[-]` : 연속 문자의 범위를 지정
- `[^]` : 대괄호 안 문자들을 제외한 나머지 문자 중 하나와 일치
- `()` : 소괄호로 묶인 표현식을 한 단위로 취급

POSIX 문자 클래스

- `[:digit:]` : `[0-9]`
- `[:lower:]` : `[a-z]`
- `[:upper:]` : `[A-Z]`
- `[:alpha:]` : `[a-zA-Z]`
- `[:alnum:]` : `[0-9a-zA-Z]`
- `[:xdigit:]` : `[0-9a-fA-F]` (16진수)
- `[:punct:]` : `[^[:alnum:][:cntrl:]]`
  - 구두점 문자 : 문장의 구조를 나타내기 위해 사용하는 기호 / 특수 문자 포함
- `[:blank:]` : 공백 문자(White space)
- `[:space:]` : 공간 문자(Space, Enter, Tab)

함수

- `REGEXP_SUBSTR` : 문자열을 추출 / 매칭 없을 시 `NULL`
  - `REGEXP_SUBSTR('~~~', 1, 1, 'i', 1)`
    - 매개변수 순서대로 설명
    - 1 : 검색 시작 위치
    - 1 : 첫 번째로 일치하는 패턴
    - 'i' : 대소문자 구분 없이 검색
    - 1 : 첫 번째 그룹과 일치
- `REGEXP_REPLACE` : 문자열 대체
- `REGEXP_INSTR` : 위치 반환 (인덱스가 아니라, 1부터 시작)
  - 마찬가지로 시작 순서(위치) / 그룹을 추가로 매개변수 설정
- `REGEXP_COUNT` : 몇 번 나타나는지 카운팅
- `REGEXP_LIKE` : 정규식으로 필터링
  - `WHERE REGEXP_LIKE (FIRST_NAME, '^Ste(v|ph)en$')`
  - Stephen / Steven / Steven ...
