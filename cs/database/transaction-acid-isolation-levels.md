---
track: CS
notion: https://app.notion.com/p/416178c4afac81b6af68e114f3349bba
---

# Transaction / ACID / Isolation Level

<callout icon="📍" color="gray_bg">
**학습 범위**: Database · 5/7 · 트랜잭션 경계, ACID와 격리 수준

원자성과 격리성을 구분하고 성공·실패·커밋의 상태를 SQL로 따라간다. 읽기 이상 세 종류의 상세 재현은 다음 페이지에서, 잠금과 교착 상태는 마지막 페이지에서 다룬다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 함께 확정해야 하는 변경의 경계를 업무 규칙에서 도출한다.
- BEGIN·COMMIT·ROLLBACK·SAVEPOINT의 상태와 효과를 구분한다.
- ACID의 네 성질이 각각 해결하는 문제와 보장의 전제를 설명한다.
- 트랜잭션을 사용해도 갱신 분실 같은 간섭이 남는 이유를 설명한다.
- 표준 격리 수준과 PostgreSQL 구현을 구분하고 실패·재시도 범위를 판단한다.

## 2. 이 주제를 왜 배우는가

한 주문이 여러 SQL로 저장된다면 첫 문장의 성공이 전체 주문의 성공은 아니다. 재고만 줄고 주문 저장이 실패하면 데이터가 업무의 의미와 어긋난다. 반대로 각각의 주문을 잘 묶었어도 두 요청이 같은 재고를 보고 동시에 판단하면 다른 종류의 문제가 남는다.

트랜잭션은 이 둘을 하나의 마법 같은 보장으로 합치는 단어가 아니다. 무엇을 함께 확정하고 취소할지, 다른 실행과 어떤 간섭을 허용할지, 성공이라고 알린 변경을 어떻게 보존할지를 나누어 이해해야 한다.

3절의 재고 예제와 뒤의 계좌 예제는 독립적인 학습용 데이터다. 먼저 다음 스키마를 만든다. 같은 예제를 반복할 때는 해당 재고·잔액을 초기 상태로 맞춘다.

```sql
CREATE SCHEMA db_transaction_lab;
SET search_path TO db_transaction_lab;
CREATE TABLE products (
    id integer PRIMARY KEY,
    stock integer NOT NULL CHECK (stock >= 0)
);
INSERT INTO products VALUES (10, 10);
```

## 3. 트랜잭션과 ACID, 격리 수준의 기본

### 트랜잭션은 함께 성공하거나 취소할 작업의 경계다

**트랜잭션(transaction)** 은 하나의 논리적인 작업으로 취급할 데이터베이스 연산의 묶음이다. 주문 저장과 재고 감소가 따로 확정되면 주문만 남거나 재고만 줄 수 있다. 트랜잭션으로 묶으면 이 DB 변경들을 함께 확정하거나 취소할 수 있다.

| 명령 | 역할 |
| --- | --- |
| `BEGIN` | 명시적인 트랜잭션 시작 |
| `COMMIT` | 변경을 확정 |
| `ROLLBACK` | 아직 확정하지 않은 변경을 취소 |

PostgreSQL에서는 명시적으로 묶지 않은 문장도 트랜잭션 안에서 실행된다. 일반적인 자동 커밋 설정에서는 각 문장이 성공하면 개별적으로 확정되므로 여러 SQL을 연달아 호출했다고 하나의 작업으로 묶이지 않는다. 여러 문장을 한 트랜잭션으로 실행할 때는 같은 DB 세션을 사용해야 한다. [PostgreSQL Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html)를 참고한다.

### 주문 트랜잭션의 실행 흐름

```text
BEGIN
  -> 재고가 충분한 상품 행의 재고를 조건부로 감소
  -> 감소한 행이 1개인지 확인
     -> 0개: 품절 또는 대상 없음으로 판단하고 ROLLBACK
     -> 1개: 주문과 주문 항목 저장
  -> 모든 단계 성공: COMMIT
  -> SQL 오류나 업무 조건 실패: ROLLBACK
```

다음 SQL은 재고 확인과 감소를 한 문장으로 표현한다. `RETURNING`은 실제로 변경한 행의 값을 반환한다.

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 10 AND stock >= 1
RETURNING id, stock;
```

반환 행이 0개인 것은 SQL 문법 오류가 아니다. `COMMIT`을 무조건 실행하지 말고 애플리케이션이 그 결과를 확인해야 한다. PostgreSQL Read Committed에서 같은 행의 갱신과 충돌하면 기다린 뒤 최신 행에 대해 조건을 다시 검사하므로 이 패턴은 단순한 “조회 후 값 덮어쓰기”와 다르게 동작한다. 조건 재검사 규칙은 [Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html#XACT-READ-COMMITTED)을 참고한다.

### ACID는 서로 다른 보장을 묶은 이름이다

| 성질 | 의미 | 주문 예시 |
| --- | --- | --- |
| Atomicity, 원자성 | 트랜잭션의 변경을 전부 반영하거나 전부 취소 | 주문 항목 저장 실패 시 앞선 재고 감소도 취소 |
| Consistency, 일관성 | 올바른 트랜잭션이 정해진 무결성 규칙을 지키며 상태를 전환 | 재고 음수 금지, 유효한 회원 참조 |
| Isolation, 격리성 | 동시 실행 간 간섭을 선택한 규칙에 따라 제한 | 어떤 시점의 재고를 읽고 충돌 시 어떻게 처리할지 결정 |
| Durability, 지속성 | 성공적으로 확정된 결과를 장애 뒤에도 보존 | DB 프로세스 재시작 후에도 확정 주문 유지 |

일관성은 DBMS가 모든 업무 규칙을 추론한다는 뜻이 아니다. `CHECK (stock >= 0)`은 음수를 막지만, 잘못 작성한 주문 코드가 재고를 줄이지 않는 문제까지 찾아 주지는 않는다. 제약 조건, 트랜잭션 로직, 동시성 제어가 함께 규칙을 지켜야 한다. ACID와 구현 설정의 관계는 [MySQL InnoDB and the ACID Model](https://dev.mysql.com/doc/refman/8.4/en/mysql-acid.html)을 참고한다.

지속성을 위해 매 커밋마다 모든 테이블 페이지를 즉시 기록해야 하는 것은 아니다. PostgreSQL은 변경 내용을 먼저 영구 저장소의 **WAL(Write-Ahead Log)** 에 기록하고, 장애가 나면 로그로 필요한 변경을 다시 적용할 수 있다. 지속성의 실제 범위는 동기화 설정과 저장 장치에도 영향을 받으며, 지속성이 백업이나 모든 장치의 영구 손실에 대한 보장까지 대신하지는 않는다. [PostgreSQL WAL](https://www.postgresql.org/docs/18/wal-intro.html)을 참고한다.

### 원자성과 격리성은 같은 문제가 아니다

두 요청이 각각 트랜잭션을 사용해도 다음처럼 실행될 수 있다.

```text
초기 stock = 10
T1: stock을 읽음 -> 10
T2: stock을 읽음 -> 10
T1: 읽은 값 - 1인 9를 저장하고 COMMIT
T2: 읽은 값 - 1인 9를 저장하고 COMMIT
최종 stock = 9, 주문은 2건
```

각 트랜잭션은 자기 작업을 전부 확정했지만 두 감소 중 하나가 사라졌다. 이것이 **갱신 분실(lost update)** 의 한 형태다. 트랜잭션을 사용한다는 사실에 더해 갱신식, 잠금 또는 충돌 감지 방식을 검토해야 한다.

### Isolation Level: 어떤 간섭까지 허용할 것인가

**격리 수준(isolation level)** 은 동시에 실행되는 트랜잭션 사이의 관찰과 간섭을 제한하는 규칙이다. 다음 표는 SQL 표준이 금지하는 읽기 이상을 중심으로 정리한 최소 보장이다. “허용”은 반드시 발생한다는 뜻이 아니라 해당 수준에서 금지하도록 요구하지 않는다는 뜻이다.

| 격리 수준 | Dirty Read | Non-repeatable Read | Phantom Read |
| --- | --- | --- | --- |
| Read Uncommitted | 허용 | 허용 | 허용 |
| Read Committed | 금지 | 허용 | 허용 |
| Repeatable Read | 금지 | 금지 | 허용 |
| Serializable | 금지 | 금지 | 금지 |

세 읽기 현상의 자세한 실행 순서는 다음 학습 페이지에서 살펴본다. Serializable은 이 세 현상뿐 아니라 **성공한 트랜잭션들의 결과가 어떤 직렬 실행 순서와도 맞지 않는 문제**까지 금지한다. 직렬 실행은 한 트랜잭션을 끝낸 뒤 다음 트랜잭션을 실행하는 것이고, 직렬 가능성은 동시에 실행했어도 그중 어떤 순서로 하나씩 실행한 것과 같은 결과를 내는 성질이다.

**PostgreSQL의 실제 동작**에서는 기본값이 Read Committed이고, Read Uncommitted를 요청해도 Read Committed처럼 동작한다. Repeatable Read의 일반 조회는 같은 스냅샷을 사용하여 Phantom Read도 막지만, 이것만으로 Serializable이 되지는 않는다. 직렬화 실패가 발생할 수도 있으므로 트랜잭션 전체를 재시도할 준비가 필요하다. 표준과 이 구현의 차이는 [PostgreSQL Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html)에 명시되어 있다.

격리를 강화하면 대기나 충돌 검사·재시도 비용이 늘 수 있다. “모든 트랜잭션이 물리적으로 한 줄로 실행된다”거나 “모든 읽기가 잠금을 건다”고 일반화하지 않는다. 예를 들어 MySQL InnoDB의 기본 격리 수준은 PostgreSQL과 다르며, 같은 수준 안에서도 일반 조회와 잠금 조회의 규칙을 나누어 설명한다. [MySQL Transaction Isolation Levels](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html)를 참고한다.

## 4. 트랜잭션의 경계를 업무의 불변식에서 찾기

**불변식(invariant)** 은 유효한 상태에서 지켜야 하는 규칙이다. 두 계좌 사이의 단순 이체라면 다른 입출금이 없는 예에서 전체 잔액 합이 보존되어야 한다. 주문이라면 주문이 확정된 수량과 재고 변화가 일치해야 할 수 있다.

이 규칙을 지키기 위해 함께 완료해야 하는 DB 변경들을 같은 트랜잭션으로 묶는다. 첫 작업만 확정되거나 두 번째 작업만 확정된 상태를 최종 결과로 허용하면 안 된다면 두 작업 사이에 경계를 끊으면 안 된다.

반대로 사용자 입력을 기다리는 모든 시간이나 장시간 보고서 생성 전체를 무조건 한 트랜잭션으로 묶는 것도 좋은 기본값은 아니다. 경계가 길면 잠금과 오래된 버전의 유지, 연결 점유 같은 비용이 커질 수 있다. **같이 확정해야 하는 최소한의 논리적 작업**이 무엇인지 설명하는 것부터 시작한다.

다음은 이번 계좌 예제의 규칙이다. 통화는 하나이고 수수료·환율·외부 은행은 다루지 않는다.

```text
각 계좌의 잔액은 0 이상이다.
존재하는 계좌 1에서 200을 빼면 계좌 2에 200을 더한다.
두 변경을 모두 성공시킬 수 없으면 둘 다 취소한다.
성공 전후의 잔액 합은 같다.
```

무결성 제약은 일부 규칙을 표현하고 트랜잭션 로직은 그 규칙을 지키는 변경 순서를 구현한다. CHECK로 잔액 음수를 막는 것만으로 이체 대상에게 같은 금액을 더했다는 사실까지 보장되지는 않는다.

## 5. SQL을 실행하며 확정과 취소를 구분하기

### 준비와 성공 예제

```sql
CREATE TABLE accounts (
    id integer PRIMARY KEY,
    balance integer NOT NULL CHECK (balance >= 0)
);
INSERT INTO accounts VALUES (1, 1000), (2, 500);

BEGIN;
UPDATE accounts SET balance = balance - 200 WHERE id = 1;
UPDATE accounts SET balance = balance + 200 WHERE id = 2;
COMMIT;

SELECT id, balance FROM accounts ORDER BY id;
```

실습의 두 계좌는 이미 존재한다. 결과는 계좌 1의 800과 계좌 2의 700이다. 총합은 1500으로 유지된다. COMMIT 이전에 자기 세션에서 갱신된 값을 볼 수 있다는 사실은 다른 세션에도 모두 확정되었다는 뜻이 아니다.

### 일부 문장을 실행한 뒤 취소

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SELECT balance FROM accounts WHERE id = 1;
ROLLBACK;
SELECT balance FROM accounts WHERE id = 1;
```

위 성공 예제에 이어 실행하면 트랜잭션 안에서는 700을 보고, ROLLBACK 뒤에는 다시 800을 본다. 변경 SQL이 정상 실행되었다는 사실과 그 변경이 확정되었다는 사실을 구분할 수 있다. PostgreSQL의 기본 자동 커밋과 명시적 블록은 [BEGIN](https://www.postgresql.org/docs/18/sql-begin.html)과 [Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html)를 참고한다.

### 오류가 없다고 업무 성공인 것은 아니다

`WHERE id = 999`가 일치하지 않는 UPDATE는 정상 종료하며 영향받은 행이 0개일 수 있다. 이체 대상 계좌가 없는데 출금만 성공했다면 SQL 문법이 모두 맞아도 업무 규칙은 깨진다.

따라서 변경 행 수나 RETURNING 결과를 확인하고, 각 단계의 전제가 실패하면 전체를 취소해야 한다. 트랜잭션은 개발자가 잘못된 성공 조건을 작성한 문제까지 추론하지 않는다. 이 구분은 주문의 품절 검사에서도 동일하다.

## 6. 오류가 난 트랜잭션의 상태를 정리하기

PostgreSQL의 명시적 트랜잭션에서 SQL 오류가 발생하면 실패 상태가 되어 이후 일반 명령을 계속 실행할 수 없는 경우가 있다. 이때 마지막 문장만 고쳐 재실행하는 것으로 원래의 작업이 자동 복구되는 것은 아니다. ROLLBACK하거나 적절히 마련한 저장점으로 돌아가 상태를 복구해야 한다.

다음은 오류를 관찰하는 실습이므로 문장을 순서대로 따로 실행한다.

```sql
BEGIN;
UPDATE accounts SET balance = -1 WHERE id = 1; -- CHECK 위반 예상
SELECT * FROM accounts; -- 실패 상태라 실행이 거부됨
ROLLBACK;
```

이것을 “마지막 UPDATE만 실패하고 나머지 트랜잭션은 정상”이라고 해석하면 안 된다. 드라이버나 프레임워크가 대신 ROLLBACK하는지 여부와 DBMS의 상태도 나누어 본다. 실패한 세션을 정리하지 않고 연결 풀에 돌려주면 다음 작업이 같은 실패 상태를 이어받는 문제가 생길 수 있다.

```text
대기 중인 세션
  -> BEGIN: 작업 중
      -> COMMIT 성공: 변경 확정, 다음 작업 가능
      -> ROLLBACK: 변경 취소, 다음 작업 가능
      -> SQL 오류: 실패 상태
           -> ROLLBACK 또는 사용 가능한 저장점으로 복구
```

같은 세션에서의 상태와 오류 후 복구는 클라이언트 API에도 영향을 준다. 구현 언어가 달라도 트랜잭션은 해당 연결의 문맥에서 진행된다는 원칙을 먼저 이해한다.

## 7. SAVEPOINT는 외부에 확정하는 작은 COMMIT이 아니다

저장점은 트랜잭션 내부의 위치를 기억하고 그 뒤의 일부 작업을 취소할 수 있게 한다. 다음 예는 계좌 잔액을 원래 800·700에서 시작한다고 가정한다.

```sql
BEGIN;
UPDATE accounts SET balance = balance + 10 WHERE id = 1;
SAVEPOINT before_bonus;
UPDATE accounts SET balance = balance + 20 WHERE id = 2;
ROLLBACK TO SAVEPOINT before_bonus;
COMMIT;
```

최종적으로 계좌 1의 +10만 남고 계좌 2의 +20은 취소된다. 이 예는 저장점의 효과만 보여 주며 두 계좌 합 보존이 필요한 이체 규칙을 구현한 것은 아니다.

저장점을 해제하거나 저장점 이전 작업을 유지해도 전체 트랜잭션을 COMMIT하기 전에는 그 부분을 독립적으로 외부 확정한 것이 아니다. 바깥 트랜잭션이 ROLLBACK되면 유지했던 변경도 취소된다. BEGIN을 두 번 적는 것 역시 독립적인 중첩 트랜잭션 두 개를 만드는 방법이 아니다. [SAVEPOINT](https://www.postgresql.org/docs/18/sql-savepoint.html), [ROLLBACK TO SAVEPOINT](https://www.postgresql.org/docs/18/sql-rollback-to.html)를 참고한다.

저장점은 업무상 일부 실패를 복구해도 되는 경우에 사용한다. 이체에서 입금 실패를 저장점으로 무시하고 출금만 확정하면 원자성의 도구를 사용했어도 업무의 경계를 잘못 정한 것이다.

## 8. ACID를 실패 상황에 각각 대입하기

### Atomicity: 무엇을 전부 취소하는가

원자성은 트랜잭션에 속하는 변경의 효과를 함께 취소·확정하는 성질이다. SQL 한 문장만 실행했다는 뜻도, 컴퓨터가 실제로 모든 저장을 한 순간에 동시에 수행한다는 뜻도 아니다. 내부적으로 여러 단계가 있어도 완료되지 않은 부분 결과를 최종 확정 상태로 남기지 않는 것이 핵심이다.

원자성을 프로그램의 모든 외부 활동으로 확장하지 않는다. 이미 보낸 이메일이나 외부 결제 API의 효과는 해당 DB의 ROLLBACK으로 자동 취소되지 않는다. DB 변경의 경계와 다른 시스템의 작업 경계를 따로 정해야 한다.

### Consistency: 선언한 규칙과 올바른 변경 로직

일관성은 유효한 상태에서 출발한 올바른 트랜잭션이 정해진 무결성 규칙을 지키는 상태로 끝나야 한다는 관점이다. 조건을 선언하지 않거나 업무 로직을 잘못 작성하면 제품 이름이 ACID를 지원한다는 이유만으로 원하는 규칙이 생기지 않는다.

정규화는 사실의 중복 구조를 다루고, 제약은 허용 상태를 제한하며, 트랜잭션은 여러 변경을 묶는다. 이들이 서로를 완전히 대체하지 않는다. 또한 ACID의 일관성을 분산 시스템의 모든 복제본이 같은 순간 같은 값을 보인다는 의미와 혼용하지 않는다.

### Isolation: 중간 상태를 숨기는 것 이상의 선택

다른 트랜잭션의 미확정 변경을 읽지 않는 것과 같은 트랜잭션에서 반복 조회가 안정적인 것은 다른 보장이다. 여러 행을 읽고 서로 다른 행을 바꾸는 실행이 직렬 실행과 같은 결과를 내는지도 별도로 생각해야 한다.

따라서 “격리되어 있다”는 말에는 격리 수준과 조회 종류를 덧붙인다. 일반 SELECT, 잠금 조회, UPDATE가 같은 스냅샷·대기 규칙을 사용한다고 단정하지 않는다. 구체적인 관찰 차이는 다음 읽기 이상 페이지에서 실험한다.

### Durability: 커밋의 보존 전제를 안다

성공했다고 응답한 커밋의 보존은 로그 기록·동기화 설정·저장 장치와 연결된다. PostgreSQL의 비동기 커밋 설정은 최근 커밋이 장애 때 사라질 수 있는 구간을 허용한다. 이것은 같은 SQL 문법을 사용하더라도 설정에 따라 지속성의 범위를 점검해야 하는 사례다. [Asynchronous Commit](https://www.postgresql.org/docs/18/wal-async-commit.html)을 참고한다.

지속성은 사람의 잘못된 DELETE까지 되돌리는 기능이 아니다. 잘못된 삭제를 성공적으로 COMMIT했다면 그 삭제도 보존될 수 있다. 백업과 복구 지점, 권한·변경 검토는 다른 요구다. 성능을 위해 동기화를 완화하는 선택을 단순히 “속도 옵션”으로만 설명하지 않는다.

## 9. 커밋과 저장 장치 사이에서 WAL의 역할

DBMS가 변경한 데이터 페이지를 전부 매번 강제 기록하면 작은 트랜잭션도 많은 임의 쓰기를 요구할 수 있다. WAL 방식은 데이터 페이지보다 먼저 복구에 필요한 로그를 안전하게 기록하는 순서를 사용한다.

```text
데이터 변경을 수행하며 필요한 로그 생성
  -> 커밋에 필요한 로그를 영구 저장소에 기록하는 조건 충족
  -> 클라이언트에 커밋 성공 응답
  -> 데이터 페이지는 정해진 정책에 따라 기록
  -> 장애가 나면 기록된 로그로 필요한 변경을 재적용
```

이는 개념 흐름이며 실제 버퍼·체크포인트·복제 처리 전체를 나열한 것은 아니다. 중요한 점은 **로그가 먼저 안전하게 기록되면 테이블 페이지를 모두 즉시 기록하지 않아도 복구 경로를 마련할 수 있다**는 것이다. [Write-Ahead Logging](https://www.postgresql.org/docs/18/wal-intro.html)을 참고한다.

원자성과 지속성을 같은 단어로 외우면 차이를 놓친다. 중간 실패에서 일부 변경만 확정하지 않는 문제와, 성공적으로 확정한 변경을 이후 장애에서도 유지하는 문제는 구분된다.

## 10. 격리 수준을 직렬 실행과 비교하기

동시에 실행되지 않는다면 T1을 끝낸 뒤 T2를 실행하거나, T2를 끝낸 뒤 T1을 실행할 수 있다. **Serializable**의 의미는 성공한 결과가 이런 직렬 순서 중 하나와 일치하도록 보장하는 것이다. CPU에서 SQL을 무조건 하나씩만 실행한다는 뜻은 아니다.

예를 들어 서로 관련 없는 두 상품만 수정하는 트랜잭션은 병행해도 같은 직렬 결과를 만들 수 있다. 반대로 두 작업이 같은 조건을 읽고 결정을 내리면 행을 직접 덮어쓰지 않아도 서로 간섭할 수 있다. 필요한 격리는 보호하려는 업무 규칙과 읽기·쓰기 범위에 따라 판단한다.

```sql
SHOW default_transaction_isolation;
BEGIN ISOLATION LEVEL REPEATABLE READ;
SHOW transaction_isolation;
ROLLBACK;
```

첫 값은 세션의 기본 설정이고 두 번째는 현재 트랜잭션의 설정이다. 기본값과 명시적 설정을 구분해 기록한다. PostgreSQL에서는 첫 실질적인 조회나 변경 이후에 격리 수준을 임의로 바꾸는 방식으로 이미 수행한 판단을 고칠 수 없다. 필요한 수준을 시작할 때 정한다. [SET TRANSACTION](https://www.postgresql.org/docs/18/sql-set-transaction.html)을 참고한다.

## 11. 연결 풀에서도 같은 세션의 경계를 지킨다

애플리케이션이 연결 풀에서 매 SQL마다 서로 다른 연결을 받는다면, 첫 연결의 BEGIN이 두 번째 연결의 UPDATE까지 묶어 주지 않는다. 하나의 논리적 트랜잭션 동안 같은 연결을 확보하고 그 연결로 모든 명령을 수행해야 한다.

```text
연결 하나 확보
  -> 같은 연결에서 BEGIN
  -> 조회와 변경, 반환 행 수 확인
  -> 같은 연결에서 COMMIT 또는 ROLLBACK
  -> 정리된 연결을 반환
```

업무 오류와 DB 오류가 나도 종료 경로를 빠뜨리지 않는다. 사용자 요청이 끝났는데 DB 세션이 여전히 열린 트랜잭션 안에 있으면 잠금과 자원을 계속 보유할 수 있다. HTTP 요청이 완료되었다는 이벤트만으로 DB 트랜잭션의 종료를 추정하지 않는다.

연결 관리의 실제 API는 라이브러리에 따라 다르다. 이번 페이지에서는 특정 프레임워크 코드를 외우기보다 **BEGIN 이후 어떤 연결이 어떤 상태인지 추적하는 것**을 기준으로 삼는다. 같은 클라이언트를 사용해야 하는 실제 드라이버의 예는 [node-postgres의 Transactions](https://node-postgres.com/features/transactions)에서 확인할 수 있다.

## 12. 실패 응답과 결과 불명의 상황을 구분하기

제약 위반처럼 DB가 명확하게 오류를 응답한 경우와, COMMIT 직후 네트워크가 끊겨 응답을 못 받은 경우는 다르다. 후자에는 서버에서 이미 커밋했지만 클라이언트만 모르는 가능성이 있다.

따라서 네트워크 오류를 받았다는 이유만으로 업무를 아무 조건 없이 재실행하면 중복 주문이 생길 수 있다. 이미 확정한 작업인지 확인할 식별 기준과 중복 처리 정책이 필요하다. 이는 트랜잭션의 원자성이 없다는 뜻이 아니라 DB의 확정 상태와 클라이언트의 관찰이 별개라는 뜻이다.

반면 직렬화 실패처럼 **해당 트랜잭션이 성공적으로 확정되지 않았음을 알리는 오류**에는 처음의 읽기와 판단부터 다시 실행해야 한다. 실패한 마지막 UPDATE만 재전송하면 이미 읽은 낡은 조건을 그대로 사용할 수 있다. 오류별 재시도 범위는 [Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html)을 참고한다.

## 13. 흔한 오해와 운영 점검

| 오해 또는 증상 | 구분해서 확인할 것 |
| --- | --- |
| 모든 UPDATE가 성공했으니 업무도 성공 | 영향받은 행 수와 업무의 사후 조건 |
| BEGIN을 여러 번 쓰면 독립된 중첩 작업 | 저장점과 바깥 COMMIT의 경계 |
| ROLLBACK하면 자동 생성 번호도 항상 되돌아감 | 시퀀스 등 별도 동작을 갖는 객체 |
| ACID이면 외부 결제도 같이 취소 | DB 밖의 효과와 별도 작업 경계 |
| 오류를 잡았으니 다음 SQL을 실행하면 됨 | DB 세션의 실패 상태와 정리 여부 |
| 트랜잭션이 길수록 더 안전 | 보유 자원·잠금·버전 유지와 업무 경계 |

PostgreSQL의 시퀀스 변경처럼 일반 행 변경과 다른 규칙을 가진 기능도 있다. 따라서 원자성을 설명할 때 “프로그램이 건드린 모든 것이 과거로 돌아간다”라고 표현하지 않는다. 제품의 비트랜잭션적 동작은 [Transaction Isolation의 중요 주의사항](https://www.postgresql.org/docs/18/transaction-iso.html)에 정리되어 있다.

운영 로그에는 트랜잭션의 시작·종료, 대상 업무 식별자, 오류 코드, 영향 행 수를 연결해 두면 좋다. 단일 SQL의 시간만으로는 잠금을 오래 붙잡는 트랜잭션의 원인을 설명하기 어렵다. 커밋 성공과 애플리케이션 응답 성공도 다른 이벤트로 관찰한다.

## 14. 핵심 정리

<callout icon="✅" color="green_bg">
트랜잭션은 올바른 업무 경계와 성공 조건을 전제로 한다. ACID의 네 성질을 나누고, 연결·실패 상태·격리 수준·지속성 설정까지 함께 이해한다.
</callout>

- 여러 SQL의 실행 성공과 전체 변경의 확정은 다르다.
- 원자성은 부분 확정, 격리성은 동시 간섭이라는 다른 문제를 다룬다.
- 제약과 업무 로직은 일관성의 전제이며 DBMS가 모든 의도를 추론하지 않는다.
- 저장점은 부분 취소 지점이며 독립적인 외부 커밋이 아니다.
- WAL은 로그를 먼저 안전하게 기록하여 복구할 기반을 만든다.
- 같은 트랜잭션은 같은 세션에서 실행하고 실패 상태를 정리한다.
- 오류로 중단된 경우와 커밋 결과를 모르는 경우를 나누어 재시도한다.

## 출처

- [PostgreSQL 18 — Transactions](https://www.postgresql.org/docs/18/tutorial-transactions.html), [BEGIN](https://www.postgresql.org/docs/18/sql-begin.html): 자동 커밋, 작업 경계와 확정·취소.
- [SAVEPOINT](https://www.postgresql.org/docs/18/sql-savepoint.html), [ROLLBACK TO SAVEPOINT](https://www.postgresql.org/docs/18/sql-rollback-to.html): 부분 취소와 바깥 트랜잭션의 관계.
- [MySQL 8.4 — InnoDB and the ACID Model](https://dev.mysql.com/doc/refman/8.4/en/mysql-acid.html): ACID의 구분과 구현 설정.
- [PostgreSQL 18 — WAL](https://www.postgresql.org/docs/18/wal-intro.html), [Asynchronous Commit](https://www.postgresql.org/docs/18/wal-async-commit.html): 로그 기록 순서와 지속성·성능 선택.
- [Transaction Isolation](https://www.postgresql.org/docs/18/transaction-iso.html), [SET TRANSACTION](https://www.postgresql.org/docs/18/sql-set-transaction.html): 표준 수준·실제 구현, 시퀀스의 별도 규칙과 설정 시점.
- [Serialization Failure Handling](https://www.postgresql.org/docs/18/mvcc-serialization-failure-handling.html): 전체 트랜잭션 재시도.
- [node-postgres — Transactions](https://node-postgres.com/features/transactions): 같은 연결을 사용하는 트랜잭션과 오류 시 정리.

## 15. 이해도 확인 질문

1. 이체의 출금과 입금 SQL이 각각 자동 커밋된다면 어떤 실패 상태가 가능한가? 같은 트랜잭션으로 묶어도 대상 계좌의 UPDATE가 0행이면 왜 추가 판단이 필요한가?
2. 원자성·일관성·격리성·지속성에 각각 대응하는 실패 사례를 들고 DBMS가 자동으로 알 수 없는 업무 규칙도 설명하라.
3. SQL 오류 뒤의 실패 상태, ROLLBACK, ROLLBACK TO SAVEPOINT, COMMIT은 무엇이 다른가? 저장점이 독립적인 커밋이 아닌 이유는 무엇인가?
4. 여러 요청이 연결 풀을 사용하는 상황에서 한 트랜잭션의 모든 SQL을 같은 연결로 수행해야 하는 이유와 연결 반환 전 확인할 상태를 설명하라.
5. 직렬화 실패 오류와 COMMIT 직후 응답을 받지 못한 네트워크 오류를 같은 방식으로 재시도하면 안 되는 이유는 무엇인가? 재실행할 범위와 중복 여부를 어떻게 나누어 판단할 것인가?
