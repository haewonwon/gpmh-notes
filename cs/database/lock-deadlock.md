---
track: CS
notion: https://app.notion.com/p/416178c4afac81f28862e33bd3b1cc4e
---

# Lock / Deadlock

<callout icon="📍" color="gray_bg">
**학습 범위**: Database · 7/7 · 잠금의 호환성, 대기와 교착 상태

보호할 자원과 잠금 범위·유지 시간을 구분하고 두 세션의 대기 관계를 추적한다. SQL과 관찰 도구는 PostgreSQL 18 기준이며 InnoDB의 규칙은 별도로 표시한다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 잠금의 모드·단위·유지 시간과 MVCC의 역할을 구분한다.
- 일반 SELECT와 FOR UPDATE, UPDATE의 충돌을 두 세션에서 관찰한다.
- 단방향 대기와 순환 대기를 그래프로 구분한다.
- 같은 자원 순서, 짧은 트랜잭션, 오류별 전체 재시도로 교착 상태에 대응한다.
- 세션·잠금 정보를 읽어 막는 쪽과 기다리는 쪽을 찾아낸다.

## 2. 이 주제를 왜 배우는가

재고를 잘못 덮어쓰지 않도록 잠금을 추가했더니 요청이 멈추거나 간헐적으로 교착 상태 오류가 날 수 있다. 이것은 잠금이 무의미하다는 뜻이 아니다. 충돌을 조정하는 과정에서 누가 무엇을 보유하고 누구를 기다리는지까지 이해해야 한다는 뜻이다.

잠금은 단순히 “동시에 실행하지 못하게 한다”보다 구체적인 계약이다. 같은 행을 읽는 것은 허용하되 바꾸는 것은 기다리게 할 수도 있고, 테이블 전체의 구조 변경을 제한할 수도 있다. 이번 페이지에서는 그 범위와 해제를 함께 추적한다.

먼저 관리자 세션에서 별도의 학습 구조를 만든다. A·B·관찰용 세션은 같은 DB에 접속하고 각각 `SET search_path TO db_lock_lab`을 실행한다. 기존 세션의 트랜잭션을 끝낸 뒤 다음 실험을 진행한다.

```sql
CREATE SCHEMA db_lock_lab;
SET search_path TO db_lock_lab;
CREATE TABLE products (
    id integer PRIMARY KEY,
    stock integer NOT NULL CHECK (stock >= 0),
    version integer NOT NULL DEFAULT 0
);
INSERT INTO products (id, stock) VALUES (10, 10), (20, 20);
```

## 3. 잠금과 교착 상태의 기본 흐름

### 잠금은 충돌하는 접근을 조정한다

**잠금(lock)** 은 같은 자원에 대한 동시 접근 중 함께 허용할 수 없는 작업을 조정하는 장치다. 다른 트랜잭션이 필요한 잠금을 이미 보유했다면 기다리거나 설정에 따라 실패할 수 있다.

대표적인 공유·배타 잠금 모델은 다음과 같다. 여기서 호환성은 서로 다른 트랜잭션이 같은 자원의 잠금을 요청하는 경우다.

| 이미 가진 잠금 / 새 요청 | 공유 잠금 S | 배타 잠금 X |
| --- | --- | --- |
| 공유 잠금 S | 함께 보유 가능 | 충돌 |
| 배타 잠금 X | 충돌 | 충돌 |

공유 잠금은 여러 읽기를 함께 허용하고, 배타 잠금은 충돌하는 변경 등을 제한한다. 이 표는 기본 모델이며 실제 DBMS에는 더 세분된 모드가 있다. 특히 MVCC를 쓰는 DBMS에서 **일반 SELECT가 항상 이 표의 S 잠금을 행마다 획득한다고 가정하면 안 된다.** [MySQL InnoDB의 S/X 잠금 설명](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)을 참고한다.

### 잠금의 단위와 유지 시간을 구분한다

행 잠금은 특정 행의 충돌을 조정하므로 서로 다른 행의 작업을 병행하기에 유리하다. 테이블 잠금은 더 넓은 작업과 충돌을 조정하며 스키마 변경 등에도 필요하다. 좁은 잠금은 관리할 대상이 많아질 수 있고, 넓은 잠금은 병행할 수 있는 작업을 줄일 수 있다.

PostgreSQL의 행 잠금은 일반적으로 트랜잭션이 끝날 때 해제된다. 행 잠금 때문에 다른 갱신이나 잠금 조회는 기다릴 수 있지만, 일반 SELECT는 MVCC에 따라 읽을 수 있다. 다만 테이블의 강한 잠금 등으로 일반 조회도 막힐 수 있으므로 “읽기는 절대 기다리지 않는다”로 일반화하지 않는다. [PostgreSQL Explicit Locking](https://www.postgresql.org/docs/18/explicit-locking.html)을 참고한다.

### SELECT FOR UPDATE로 조회 뒤의 결정을 보호한다

이미 존재하는 상품 행을 읽고 여러 판단을 거쳐 바꿔야 한다면 PostgreSQL에서 다음과 같이 잠금 조회를 사용할 수 있다.

```sql
BEGIN;

SELECT id, stock
FROM products
WHERE id = 10
FOR UPDATE;

-- 애플리케이션이 재고를 확인한다.
-- 부족하면 ROLLBACK하고 아래 갱신을 실행하지 않는다.
UPDATE products SET stock = stock - 1 WHERE id = 10;

COMMIT;
```

다른 세션이 같은 행을 갱신하려 하면 이 트랜잭션의 종료를 기다리게 된다. 일반 SELECT를 실행한 뒤 별개 트랜잭션에서 UPDATE하는 것은 같은 보호를 제공하지 않는다. 간단한 재고 감소라면 앞에서 본 조건부 UPDATE 하나로 판단과 변경을 표현하는 방법도 있다.

**없는 행을 조회했다고 그 자리에 앞으로 생길 행까지 자동으로 잠그는 것은 아니다.** PostgreSQL에서 “이 이메일의 회원이 없네”를 `FOR UPDATE`로 확인한 것만으로 동시 가입을 막을 수 없다. 이메일의 `UNIQUE` 제약처럼 지켜야 할 규칙을 직접 표현해야 한다.

### 기다림과 교착 상태는 다르다

T2가 T1의 잠금 해제를 기다리고 T1은 계속 실행할 수 있다면 일반적인 **잠금 대기(lock wait)** 다. **교착 상태(deadlock)** 는 여러 트랜잭션이 서로가 가진 잠금을 기다리는 순환 구조 때문에 누구도 진행하지 못하는 상황이다.

상품 10과 상품 20을 수정하는 두 트랜잭션을 보자.

| 순서 | T1 | T2 |
| --- | --- | --- |
| 1 | 상품 10을 잠금 | |
| 2 | | 상품 20을 잠금 |
| 3 | 상품 20 잠금 요청 → T2를 기다림 | |
| 4 | 계속 대기 | 상품 10 잠금 요청 → T1을 기다림 |

```text
T1 -- 상품 20의 잠금을 기다림 --> T2
T2 -- 상품 10의 잠금을 기다림 --> T1
```

교착 상태를 만드는 핵심은 순환 대기다. 요청이 오래 걸린다는 사실만으로 교착 상태라고 판단하지 않는다. 한쪽이 외부 응답을 오래 기다리며 잠금을 붙잡는 문제도 오래 걸리지만 이 순환 구조가 없을 수 있다.

### 두 세션에서 교착 상태 관찰하기

두 상품 행이 있는 상태에서 아래 명령을 표의 순서대로 실행한다. 결과를 기다리는 명령 뒤의 조작은 다른 세션에서 진행한다.

| 순서 | 실행 세션 | 실행할 SQL |
| --- | --- | --- |
| 1 | A | `BEGIN;` |
| 2 | A | `SELECT id FROM products WHERE id = 10 FOR UPDATE;` |
| 3 | B | `BEGIN;` |
| 4 | B | `SELECT id FROM products WHERE id = 20 FOR UPDATE;` |
| 5 | A | `SELECT id FROM products WHERE id = 20 FOR UPDATE;` — 대기 |
| 6 | B | `SELECT id FROM products WHERE id = 10 FOR UPDATE;` — 순환 형성 |

PostgreSQL은 교착 상태를 감지하면 한 트랜잭션을 중단하여 순환을 끊는다. 어느 쪽이 중단될지에 의존해 프로그램을 작성하면 안 된다. 중단된 세션은 `ROLLBACK`으로 정리하고, 살아남은 세션도 실험이 끝나면 `ROLLBACK`하여 잠금을 해제한다. [PostgreSQL Deadlocks](https://www.postgresql.org/docs/18/explicit-locking.html#LOCKING-DEADLOCKS)를 참고한다.

### 교착 상태를 줄이고 실패에 대응한다

- **잠금을 같은 순서로 획득한다.** 여러 상품을 수정할 때 모든 경로가 상품 ID 오름차순으로 행을 잠그는 식의 규칙을 정한다.
- **트랜잭션을 짧게 유지한다.** 사용자 입력이나 느린 외부 API 응답을 기다리며 잠금을 오래 보유하지 않는다.
- **접근 범위를 줄인다.** 적절한 조건과 인덱스로 불필요한 검색과 잠금 유지 시간을 줄인다.
- **중단될 수 있음을 처리한다.** 교착 상태 감지는 충돌을 없애는 기능이 아니라 진행을 위해 한쪽을 실패시키는 기능이다.

이 원칙은 [MySQL Deadlocks in InnoDB](https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks.html)에서도 확인할 수 있다. 격리 수준을 바꾼다는 이유만으로 교착 상태가 사라진다고 기대하지 않는다.

PostgreSQL에서 직렬화 실패의 SQLSTATE는 `40001`, 교착 상태는 `40P01`이다. 재시도할 때는 마지막 SQL만 다시 보내는 것이 아니라 **판단에 사용한 조회부터 트랜잭션 전체를 다시 실행**해야 한다. 재시도 횟수와 간격에 제한을 두고, 이미 외부에서 일어난 결제 같은 부수 효과를 중복 실행하지 않도록 구분한다. [PostgreSQL Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html)을 참고한다.

## 4. 잠금의 모드와 단위는 서로 다른 축이다

### 모드는 어떤 접근과 충돌하는지를 나타낸다

공유·배타 모델을 이해한 뒤에는 제품의 실제 호환성을 확인한다. PostgreSQL의 행 잠금에는 다음과 같은 구분이 있다.

| 행 잠금 모드 | 기본 용도와 보호 관점 |
| --- | --- |
| `FOR KEY SHARE` | 참조 키를 바꾸거나 행을 없애는 변경을 제한하는 약한 보호 |
| `FOR SHARE` | 다른 세션의 관련 행 갱신·삭제를 제한하며 공유 가능 |
| `FOR NO KEY UPDATE` | 키를 바꾸지 않는 갱신 등에 대응하는 보호 |
| `FOR UPDATE` | 같은 행에 대한 다른 잠금 조회·변경과 강하게 충돌 |

이 표는 모든 호환 조합을 대체하는 암기 표가 아니다. 어떤 변경을 막아야 하는지부터 정하고 공식 호환성 표로 확인한다. 가장 강한 모드를 항상 선택하면 필요한 보호보다 많은 작업을 기다리게 만들 수 있다. [Explicit Locking의 Row-Level Locks](https://www.postgresql.org/docs/18/explicit-locking.html#LOCKING-ROWS)를 참고한다.

### 단위는 무엇을 잠그는지를 나타낸다

행 잠금과 테이블 잠금은 보호 자원의 범위가 다르다. PostgreSQL의 `ROW EXCLUSIVE`라는 이름은 이름에 row가 들어가도 **테이블 수준 모드**다. SQL의 UPDATE가 테이블 모드와 대상 행의 잠금을 함께 획득할 수 있으므로 “한 문장은 언제나 잠금 하나만 가진다”라고 생각하지 않는다.

테이블 잠금도 모두 테이블의 모든 읽기를 막는 것은 아니다. 어떤 모드와 충돌하는지가 중요하다. `ACCESS EXCLUSIVE`처럼 일반 SELECT의 테이블 잠금과도 충돌하는 강한 모드가 있고, 읽기와 함께 허용되는 모드도 있다. [LOCK](https://www.postgresql.org/docs/18/sql-lock.html)을 참고한다.

### 유지 시간도 별도로 확인한다

트랜잭션용 행 잠금은 일반적으로 COMMIT·ROLLBACK까지 유지된다. UPDATE 문장의 실행이 끝났다고 바로 풀리는 것으로 가정하지 않는다. PostgreSQL에는 내부 페이지 보호처럼 더 짧은 기간의 잠금과, 명시적으로 관리하는 다른 종류의 잠금도 있으므로 모든 잠금의 수명이 같다고 일반화하지 않는다.

응용 코드에서 중요한 질문은 “이 잠금은 어느 트랜잭션에서 언제 획득했고 어떤 종료 경로에서 해제되는가”다. 예외가 나거나 사용자 요청이 끊겼을 때도 DB 세션이 끝났는지 확인해야 한다.

## 5. 같은 행을 읽는 일과 바꾸는 일을 직접 비교하기

세션 A에서 다음을 실행하고 트랜잭션을 유지한다.

```sql
BEGIN;
SELECT id, stock FROM products WHERE id = 10 FOR UPDATE;
```

세션 B의 일반 SELECT는 PostgreSQL의 MVCC 규칙에 따라 기존 확정 값을 읽을 수 있다.

```sql
SELECT id, stock FROM products WHERE id = 10;
```

그러나 같은 행을 바꾸려는 다음 명령은 A의 종료를 기다린다.

```sql
UPDATE products SET stock = stock - 1 WHERE id = 10;
```

A가 ROLLBACK하면 A의 잠금이 해제되어 B의 UPDATE가 진행할 수 있다. A가 COMMIT하는 경우에도 잠금은 풀리지만 A가 실제 값을 바꾸었다면 B의 동작과 조건 재검사를 함께 확인해야 한다.

이 실험에는 대기 관계 `B -> A`만 있다. A는 B가 가진 자원을 기다리지 않으므로 순환 대기가 아니다. 오래 기다린다는 이유만으로 교착 상태라고 부를 수 없는 가장 작은 예다.

## 6. NOWAIT와 SKIP LOCKED는 서로 다른 결과를 만든다

A가 상품 10의 FOR UPDATE 잠금을 가진 동안 B에서 각각 독립적으로 실행해 보자.

```sql
SELECT id FROM products WHERE id = 10 FOR UPDATE NOWAIT;
```

필요한 행 잠금을 즉시 얻을 수 없으면 기다리지 않고 오류가 난다. 이 동작은 “잠금을 생략하고 최신 값을 읽는다”가 아니다. 명시적 트랜잭션 안에서 오류가 났다면 그 실패 상태도 정리해야 한다.

```sql
SELECT id FROM products
WHERE id IN (10, 20)
ORDER BY id
FOR UPDATE SKIP LOCKED;
```

이 조회는 잠긴 10을 건너뛰고 잠글 수 있는 20을 반환할 수 있다. 대기하지 않는 대신 결과 집합이 달라진다. 따라서 일반적인 전체 목록이나 정확한 전체 재고를 읽는 용도로 무조건 바꾸면 안 된다. 여러 작업자가 처리 가능한 작업만 나누어 가져가는 종류의 요구에서 의미를 검토할 수 있다.

NOWAIT·SKIP LOCKED는 관련 행 잠금의 동작을 조정하는 옵션이다. 필요한 모든 테이블 잠금까지 어떤 상황에서도 전혀 기다리지 않는다는 의미로 넓히지 않는다. 정확한 범위는 [SELECT의 Locking Clause](https://www.postgresql.org/docs/18/sql-select.html#SQL-FOR-UPDATE-SHARE)를 참고한다.

## 7. 없는 행과 조건의 범위를 어떻게 보호할 것인가

이메일이 없는지 조회한 결과가 0행이면 PostgreSQL FOR UPDATE가 보호할 기존 행도 없다. 두 요청이 동시에 0행을 보고 같은 이메일로 INSERT할 수 있으므로 UNIQUE 같은 데이터 제약이 필요하다.

이 문제는 pending 주문의 모든 기존 행을 잠근 뒤 새로운 pending 주문이 생기지 않는다고 기대하는 상황과도 연결된다. **현재 존재하는 행을 잠그는 것과 검색 조건의 미래 구성원까지 제한하는 것은 다르다.** 필요한 보장을 제품의 범위 제어·격리·제약 설계와 함께 검토한다.

MySQL InnoDB는 인덱스 레코드뿐 아니라 범위의 gap·next-key 잠금을 설명한다. 이것을 PostgreSQL FOR UPDATE가 같은 방식으로 동작한다는 근거로 쓰면 안 된다. 이름이 같은 SQL 문법이 있어도 엔진의 접근 경로와 격리 수준에 따라 범위가 달라질 수 있다. [InnoDB Locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html)을 참고한다.

## 8. 교착 상태를 대기 그래프로 읽기

그래프의 노드를 트랜잭션으로 놓고, T1이 T2 때문에 진행할 수 없으면 `T1 -> T2` 화살표를 그린다. 단방향 사슬의 끝에 진행 가능한 트랜잭션이 있으면 그 작업이 끝나며 대기가 풀릴 수 있다.

```text
T3 -> T2 -> T1   : T1이 진행하고 종료하면 뒤의 대기도 풀릴 수 있음
T1 -> T2 -> T1   : 순환, 서로가 가진 잠금을 기다림
```

3절의 상품 10·20 예에서는 각 트랜잭션이 하나를 가진 채 다른 하나를 기다린다. 이 구조를 이해하려면 대기 중인 SQL뿐 아니라 **그 전에 무엇을 실행해 어떤 자원을 얻었는지**도 알아야 한다. 현재 보이는 마지막 SQL 두 줄만으로 원인을 단정하지 않는다.

여러 자원이 배타적으로 사용되고, 가진 것을 놓지 않은 채 추가 자원을 기다리며, 필요 자원을 임의로 빼앗을 수 없고, 기다림이 순환하면 진행이 막힐 수 있다. DBMS의 피해 트랜잭션 중단은 이 상태를 깨서 나머지가 진행하게 만드는 방법이다. 한쪽이 실패했다는 사실이 데이터베이스 전체가 복구 불가능하게 멈췄다는 뜻은 아니다.

## 9. 두 상품을 항상 같은 순서로 잠그기

이 페이지처럼 상품 ID가 바뀌지 않는 경우, 관련 상품을 잠그는 순서를 ID 오름차순으로 맞출 수 있다.

```sql
BEGIN;
SELECT id, stock
FROM products
WHERE id IN (10, 20)
ORDER BY id
FOR UPDATE;

UPDATE products SET stock = stock - 1 WHERE id = 10;
UPDATE products SET stock = stock - 1 WHERE id = 20;
COMMIT;
```

두 경로가 모두 10을 먼저 요청하면 두 번째는 첫 단계에서 기다릴 수 있지만, 20을 먼저 쥔 채 10을 기다리는 반대 순환은 줄일 수 있다. 여러 곳에서 같은 규칙을 적용해야 하며 한 함수만 정렬하고 다른 코드 경로는 반대로 잠그면 효과가 제한된다.

`IN (10, 20)`의 나열 순서만으로 잠금 획득 순서를 지정했다고 보지 않는다. 실제로 반환·잠금할 순서를 표현하고, 정렬 기준의 키가 변경되는 복잡한 경우에는 추가 검토가 필요하다. FK 검사·트리거·다른 테이블 접근처럼 겉으로 보이지 않는 관련 작업도 실제 대기 관계에 참여할 수 있다.

잠금 승격도 주의한다. 둘 다 공유 성격의 잠금을 얻은 뒤 각각 배타 변경을 요구하면 처음부터 반대 행을 잡지 않아도 순환이 생길 수 있다. 이후 갱신이 필요하다면 알맞은 모드를 처음부터 일관된 순서로 얻는 설계를 검토한다. 테이블 잠금의 유사 사례는 [LOCK 문서](https://www.postgresql.org/docs/18/sql-lock.html)에서 확인할 수 있다.

## 10. 타임아웃은 교착 상태의 증명이 아니다

트랜잭션이 단방향으로 오래 기다려도 정한 대기 시간은 초과할 수 있다. 반대로 교착 상태는 타임아웃이 끝나기 전에 DBMS의 탐지로 오류가 날 수 있다. 이 둘의 오류 원인을 구분한다.

```sql
SET lock_timeout = '3s';
```

이 설정은 해당 세션에서 잠금 획득을 기다리는 시간을 제한한다. 문장 전체의 실행 시간 제한인 `statement_timeout`과는 목적이 다르다. 실습에서 대기 시간을 제한할 수 있지만 너무 짧게 정하면 정상적인 경합도 실패할 수 있다.

교착 상태를 관찰하려는 실험에 짧은 lock_timeout을 걸면 순환을 만들기 전에 첫 명령이 시간 초과될 수도 있다. 테스트할 현상과 제한 시간을 맞춰야 한다. 설정의 의미는 [Client Connection Defaults](https://www.postgresql.org/docs/18/runtime-config-client.html)를 참고한다.

실험이 끝나면 설정과 트랜잭션을 정리한다.

```sql
ROLLBACK;
RESET lock_timeout;
```

타임아웃을 키우는 것은 근본 원인을 고치는 방법과 다르다. 사용자가 견딜 수 있는 지연·상대 작업의 진행 가능성·재시도 정책을 함께 고려한다.

## 11. 누가 누구를 막는지 관찰하기

A·B와 별도의 관찰용 연결을 사용하면 대기 중인 세션에서도 조회를 보낼 필요 없이 상태를 볼 수 있다. 먼저 각 세션의 PID를 기록한다.

```sql
SELECT pg_backend_pid();
```

관찰용 세션에서 다음을 실행한다.

```sql
SELECT pid, state, wait_event_type, wait_event,
       xact_start, query_start,
       pg_blocking_pids(pid) AS blocking_pids,
       query
FROM pg_stat_activity
WHERE datname = current_database()
  AND pid <> pg_backend_pid()
ORDER BY pid;
```

`blocking_pids`는 해당 세션의 잠금 획득을 막는 PID 목록이다. `xact_start`와 `query_start`를 나누면 오래 열린 트랜잭션 안에서 짧은 최근 SQL만 보이는 상황도 구분할 수 있다. `state`가 idle in transaction이면 지금 SQL을 실행하지 않아도 트랜잭션은 열려 있을 수 있다.

세션 상태와 대기 유형은 다른 정보다. active라고 CPU를 계속 쓰는 것은 아니며 잠금을 기다리면서 active로 보일 수도 있다. 반대로 wait_event_type이 Lock이 아니면 저장 장치나 클라이언트 응답 등 다른 대기일 수 있다.

권한에 따라 다른 세션의 SQL이나 일부 정보가 가려질 수 있다. 정확한 열의 의미는 [pg_stat_activity](https://www.postgresql.org/docs/18/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW), 함수의 의미는 [System Information Functions](https://www.postgresql.org/docs/18/functions-info.html)를 참고한다.

### 잠금 자체를 조회하기

```sql
SELECT pid, locktype, mode, granted,
       relation::regclass AS relation_name,
       transactionid
FROM pg_locks
WHERE pid IS NOT NULL
ORDER BY pid, granted, locktype;
```

`granted = false`이면 요청한 잠금을 기다리는 상태다. 하나의 행 잠금 때문에 생긴 대기가 관찰 뷰에는 transactionid 관련 잠금으로 나타날 수 있다. 따라서 행마다 항상 보기 좋은 상품 ID가 한 줄씩 나온다고 기대하지 않는다.

관찰 뷰는 읽는 순간의 상태다. 교착 상태가 탐지되어 한쪽이 중단되면 순환이 이미 풀렸을 수 있으므로 오류 로그와 실행 순서도 함께 남긴다. [pg_locks](https://www.postgresql.org/docs/18/view-pg-locks.html)를 참고한다.

## 12. 실패 후 재시도의 시작점 정하기

교착 상태의 피해 트랜잭션이 중단되면 변경을 취소하고 새 트랜잭션에서 다시 시작한다. 그동안 재고·가격·상태가 바뀌었을 수 있으므로 처음 조회한 조건부터 다시 확인한다.

```text
새 트랜잭션 시작
  -> 조건 조회와 검증
  -> 같은 순서로 잠금·변경
  -> COMMIT
실패 시
  -> DB 상태 정리
  -> 재시도 가능한 오류인지 판단
  -> 제한된 횟수와 간격으로 전체 작업 재실행
```

모든 오류를 무한 재시도하면 안 된다. 필수 값 누락이나 유일성 규칙 위반이 업무상 확정된 거절이라면 같은 입력으로 반복해도 해결되지 않을 수 있다. PostgreSQL의 `40P01`과 `40001` 같은 충돌 오류도 재시도 횟수·대기·외부 부수 효과를 함께 고려해야 한다. [Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html)을 참고한다.

마지막 SQL만 다시 보내는 것은 전체 재시도와 다르다. 예전에 읽은 재고를 변수에 보관하고 UPDATE만 다시 하면 이미 바뀐 조건으로 잘못 판단할 수 있다. 외부 결제 같은 DB 밖 작업은 이 재실행에 그대로 포함해 중복 호출하지 않도록 별도로 설계한다.

## 13. 잠금으로 먼저 보호할지, 변경 충돌을 확인할지

비관적 접근은 충돌할 수 있는 변경 전에 잠금을 얻고 보호 구간 안에서 결정하는 방식이다. 낙관적 접근은 읽은 뒤 변경할 때 그동안 상태가 바뀌었는지 검사하는 식으로 구현할 수 있다. 낙관적이라는 이름이 실제 DB 내부에서 잠금을 전혀 사용하지 않는다는 뜻은 아니다. 두 접근의 구분은 [Microsoft 동시성 제어 설명](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17)을 참고한다.

예를 들어 읽을 때 버전 0을 보았다면 다음처럼 기대한 버전과 현재 버전이 같은지 변경 조건에 넣을 수 있다. 다른 실험의 트랜잭션을 끝내고 버전과 재고를 초기화한 별도 시나리오에서 실행한다.

```sql
UPDATE products SET stock = 10, version = 0 WHERE id = 10;

UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 10 AND version = 0 AND stock >= 1
RETURNING id, stock, version;
```

같은 기대 버전 0으로 다시 변경하려 하면 조건에 맞는 행이 없어질 수 있다. 0행이라는 결과를 검사하고 현재 상태를 다시 읽어 충돌·품절·대상 없음 중 무엇인지 판단한다. 이 패턴 역시 성공 결과를 확인하는 업무 로직과 트랜잭션 경계가 필요하다.

어느 접근이 맞는지는 충돌 빈도와 보호해야 할 규칙에 달려 있다. 잠금을 오래 보유하는 비용과 실패 시 재실행하는 비용을 비교하며, 여러 행에 걸친 규칙을 행 하나의 버전 조건으로 모두 해결했다고 가정하지 않는다.

## 14. 흔한 오해와 운영·디버깅 관점

| 증상 또는 오해 | 확인할 질문 |
| --- | --- |
| 느린 UPDATE는 인덱스 부족이다 | 실제 실행 비용인가, 잠금 대기인가? |
| SELECT는 아무 잠금도 없다 | 일반 MVCC 읽기와 테이블·잠금 조회 모드를 구분했는가? |
| 모든 잠금은 SQL 문장이 끝나면 풀린다 | 명령이 속한 트랜잭션이 끝났는가? |
| NOWAIT는 잠금 없이 처리한다 | 잠금 획득 실패를 오류로 처리하는가? |
| SKIP LOCKED를 쓰면 같은 전체 결과를 얻는다 | 건너뛴 행이 있어도 되는 업무인가? |
| 교착 상태는 DB 전체가 고장 난 것이다 | 탐지 후 한 트랜잭션이 중단되어 순환이 풀렸는가? |
| ID만 정렬하면 모든 교착 상태가 사라진다 | 다른 자원·모드 전환·내부 제약 검사 경로는 없는가? |

장애 분석에서는 기다리는 요청의 SQL만 보지 않는다. 그 요청을 막는 세션의 트랜잭션 시작 시각, 이전 접근 순서와 종료 누락을 찾는다. 외부 API 응답을 기다리며 잠금을 보유하면 DB에서 실제로 실행한 SQL이 짧아도 다른 요청의 지연은 커질 수 있다.

잠금이 많다는 개수만으로 문제를 판단하지 않는다. 충돌하지 않는 보호는 정상적인 실행의 일부일 수 있다. 어떤 자원에 어떤 모드가 충돌하고 얼마나 오래 기다리는지가 중요하다. 인덱스는 접근 시간을 줄여 보호 구간을 짧게 할 수 있지만 업무 규칙과 잠금 순서를 대신하지 않는다.

## 15. 핵심 정리

<callout icon="✅" color="green_bg">
잠금은 자원·모드·유지 시간을 가진 동시성 제어다. 교착 상태는 순환 대기이며, 일관된 접근 순서와 짧은 경계로 줄이고 중단 시 전체 작업을 다시 판단한다.
</callout>

- 일반 읽기·잠금 조회·변경의 충돌은 제품과 모드에 따라 다르다.
- 행 잠금의 보호와 테이블 수준 모드, 범위의 보호를 구분한다.
- 단방향 대기와 순환 대기는 원인과 대응이 다르다.
- NOWAIT는 오류, SKIP LOCKED는 결과 집합의 생략이라는 선택을 만든다.
- 같은 자원을 같은 순서와 알맞은 모드로 획득하고 트랜잭션을 짧게 유지한다.
- 오류 코드와 세션·잠금·실행 순서를 연결해 원인을 확인한다.
- 중단된 트랜잭션은 조회와 업무 판단부터 제한적으로 재시도한다.

## 출처

- [PostgreSQL 18 — Explicit Locking](https://www.postgresql.org/docs/18/explicit-locking.html): 테이블·행 모드와 유지 시간, 교착 상태.
- [SELECT Locking Clause](https://www.postgresql.org/docs/18/sql-select.html#SQL-FOR-UPDATE-SHARE), [LOCK](https://www.postgresql.org/docs/18/sql-lock.html): 잠금 조회·NOWAIT·SKIP LOCKED, 모드 전환과 순서.
- [MySQL 8.4 — InnoDB Locking](https://dev.mysql.com/doc/refman/8.4/en/innodb-locking.html), [Deadlocks](https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks.html): S/X 모델과 엔진별 범위 보호, 대기 순환과 완화.
- [PostgreSQL 18 — Client Connection Defaults](https://www.postgresql.org/docs/18/runtime-config-client.html): lock_timeout·statement_timeout의 구분.
- [pg_stat_activity](https://www.postgresql.org/docs/18/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW), [pg_locks](https://www.postgresql.org/docs/18/view-pg-locks.html), [System Information Functions](https://www.postgresql.org/docs/18/functions-info.html): 세션·잠금·막는 PID의 관찰.
- [Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html): 교착 상태·직렬화 실패와 전체 재시도.
- [Microsoft — Transaction Locking and Row Versioning Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17): 비관적·낙관적 동시성 제어의 구분.

## 16. 이해도 확인 질문

1. A가 상품 10을 FOR UPDATE로 잠갔을 때 B의 일반 SELECT와 UPDATE는 PostgreSQL에서 어떻게 달라지는가? 테이블의 강한 잠금이 추가되면 같은 결론을 무조건 적용할 수 있는가?
2. T3가 T2를, T2가 T1을 기다리는 상황과 T1·T2가 서로 기다리는 상황을 그래프로 그리고 일반 대기·교착 상태·타임아웃을 구분하라.
3. 두 상품을 바꾸는 모든 경로에서 같은 순서로 잠그면 어떤 순환을 줄일 수 있는가? IN 목록의 순서만 믿거나 다른 내부 접근을 무시하면 왜 충분하지 않은가?
4. NOWAIT와 SKIP LOCKED는 각각 어떤 결과를 만들며 일반적인 전체 목록 조회에 무조건 적용하면 왜 문제가 되는가? 없는 행의 보호도 함께 설명하라.
5. 교착 상태 오류가 난 주문을 다시 시도하려 한다. 어디서부터 다시 실행해야 하며 어떤 세션·잠금 정보와 오류 코드를 기록하면 원인을 찾는 데 도움이 되는가?
