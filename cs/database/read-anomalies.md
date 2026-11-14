---
track: CS
notion: https://app.notion.com/p/416178c4afac81268fb7f319a0f11235
---

# Dirty Read / Non-repeatable Read / Phantom Read

<callout icon="📍" color="gray_bg">
**학습 범위**: Database · 6/7 · 동시 실행에서 읽은 값과 행 집합의 변화

미확정 값·재조회한 데이터·검색 조건의 집합을 구분하고 두 세션의 실행 순서로 검증한다. 표준 수준의 정의와 PostgreSQL 18의 일반 SELECT 동작을 나누어 설명한다.
</callout>

<table-of-contents />

## 1. 학습 목표

- Dirty Read, Non-repeatable Read, Phantom Read를 실행 순서에서 식별한다.
- 미확정 변경과 확정된 과거 버전을 읽는 일을 구분한다.
- 행 값·행의 존재·조건에 맞는 집합의 변화를 나누어 관찰한다.
- 문장 스냅샷과 트랜잭션 스냅샷의 시점을 설명한다.
- 두 세션 실험에서 연결·격리 수준·초기화·커밋 시점을 통제한다.

## 2. 이 주제를 왜 배우는가

한 세션에서 SELECT를 두 번 실행해 같은 값이 나왔다는 사실만으로 동시성에 안전한 것은 아니다. 두 조회 사이에 다른 트랜잭션이 변경을 확정하면 값이나 행 집합이 달라질 수 있다. 반대로 옛 값이 보인다고 곧바로 캐시 오류나 Dirty Read인 것도 아니다.

이 페이지에서는 시간 순서와 관찰 대상을 기록한다. **누가 어떤 값을 썼는가, 그 값은 커밋되었는가, 조회는 어느 시점의 어떤 집합을 보는가**를 분리하면 현상을 정확히 설명할 수 있다.

실습용 관리자 세션에서 아래 구조만 먼저 만든다. 3절의 초기 데이터 INSERT는 이 스키마에서 한 번 실행한다. A·B 세션 모두 같은 DB에 접속하고 각 세션에서 search_path를 동일하게 설정한다.

```sql
CREATE SCHEMA db_read_lab;
SET search_path TO db_read_lab;
CREATE TABLE users (
    id integer PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL
);
CREATE TABLE products (
    id integer PRIMARY KEY,
    name text NOT NULL,
    price integer NOT NULL,
    stock integer NOT NULL CHECK (stock >= 0)
);
CREATE TABLE orders (
    id integer PRIMARY KEY,
    user_id integer NOT NULL REFERENCES users(id),
    status text NOT NULL
);
```

각 세션에서 사용할 설정은 다음과 같다. 같은 터미널 창을 이름만 A와 B로 바꾼 것이 아니라 **서로 다른 DB 연결**이 필요하다.

```sql
SET search_path TO db_read_lab;
SELECT pg_backend_pid();
```

두 세션의 PID가 다른지 확인하고, 진행 중인 트랜잭션을 모두 끝낸 뒤 실험을 초기화한다. 실습 도구가 자동으로 BEGIN을 실행하거나 조회마다 연결을 바꾸는지도 확인한다.

## 3. 세 가지 읽기 이상과 기본 재현

### 두 트랜잭션의 순서를 분리해서 읽는다

다음 표의 `T1`과 `T2`는 같은 DB에 접속한 서로 다른 세션의 트랜잭션이다. 각 예는 독립적이며 다른 트랜잭션이나 자기 자신의 추가 변경은 없다고 가정한다. 중요한 것은 SQL 이름보다 **읽은 값의 확정 여부, 읽는 시점, 어떤 행 집합을 읽는가**다.

### Dirty Read: 아직 확정하지 않은 값을 읽는다

초기 상품 10의 재고가 10이라고 하자.

| 순서 | T1 | T2 |
| --- | --- | --- |
| 1 | `BEGIN`, 재고를 9로 변경 | |
| 2 | 아직 확정하지 않음 | 미확정 값을 읽을 수 있는 환경에서 재고 9를 읽음 |
| 3 | `ROLLBACK`, 재고는 다시 10 | |
| 4 | | 실제로 확정되지 않은 9를 근거로 판단한 상태 |

**Dirty Read** 는 다른 트랜잭션이 커밋하지 않은 데이터를 읽는 현상이다. 나중에 롤백될 값을 믿고 행동할 수 있어서 문제가 된다. PostgreSQL에서는 Read Uncommitted를 지정해도 이런 조회를 허용하지 않으므로, 위 표를 그대로 실행해 9가 보일 것이라고 기대하면 안 된다. 이 현상을 허용하는 수준의 예는 [MySQL Read Uncommitted 설명](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)에 있다.

### Non-repeatable Read: 같은 행을 다시 읽었더니 값이 달라진다

초기 상품 10의 재고는 10이고 T1은 Read Committed라고 하자.

| 순서 | T1 | T2 |
| --- | --- | --- |
| 1 | `BEGIN`, 상품 10의 재고 조회 → 10 | |
| 2 | | 재고를 9로 변경하고 `COMMIT` |
| 3 | 같은 상품의 재고 재조회 → 9 | |
| 4 | `COMMIT` | |

**Non-repeatable Read** 는 같은 트랜잭션에서 이미 읽은 데이터를 다시 읽었을 때 다른 트랜잭션이 확정한 변경 때문에 결과가 달라지는 현상이다. 위 예는 매번 확정된 값만 읽으므로 Dirty Read는 없다. 같은 행이 다른 트랜잭션에 의해 삭제되어 재조회되지 않는 경우도 함께 생각할 수 있다.

### Phantom Read: 같은 조건의 행 집합이 달라진다

초기 `pending` 주문이 2건이고 T1은 Read Committed라고 하자.

| 순서 | T1 | T2 |
| --- | --- | --- |
| 1 | `BEGIN`, `pending` 주문 집합 조회 → 2행 | |
| 2 | | 새 `pending` 주문을 추가하고 `COMMIT` |
| 3 | 같은 조건 재조회 → 3행 | |
| 4 | `COMMIT` | |

**Phantom Read** 는 같은 검색 조건을 반복했을 때 조건에 맞는 행 집합이 달라지는 현상이다. 다른 트랜잭션의 삽입뿐 아니라 삭제나 조건 열의 갱신으로도 집합이 달라질 수 있다. 예를 들어 기존 주문의 상태가 `paid`에서 `pending`으로 바뀌어도 해당 조회에는 새로운 행이 들어온다.

행 수가 같아도 구성원이 바뀔 수 있으므로 `COUNT(*)`가 같다는 사실만으로 현상이 없다고 판단하지 않는다. 한편 조인 조건을 잘못 작성해서 한 번의 조회 안에 중복 행이 나온 것은 동시 실행으로 발생하는 Phantom Read와 다른 문제다.

세 읽기 현상의 일반적인 구분은 [Microsoft Transaction Locking and Row Versioning Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17)의 동시성 현상 설명을 참고한다. 제품별로 이를 막는 구현은 다르다.

### 두 세션에서 직접 관찰하기

2절의 실습 스키마를 만든 뒤 아래 초기 데이터를 한 번 넣는다. 이후 예제들은 이 데이터를 기준으로 하며, 각 시나리오를 다시 실행할 때는 변경한 값을 초기 상태로 맞춘다.

```sql
INSERT INTO users (id, email, name)
VALUES (1, 'hana@example.com', '하나');

INSERT INTO products (id, name, price, stock) VALUES
    (10, '키보드', 50000, 10),
    (20, '마우스', 20000, 20);

INSERT INTO orders (id, user_id, status) VALUES
    (100, 1, 'pending'),
    (101, 1, 'pending');
```

먼저 **세션 A**에서 다음까지만 실행한다.

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;
SELECT stock FROM products WHERE id = 10;       -- 10
SELECT id FROM orders WHERE status = 'pending' ORDER BY id; -- 100, 101
```

이어서 **세션 B**에서 실행하여 변경을 확정한다.

```sql
BEGIN;
UPDATE products SET stock = 9 WHERE id = 10;
INSERT INTO orders (id, user_id, status) VALUES (102, 1, 'pending');
COMMIT;
```

다시 **세션 A**에서 조회한다.

```sql
SELECT stock FROM products WHERE id = 10;       -- 9
SELECT id FROM orders WHERE status = 'pending' ORDER BY id; -- 100, 101, 102
COMMIT;
```

첫 조회와 비교하면 재고 값의 변화와 주문 집합의 변화를 각각 관찰할 수 있다. 실험 후 추가 주문 102를 삭제하고 재고를 10으로 되돌린 다음, 세션 A를 `BEGIN ISOLATION LEVEL REPEATABLE READ`로 시작하여 같은 순서를 반복해 보자. PostgreSQL의 일반 조회에서는 A가 처음 읽은 재고와 주문 집합이 유지된다. 단, A가 스스로 변경한 내용까지 보이지 않는다는 뜻은 아니다.

### MVCC는 읽는 버전을 관리한다

**MVCC(Multiversion Concurrency Control)** 는 여러 데이터 버전과 가시성 규칙을 이용해 각 조회가 볼 수 있는 상태를 정하는 방법이다. 스냅샷은 매번 DB 전체를 파일로 복사한다는 뜻이 아니라 어떤 버전을 볼 수 있는지 결정하는 기준이다.

PostgreSQL에서 일반적인 Read Committed 조회는 문장 시작 시점의 스냅샷을 사용한다. Repeatable Read는 트랜잭션의 첫 실질적인 문장으로 정해진 스냅샷을 후속 조회에도 사용한다. `BEGIN`을 실행한 순간부터 무조건 스냅샷이 고정된다고 외우지 않는다. MVCC로 읽기와 쓰기의 충돌을 줄일 수 있지만 쓰기 충돌과 잠금 자체가 사라지지는 않는다. [PostgreSQL MVCC Introduction](https://www.postgresql.org/docs/18/mvcc-intro.html)을 참고한다.

<callout icon="🔭" color="gray_bg">
**[미리보기: Write Skew]**

같은 스냅샷을 읽는 두 트랜잭션이 서로 다른 행을 바꿔 여러 행에 걸친 업무 규칙을 깨뜨리는 경우도 있다. 따라서 세 읽기 이상을 막았다는 사실만으로 모든 동시 실행이 직렬 가능하다고 판단할 수 없다.
</callout>

## 4. 읽기 이상을 분류하는 네 가지 질문

1. 다른 트랜잭션의 변경인가, 자기 자신의 변경인가?
2. 읽은 변경은 그때 커밋되었는가?
3. 같은 행을 다시 읽은 것인가, 같은 검색 조건을 다시 실행한 것인가?
4. 두 번의 조회는 정말 같은 트랜잭션의 비교인가?

Dirty Read는 미확정 변경을 읽었다는 점이 핵심이다. 나중에 결국 커밋되었더라도 읽는 순간 미확정이었다면 그 위험이 있다. 반면 다른 트랜잭션이 확정한 변경을 두 번째 조회에서 읽었다면 Dirty Read 대신 반복 읽기의 문제를 검토한다.

Non-repeatable Read와 Phantom Read는 학습상 초점을 나누지만 모든 SQL 변경이 한 이름에만 배타적으로 속한다고 생각하지 않는다. 예를 들어 기존 행의 상태를 바꾸면 행의 값도 변하고 어떤 조건의 집합에 들어오거나 나갈 수도 있다. 설명할 때는 자신이 비교하는 관찰을 정확히 적는다.

세 용어는 결과가 “이상해 보인다”는 인상에 붙이는 이름이 아니다. 조인 조건 누락, 매번 다른 정렬, 시간에 따라 값이 바뀌는 함수, 애플리케이션 캐시 문제는 다른 원인일 수 있다. 먼저 같은 질의를 같은 범위에서 비교했는지 확인한다. 현상의 기본 구분은 [Microsoft의 동시성 가이드](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17)를 참고한다.

## 5. Dirty Read와 확정된 과거 값의 차이

### 더 최신으로 보인다고 더 안전한 값은 아니다

재고 10을 T1이 0으로 바꿨지만 아직 커밋하지 않았다고 하자. T2가 0을 읽으면 화면에는 최신 변경처럼 보일 수 있다. 하지만 T1이 롤백하면 그 0은 확정된 업무 상태가 아니었다. T2가 이를 근거로 품절 통보나 다른 처리를 했다면 판단의 기반이 사라진다.

반면 T2가 커밋된 기존 버전인 10을 읽는 것은 낡을 수 있어도 미확정 값을 읽는 현상과 다르다. 업무에 따라 최신 상태를 확인하는 추가 제어가 필요할 수 있지만, 두 문제의 원인과 보장은 구분해야 한다.

### PostgreSQL에서는 낮은 이름의 수준도 Dirty Read를 허용하지 않는다

초기화는 모든 세션이 트랜잭션 밖에 있는 상태에서 수행한다.

```sql
UPDATE products SET stock = 10 WHERE id = 10;
```

세션 A에서 시작하고 COMMIT하지 않는다.

```sql
BEGIN;
UPDATE products SET stock = 9 WHERE id = 10;
```

세션 B에서는 다음을 실행한다.

```sql
BEGIN ISOLATION LEVEL READ UNCOMMITTED;
SELECT stock FROM products WHERE id = 10; -- PostgreSQL에서는 10
COMMIT;
```

A를 ROLLBACK하면 원래 재고가 유지된다. A가 COMMIT하더라도 B가 그 전에 본 10은 당시의 확정된 버전이었다. 이 예로 Read Uncommitted라는 이름의 표준 최소 보장과 PostgreSQL이 실제로 제공하는 더 강한 동작을 구분할 수 있다.

다른 DBMS에 같은 결과를 무조건 기대하면 안 된다. 예를 들어 MySQL InnoDB는 Read Uncommitted에서 미확정 값을 읽을 수 있는 동작을 설명한다. 반면 Oracle도 Dirty Read를 허용하지 않는다고 명시한다. [MySQL Isolation Levels](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html), [Oracle Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/data-concurrency-and-consistency.html)를 참고한다.

## 6. Non-repeatable Read의 비교 대상은 무엇인가

상품 10을 먼저 읽고 다른 트랜잭션이 값을 바꾼 뒤 다시 읽으면 같은 PK를 지정해도 값이 달라질 수 있다. 행이 삭제되어 두 번째에 0행이 되는 상황도 반복해서 관찰한 데이터가 달라진 경우다.

반면 자기 트랜잭션이 `stock = stock - 1`을 수행한 뒤 재조회해서 값이 달라진 것을 다른 트랜잭션의 간섭으로 분류하지 않는다. 대부분의 일반적인 조회는 자기 트랜잭션의 앞선 쓰기를 볼 수 있다. “Repeatable Read니까 내가 UPDATE한 값도 이전대로 보여야 한다”는 해석은 잘못이다.

또한 두 SELECT가 각각 자동 커밋으로 실행되었다면 서로 다른 트랜잭션이다. 이때 값이 바뀌었다는 것은 자연스러운 일일 수 있으며, 같은 트랜잭션 내의 반복 읽기 보장을 시험했다고 말할 수 없다.

조회 결과에 현재 시각이나 임의 값처럼 실행마다 바뀌는 식을 넣는 것도 주의한다. 저장된 같은 데이터가 같은 스냅샷에서 유지된다는 보장을 모든 함수의 반환값이 영원히 같다는 뜻으로 넓히면 안 된다. 비교할 열과 대상 행을 명시한다.

## 7. Phantom Read는 숫자 하나보다 집합의 구성원이다

### 삽입만이 집합을 바꾸는 것은 아니다

`WHERE status = 'pending'`이라는 조건을 사용하면 새 pending 주문의 삽입뿐 아니라 다음 변경도 집합을 바꿀 수 있다.

- pending 주문을 삭제하면 구성원이 사라진다.
- paid 주문을 pending으로 바꾸면 기존 행이 새 구성원으로 들어온다.
- pending 주문을 paid로 바꾸면 그 행이 조건 밖으로 나간다.

따라서 “Phantom은 INSERT 때문에만 생긴다”로 외우면 검색 조건의 의미를 놓친다. 핵심은 같은 조건을 다시 평가했을 때 반환 집합이 다른 트랜잭션의 확정 변경으로 달라졌는지다.

### COUNT가 같아도 같은 집합은 아니다

첫 조회가 주문 `{100, 101}`을 보고 두 번째가 `{101, 102}`를 보면 개수는 둘 다 2다. 그러나 구성원이 다르다. 보고서에서 건수만 비교하면 이런 변화를 놓칠 수 있다.

실험할 때는 먼저 ID 목록을 확정된 순서로 출력하고 필요하면 개수도 함께 본다. 정렬 순서만 다른 경우와 실제 구성원이 다른 경우를 구분하기 위해 `ORDER BY id`를 사용한다. 다만 ORDER BY는 집합의 변화를 막는 동시성 제어가 아니라 표시를 비교하기 쉽게 만드는 도구다.

### 아직 없는 행을 읽기 잠금으로 모두 보호할 수 있는가

조건에 맞는 기존 행만 잠근다는 생각으로는 앞으로 그 조건에 들어올 새 행을 설명하기 어렵다. DBMS는 범위 잠금, 스냅샷, 직렬화 충돌 검사 등 서로 다른 방법을 사용할 수 있다. 이 때문에 격리 수준의 이름과 잠금 구현을 하나의 고정된 표로 동일시하지 않는다.

구체적인 잠금 조회와 “행이 없어서 잠글 대상도 없는” 상황은 다음 Lock / Deadlock 페이지에서 확인한다. 이번 페이지에서는 같은 조건이 나타내는 **논리적인 행 집합**과 이미 읽은 **물리적인 행들**을 나누어 이해한다.

## 8. 문장 스냅샷과 트랜잭션 스냅샷을 시간축으로 보기

스냅샷은 조회가 어떤 버전을 볼 수 있는지 정하는 규칙이다. PostgreSQL의 일반 SELECT에서는 다음처럼 비교할 수 있다. 이 예에는 T1의 자체 변경이 없다고 가정한다.

| 시점 | 다른 트랜잭션의 상태 | Read Committed의 T1 | Repeatable Read의 T1 |
| --- | --- | --- | --- |
| t0 | 기존 재고 10 확정 | BEGIN | BEGIN |
| t1 | 변화 없음 | 첫 SELECT → 10 | 첫 SELECT → 10, 스냅샷 기준 확립 |
| t2 | T2가 9로 변경하고 COMMIT | 아직 다음 SELECT 전 | 같은 트랜잭션 유지 |
| t3 | 새 재고 9 확정 | 다음 SELECT → 9 | 다음 일반 SELECT → 10 |

Read Committed는 문장마다 새 기준을 사용할 수 있으므로 하나의 트랜잭션 안에서도 두 번의 SELECT가 다른 커밋을 볼 수 있다. Repeatable Read는 트랜잭션에서 정해진 기준을 유지하므로 T2가 추가한 행도 같은 일반 조회에 새로 등장하지 않는다.

**BEGIN 자체와 첫 실제 조회의 시점을 구분한다.** A가 BEGIN만 실행한 뒤 B가 변경을 커밋하고, 그 다음에 A가 첫 조회를 하면 A는 그 변경을 포함하는 기준을 잡을 수 있다. “A가 먼저 BEGIN했으니 무조건 옛 값”이라는 예측은 정확하지 않다.

스냅샷은 모든 값을 별도 파일로 복사한다는 뜻이 아니다. 여러 버전과 가시성 규칙으로 구현할 수 있으며 구체적인 저장 방법은 제품마다 다르다. 문장 단위와 트랜잭션 단위 일관성의 차이는 [Oracle의 Read Consistency 설명](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/data-concurrency-and-consistency.html)에서도 구분되지만, 위의 시작 시점과 결과 표는 PostgreSQL 규칙을 기준으로 읽는다.

## 9. 표준 수준과 실제 구현을 같은 칸에 섞지 않기

표준의 이상 현상 표는 각 수준이 금지해야 할 것을 이해하는 출발점이다. 한 제품이 더 강한 보장을 제공할 수도 있고, 일반 조회와 잠금 조회·갱신의 구현이 다를 수도 있다.

| 설명할 내용 | 함께 기록할 조건 |
| --- | --- |
| 표준 수준의 금지 현상 | 표준의 최소 보장을 설명하는지 |
| 제품의 실제 일반 SELECT | DBMS·버전·격리 수준·자체 쓰기 여부 |
| 잠금 조회나 UPDATE | 조회·충돌·대기·조건 재검사 규칙 |
| 애플리케이션의 관찰 | 같은 연결·같은 트랜잭션인지, 캐시가 개입했는지 |

PostgreSQL Read Committed의 UPDATE는 같은 행의 다른 변경을 기다린 뒤 갱신된 행에 조건을 다시 검사할 수 있다. 이 동작을 일반 SELECT의 스냅샷 설명 하나로 모두 대체하면 안 된다. MySQL InnoDB 역시 consistent read와 locking read를 구분한다. [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html), [InnoDB Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html)를 참고한다.

같은 스냅샷에서 세 읽기 이상이 관찰되지 않는다는 사실도 모든 병행 결과가 직렬 실행과 일치함을 자동 증명하지는 않는다. 읽기 이상만으로 다양한 구현을 충분히 분류하기 어렵다는 한계는 [Berenson 등의 원논문](https://www.microsoft.com/en-us/research/publication/a-critique-of-ansi-sql-isolation-levels/)에서 다룬다. 논문의 모든 심화 현상을 이번 페이지에서 암기할 필요는 없지만 보장의 범위를 과장하지 않아야 한다.

## 10. 실험을 초기화하고 비교 조건을 고정하기

3절의 Read Committed 실험을 끝내면 재고 9와 주문 102가 남는다. 다음 실험을 그대로 이어 하면 첫 조회부터 다른 값이 나와 예측이 어긋날 수 있다. A·B의 COMMIT 또는 ROLLBACK을 끝낸 뒤 관리자 세션에서 초기화한다.

```sql
UPDATE products SET stock = 10 WHERE id = 10;
DELETE FROM orders WHERE id = 102;
SELECT stock FROM products WHERE id = 10;
SELECT id FROM orders WHERE status = 'pending' ORDER BY id;
```

첫 결과 10, 두 번째 집합 `{100, 101}`을 확인한 뒤 같은 순서를 Repeatable Read로 실행한다. 각 세션에서 다음 설정도 확인할 수 있다.

```sql
SHOW default_transaction_isolation;
BEGIN ISOLATION LEVEL REPEATABLE READ;
SHOW transaction_isolation;
SELECT stock FROM products WHERE id = 10;
-- 이 지점에서 다른 세션의 변경과 COMMIT을 진행한다.
SELECT stock FROM products WHERE id = 10;
COMMIT;
```

여기서 B의 변경이 확정된 시점과 A의 첫 실제 SELECT 시점을 기록한다. B가 UPDATE만 실행하고 COMMIT하지 않았거나, A가 의도치 않게 트랜잭션을 종료했다면 다른 실험을 한 셈이다.

대기하는 UPDATE를 보았다고 즉시 읽기 이상이라고 부르지도 않는다. 잠금 때문에 작업이 아직 완료되지 않았다면 그 시점에는 값의 가시성 비교 이전에 실행의 진행 상태를 확인해야 한다.

## 11. 업무 요구에 맞춰 읽기의 범위를 정하기

상품 목록을 새로 고칠 때마다 최신 확정 값을 보는 것이 자연스러운 화면과, 여러 질의로 하나의 보고서를 만드는 작업은 요구가 다를 수 있다. 전자는 조회 사이의 변화가 정상일 수 있지만, 후자는 앞의 합계와 뒤의 상세가 다른 시점이면 해석이 어려워질 수 있다.

트랜잭션 수준의 안정된 조회를 선택했다면 그 경계 안에서 필요한 읽기를 수행해야 한다. HTTP 페이지 1과 페이지 2를 각각 별도 요청·별도 트랜잭션으로 읽었다면 하나의 Repeatable Read가 두 요청 전체를 묶는 것은 아니다.

또한 조회를 안정적으로 만든다고 그 값에 근거한 쓰기가 자동으로 안전해지는 것은 아니다. 재고를 읽고 나중에 임의의 숫자로 덮어쓰는 문제는 갱신 충돌을 함께 검토해야 한다. 읽기의 일관성과 변경의 안전성을 나누어 설명한다.

높은 격리를 선택할 때는 충돌로 인한 중단과 전체 재시도를 처리한다. “매번 값이 같아 보인다”는 관찰 하나보다 어떤 규칙이 필요한지, 실패 시 어떻게 복구하는지가 더 중요한 설계 기준이다.

## 12. 흔한 오해와 디버깅 순서

### “옛 값을 읽었으니 Dirty Read다”

Dirty는 값이 오래되었다는 뜻이 아니라 다른 트랜잭션의 미확정 변경을 읽었다는 뜻이다. 스냅샷의 확정된 과거 버전은 다른 범주다.

### “COUNT가 같으면 Phantom은 없다”

같은 개수의 다른 집합일 수 있다. 무엇을 비교했는지, ID 목록과 조건이 같은지부터 확인한다.

### “Repeatable Read면 내 UPDATE도 안 보여야 한다”

다른 트랜잭션의 새 커밋을 일반 조회에 반영하지 않는 규칙과 자기 변경의 가시성은 구분한다. 쓰기 충돌 때는 단순한 과거 버전 읽기 대신 오류가 날 수도 있다.

### “BEGIN을 먼저 했으니 언제나 먼저의 스냅샷이다”

제품의 기준 시점과 첫 실제 문장을 확인한다. 자동 커밋이나 새 연결이 개입하면 애초에 같은 트랜잭션을 비교한 것이 아닐 수 있다.

문제를 재현할 때는 다음 순서를 사용한다.

1. DBMS·버전과 두 세션의 PID를 기록한다.
2. 초기 데이터와 각 세션의 기본·현재 격리 수준을 확인한다.
3. SQL의 실행 완료와 COMMIT·ROLLBACK을 별도 이벤트로 적는다.
4. 같은 행의 값인지, 같은 조건의 집합인지 명시한다.
5. 자기 변경·애플리케이션 캐시·정렬 변화가 개입했는지 제외한다.
6. 모든 트랜잭션을 끝내고 초기화한 뒤 한 조건만 바꾸어 반복한다.

## 13. 핵심 정리

<callout icon="✅" color="green_bg">
읽기 이상은 관찰 대상과 시간 순서로 구분한다. 확정 여부, 동일 행과 검색 집합, 문장과 트랜잭션의 경계가 각각 다른 축이다.
</callout>

- Dirty Read는 다른 트랜잭션의 미확정 값이다.
- Non-repeatable Read는 다시 읽은 데이터가 다른 트랜잭션의 확정 변경으로 달라지는 현상이다.
- Phantom Read는 같은 조건의 행 집합이 달라지는 현상이며 개수만으로 충분히 비교할 수 없다.
- PostgreSQL 일반 조회의 Read Committed와 Repeatable Read는 스냅샷 유지 범위가 다르다.
- 표준 수준의 이름과 제품의 구현, 일반 조회와 잠금·갱신을 구분한다.
- 읽기가 안정적이어도 모든 쓰기 규칙이 자동으로 직렬 가능해지는 것은 아니다.

## 출처

- [Microsoft — Transaction Locking and Row Versioning Guide](https://learn.microsoft.com/en-us/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide?view=sql-server-ver17): 세 읽기 이상과 동시성 현상의 정의.
- [PostgreSQL 18 — Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html), [MVCC Introduction](https://www.postgresql.org/docs/18/mvcc-intro.html), [SET TRANSACTION](https://www.postgresql.org/docs/18/sql-set-transaction.html): 스냅샷 기준, 격리 수준과 읽기·쓰기의 동작.
- [MySQL 8.4 — Isolation Levels](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html), [Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.4/en/innodb-consistent-read.html): 다른 제품의 미확정 읽기와 consistent read 구분.
- [Oracle 19c — Data Concurrency and Consistency](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/data-concurrency-and-consistency.html): 미확정 읽기 금지, 문장·트랜잭션 수준 일관성의 개념.
- [Berenson 외 — A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/publication/a-critique-of-ansi-sql-isolation-levels/): 세 현상만으로 격리 구현 전체를 설명할 때의 한계.

## 14. 이해도 확인 질문

1. T1이 재고를 10에서 9로 바꾸고 나중에 ROLLBACK한다. T2가 변경 도중 9를 읽은 상황과 10을 읽은 상황은 무엇이 다르며 PostgreSQL에서는 어떤 값을 기대하는가?
2. 같은 트랜잭션에서 같은 PK를 두 번 조회했을 때 값 변경·행 삭제·자기 UPDATE가 각각 어떤 관찰을 만드는가? 모두 같은 읽기 이상으로 볼 수 있는가?
3. pending 주문 집합이 `{100, 101}`에서 `{101, 102}`로 바뀌었다. 건수만 비교하면 무엇을 놓치며 어떤 다른 변경도 이 집합을 바꿀 수 있는가?
4. A가 BEGIN한 뒤 B가 변경을 COMMIT하고, 그 다음 A가 첫 SELECT를 실행한다. A가 BEGIN 직후부터 과거 값을 본다고 단정할 수 없는 이유와 스냅샷 기준을 설명하라.
5. 같은 SQL을 두 번 실행했는데 격리 수준에 대한 예상과 결과가 다르다. 연결·트랜잭션·COMMIT·초기화·조회 종류 중 어떤 조건을 어떤 순서로 확인할 것인가?
