---
track: CS
notion: https://app.notion.com/p/416178c4afac810bbfebce6b6f0fc75c
---

# Table / Row / Column / PK / FK / Cardinality

<callout icon="📍" color="gray_bg">
**학습 범위**: Database · 2/7 · 테이블의 단위, 키와 관계의 제약

한 행이 나타내는 사실에서 시작해 PK·FK·복합 키·NULL·cardinality를 연결한다. SQL은 PostgreSQL 18 기준이며 이 페이지의 실습은 별도 스키마에서 진행한다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 테이블의 행 단위와 열의 의미를 먼저 정하고 키를 선택한다.
- 슈퍼키·후보 키·PK·복합 키·FK의 차이를 예제로 설명한다.
- 1:1·1:N·N:M과 필수·선택 참여를 SQL 제약으로 어디까지 표현할 수 있는지 판단한다.
- NULL, UNIQUE, CHECK의 동작을 구분하여 의도하지 않은 허용을 찾는다.
- 관계 설계의 cardinality와 값 분포·실행 계획의 cardinality를 구분한다.

## 2. 이 주제를 왜 배우는가

회원 테이블에 `id` 열을 넣는 것만으로 설계가 끝나지는 않는다. 같은 이메일로 회원이 두 명 생겨도 되는지, 주문이 회원 없이 존재해도 되는지, 한 주문에 같은 상품을 두 행으로 나눌 수 있는지 결정해야 한다. 이런 질문의 답이 키와 제약 조건이다.

잘못된 설계는 처음 몇 행에서는 드러나지 않을 수 있다. 여러 요청이 동시에 들어오거나 데이터를 삭제할 때 깨진다. 이번 페이지에서는 그림의 연결 선과 실제 DB가 검사하는 규칙을 나누어 살펴본다. 뒤의 정규화에서도 어떤 키가 사실을 결정하는지 알아야 하므로 후보 키의 의미를 정확히 이해해야 한다.

다음 명령을 먼저 실행한 뒤 3절의 테이블 생성 예제를 사용한다. 이미 실습했다면 새 스키마 이름을 사용한다.

```sql
CREATE SCHEMA db_keys_lab;
SET search_path TO db_keys_lab;
```

## 3. 테이블과 키의 기본 구조

### 테이블은 어떤 사실을 한 행으로 저장하는가

회원 테이블의 한 행은 회원 한 명, 주문 테이블의 한 행은 주문 한 건을 나타내도록 정할 수 있다. **한 행이 무엇을 의미하는지**부터 정해야 키와 제약 조건을 설계할 수 있다.

| 개념 | 의미 | 예 |
| --- | --- | --- |
| Table | 같은 구조의 행을 담는 이름 있는 구조 | `users` |
| Row | 한 대상이나 사실을 나타내는 값의 묶음 | 회원 ID 1인 회원 한 명 |
| Column | 각 행에서 같은 의미를 가지는 속성 | `email`, `name` |
| Schema | 테이블·열·타입·제약 등 구조의 정의 | `email`은 필수이며 중복 불가 |

여기서 schema는 구조 정의라는 일반적인 의미다. PostgreSQL에서 `public` 같은 객체 이름 공간도 schema라고 부르므로 문맥을 구분한다. 문자열·정수 같은 타입은 값의 종류를 제한하고, 필수 여부나 중복 금지 같은 규칙은 제약 조건으로 더 좁힌다.

### PK: 행을 식별하는 기준

**기본 키(Primary Key, PK)** 는 한 행을 유일하게 식별하도록 선택한 열 또는 열의 조합이다. 값이 중복되거나 `NULL`이어서는 안 된다. 한 테이블의 PK는 하나지만 그 PK가 여러 열로 구성될 수 있다.

- **슈퍼키(superkey)**: 행을 유일하게 식별할 수 있는 속성 집합이다.
- **후보 키(candidate key)**: 불필요한 속성을 제거할 수 없는 최소 슈퍼키다.
- **기본 키**: 후보 키 중 대표 식별 기준으로 선택한 것이다.
- **복합 키(composite key)**: 둘 이상의 열을 함께 사용하는 키다.

회원 ID와 필수·유일한 이메일이 각각 회원을 식별한다면 둘 다 후보 키가 될 수 있다. 그중 ID를 PK로 선택해도 이메일의 중복 금지 규칙은 별도로 필요하다. 번호가 증가한다는 사실과 유일성을 보장하는 제약도 구분한다.

### FK: 참조하는 값이 실제로 존재하는가

**외래 키(Foreign Key, FK)** 는 다른 테이블 또는 같은 테이블의 키 값을 참조하도록 하는 제약이다. 주문의 `user_id`에 FK를 선언하면 존재하지 않는 회원을 참조하는 주문을 막을 수 있다. 이것을 **참조 무결성(referential integrity)** 이라고 한다.

참조 대상은 반드시 PK만 가능한 것은 아니다. PostgreSQL은 적절한 유일성 제약을 만족하는 열도 허용한다. 반대로 **참조하는 쪽의 FK 값은 중복될 수 있다.** 같은 회원이 주문을 여러 번 할 수 있기 때문이다. 필수 관계라면 `NOT NULL`도 필요하다. PK·FK와 삭제 동작의 구체적인 규칙은 [PostgreSQL Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)에 설명되어 있다.

### 회원·주문·상품을 SQL로 표현하기

다음은 이 문서에서 사용할 작은 예제다. 새 학습용 DB에서 실행하며, ID는 설명을 단순하게 하기 위해 직접 지정한다. 주문 한 건에서 같은 상품은 한 행으로 합치고 `quantity`로 수량을 관리한다고 가정한다.

```sql
CREATE TABLE users (
    id bigint PRIMARY KEY,
    email text NOT NULL UNIQUE,
    name text NOT NULL
);

CREATE TABLE products (
    id bigint PRIMARY KEY,
    name text NOT NULL,
    price numeric(12, 2) NOT NULL CHECK (price >= 0),
    stock integer NOT NULL CHECK (stock >= 0)
);

CREATE TABLE orders (
    id bigint PRIMARY KEY,
    user_id bigint NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    status text NOT NULL CHECK (status IN ('pending', 'paid')),
    created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE order_items (
    order_id bigint REFERENCES orders(id) ON DELETE CASCADE,
    product_id bigint REFERENCES products(id) ON DELETE RESTRICT,
    quantity integer NOT NULL CHECK (quantity > 0),
    unit_price numeric(12, 2) NOT NULL CHECK (unit_price >= 0),
    PRIMARY KEY (order_id, product_id)
);
```

`order_items`의 두 FK 열은 복합 PK에 포함되므로 `NULL`도 허용하지 않는다. `unit_price`는 현재 상품 가격이 아니라 **주문 당시 적용한 단가**다. 같은 주문의 상품 10과 상품 20은 저장할 수 있지만 같은 `(order_id, product_id)` 조합은 다시 넣을 수 없다.

`RESTRICT`는 참조 중인 대상의 삭제를 막고 `CASCADE`는 부모가 삭제되면 해당 자식도 삭제한다. 이 예에서는 주문 항목이 주문에 종속된다고 가정했다. 실제 주문 이력 보존 정책은 별도로 정해야 하며, FK를 만들었다는 이유로 모든 관계에 `CASCADE`를 붙이지 않는다.

### Cardinality는 문맥에 따라 뜻이 다르다

| 문맥 | 뜻 | 예 |
| --- | --- | --- |
| 관계 모델 | 관계에 포함된 튜플 수 | 회원이 100명이면 회원 관계의 cardinality는 100 |
| 관계 설계·ER 모델 | 개체 간 연결 수의 제약 | 회원 한 명과 여러 주문의 1:N 관계 |
| 열의 값 분포 | 서로 다른 값의 개수 | 상태 두 종류와 이메일 100개의 차이 |
| 실행 계획 | 연산이 반환할 행 수의 추정 또는 실제 값 | 필터 뒤 10행으로 예상했지만 실제는 1,000행 |

“카디널리티가 높다”만으로는 의미가 충분하지 않다. 열의 서로 다른 값이 많은지, 조인 결과 행 수가 많은지 구체적으로 말해야 한다.

관계 설계에서는 최대 개수와 최소 개수도 나누어 본다.

- **1:N**: 회원 한 명은 주문을 0개 이상 가질 수 있고, 이 예의 주문은 회원을 정확히 한 명 참조한다. `orders.user_id`에 FK와 `NOT NULL`을 둔다.
- **1:1**: 별도의 회원 프로필이 회원을 FK로 참조하면서 그 FK에 `UNIQUE`도 두면 회원당 프로필은 최대 하나다. 이것만으로 모든 회원에게 프로필이 반드시 존재하게 되지는 않는다.
- **N:M**: 주문에는 여러 상품이, 상품에는 여러 주문이 연결된다. `order_items`라는 연결 테이블로 풀어 두 개의 1:N 관계로 표현한다.

현재 데이터에 회원마다 주문이 한 개씩만 있어도 설계가 1:1인 것은 아니다. 관계의 cardinality는 **현재 관찰한 건수와 허용하는 규칙**을 구분해서 판단한다. 최소·최대 참여 제약은 [Berkeley CS186 DB Design](https://cs186berkeley.net/notes/note13/)을 참고한다.

## 4. 행의 단위가 키보다 먼저다

### 같은 이름의 주문 테이블도 다른 사실을 담을 수 있다

주문 번호 100에 키보드 두 개와 마우스 한 개가 들어 있다고 하자. 한 행을 주문 한 건으로 정하면 주문 시각·회원·상태를 한 번 저장한다. 한 행을 주문에 포함된 상품 하나로 정하면 같은 주문 번호가 두 행에 나타난다. 둘은 서로 다른 단위이므로 같은 키 규칙을 그대로 적용하면 안 된다.

| 테이블 | 한 행의 의미 | 식별 예 |
| --- | --- | --- |
| `orders` | 주문 한 건 | 주문 번호 100 |
| `order_items` | 주문에 들어간 상품 한 종류 | 주문 100과 상품 10의 조합 |
| `products` | 현재 관리하는 상품 하나 | 상품 번호 10 |

한 행의 단위를 **grain**이라고도 부른다. “주문 데이터”라는 업무 이름보다 구체적으로 정의해야 한다. 주문에 같은 상품이 옵션·할인·배송 묶음별로 여러 줄 나올 수 있다면 `(order_id, product_id)`는 충분한 키가 아닐 수 있다. 그 경우에는 항목 번호나 옵션 등 실제 식별 조건을 새로 정한다.

반대로 행마다 아무 숫자나 붙여 유일하게 만들었다고 의미상 중복이 사라지는 것도 아니다. 회원 ID 1과 2에 같은 필수 이메일이 들어가는 것을 금지하려면 이메일에 대한 유일성 제약이 필요하다. 물리적으로 구별되는 행과 업무상 서로 다른 대상은 같은 질문이 아니다.

### 현재의 우연한 유일성과 키의 규칙

샘플 회원 세 명의 이름이 모두 달라도 이름이 후보 키라는 결론은 낼 수 없다. 앞으로 동명이인이 허용된다면 이름은 식별 규칙이 아니다. 키는 현재 표를 관찰해 우연히 찾은 구분자가 아니라 **허용되는 모든 상태에서 유일해야 하는 속성 집합**이다.

후보 키의 최소성도 “열이 하나여야 한다”는 뜻이 아니다. `(tenant_id, member_no)`가 함께 있어야 회원을 식별하고 둘 중 하나를 빼면 식별할 수 없다면 두 열의 조합이 최소일 수 있다. 최소성은 속성 수가 전 세계에서 가장 작다는 말이 아니라, 그 키의 어느 속성도 불필요하지 않다는 뜻이다. 키의 형식적 구분은 [Database System Concepts 2장](https://db-book.com/slides-dir/PDF-dir/ch2.pdf)을 참고한다.

## 5. 자연 키·대리 키·자동 생성 값을 구분하기

**자연 키(natural key)** 는 업무 자체의 식별 값이고, **대리 키(surrogate key)** 는 식별을 위해 별도로 부여한 값이다. 사번·국가 코드 등이 자연 키 후보가 될 수 있고, 의미 없는 회원 번호는 대리 키로 쓸 수 있다. 어느 값이 안정적인지는 서비스의 규칙에 달려 있다.

이메일은 사람이 이해하기 쉽지만 변경 정책이나 대소문자 처리 정책을 고려해야 한다. 이를 PK로 사용하면 참조하는 곳에도 그 변화가 영향을 줄 수 있다. 별도의 `id`를 PK로 사용하면 참조 식별자를 안정적으로 유지하기 쉬우나, 이메일의 중복 금지를 버려도 된다는 뜻은 아니다.

```sql
CREATE TABLE identities (
    id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    external_code text NOT NULL UNIQUE
);

INSERT INTO identities (external_code) VALUES ('MEMBER-A');
```

`GENERATED ... AS IDENTITY`는 값을 만드는 방식이고 `PRIMARY KEY`는 값의 유일성과 필수 여부를 보장하는 제약이다. PostgreSQL의 identity 열은 자동으로 NOT NULL이 되지만 자동 생성 기능 자체가 유일성 제약을 대신하지는 않는다. 시퀀스를 다시 설정하거나 명시적 값을 허용하는 경로 등이 있기 때문이다. [Identity Columns](https://www.postgresql.org/docs/18/ddl-identity-columns.html)에 이 구분이 명시되어 있다.

또한 발급 번호에 빈틈이 있다는 사실만으로 데이터가 유실되었다고 판단하지 않는다. 번호 생성과 업무 레코드의 성공적인 저장은 서로 다른 과정일 수 있다. 연속된 증빙 번호가 필요한 업무 요구를 단순한 PK 자동 생성으로 대체하지 않는다.

## 6. NULL과 제약 조건을 함께 이해하기

### NULL은 0이나 빈 문자열과 다르다

`stock = 0`은 재고가 없음을 아는 값이고 `stock IS NULL`은 재고 값이 없거나 알려지지 않았음을 표현한다. 빈 문자열도 길이가 0인 문자열 값이다. 도메인에 따라 어느 것을 허용할지 정해야 한다.

NULL이 포함된 일반 비교는 참·거짓 외에 알 수 없음으로 평가될 수 있다. 따라서 `column = NULL`로 결측 값을 찾지 않는다. 다음 예는 별도 테이블 없이 실행할 수 있다.

```sql
SELECT
    NULL::integer = NULL::integer AS equality_result,
    NULL::integer IS NULL AS is_null_result,
    0::integer IS NULL AS zero_is_null;
```

첫 결과는 NULL, 두 번째는 참, 세 번째는 거짓이다. `WHERE`는 조건이 참인 행을 남기므로 알 수 없음도 통과시키지 않는다. 이 규칙은 [PostgreSQL Comparison Functions](https://www.postgresql.org/docs/18/functions-comparison.html)에서 확인할 수 있다.

### CHECK만으로 필수 값을 만들지 않는다

```sql
CREATE TABLE optional_prices (
    id integer PRIMARY KEY,
    price numeric CHECK (price >= 0)
);

INSERT INTO optional_prices VALUES (1, NULL);
```

PostgreSQL에서 위 INSERT는 허용된다. CHECK는 식이 거짓이면 위반이며 NULL로 평가된 경우를 NOT NULL처럼 막지 않는다. 가격이 필수라면 `NOT NULL CHECK (price >= 0)`처럼 두 규칙을 함께 선언해야 한다. 타입이 맞는지, 값이 있는지, 범위를 만족하는지는 다른 검사다.

### UNIQUE와 PK는 같은 선언이 아니다

한 테이블에는 여러 UNIQUE 제약을 둘 수 있지만 PK는 하나다. PostgreSQL의 기본 UNIQUE는 NULL 값들을 서로 같은 중복으로 취급하지 않으므로 nullable 열에 NULL이 여러 번 들어갈 수 있다. NULL도 한 번만 허용해야 한다면 제품이 제공하는 `NULLS NOT DISTINCT` 같은 규칙을 검토한다.

후보 키로 사용하려는 업무 식별 값에는 `UNIQUE`와 `NOT NULL`을 함께 두는 것이 의미를 분명히 한다. NULL 처리까지 제품이 모두 같다고 가정하지 말고 사용 DBMS의 규칙을 확인한다. CHECK·UNIQUE·PK의 정확한 조건은 [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html)와 [CREATE TABLE](https://www.postgresql.org/docs/18/sql-createtable.html)을 참고한다.

## 7. 관계의 최대 개수와 최소 개수를 SQL로 표현하기

### 1:N의 1은 각 자식이 참조할 수 있는 부모의 수다

`orders.user_id NOT NULL REFERENCES users(id)`는 한 주문이 유효한 회원 한 명을 참조하도록 한다. 회원이 주문을 반드시 한 건 이상 가져야 한다는 뜻은 아니다. 주문을 아직 하지 않은 회원도 유효한 상태다.

반대로 FK를 nullable로 두면 주문이 부모를 참조하지 않는 상태가 허용될 수 있다. 이것을 허용할지는 비회원 주문 같은 업무 규칙으로 정한다. 관계의 선에 1과 N을 쓰는 것 외에 0..1, 1..N처럼 참여의 최소값도 기록하면 필수 관계를 놓치기 어렵다.

### 1:1에는 유일성과 방향이 모두 중요하다

```sql
CREATE TABLE profiles (
    user_id bigint PRIMARY KEY REFERENCES users(id),
    bio text
);
```

이 구조는 프로필 하나가 회원 한 명을 참조하고 회원당 프로필이 최대 하나임을 보장한다. 하지만 프로필이 아직 없는 회원은 허용된다. `users`의 모든 행이 `profiles`에 반드시 대응해야 한다는 제약을 이 FK 하나가 역방향으로 만들어 주지는 않는다.

두 사실이 항상 함께 존재하고 수명도 같다면 한 테이블로 둘지 고민할 수 있다. 별도로 나누는 경우에는 접근 빈도·선택 여부·권한·수명 같은 이유를 분명히 한다. 다만 테이블을 나누었다는 사실 자체가 성능 개선을 보장하지는 않는다.

### N:M은 연결 자체의 속성을 갖기도 한다

학생과 강좌가 N:M이라면 수강 신청은 단순한 연결 선을 넘어 신청일·성적 같은 속성을 가질 수 있다. 연결 테이블의 한 행은 학생 한 명이나 강좌 하나가 아니라 **해당 수강 사실**이다. 주문 항목의 수량과 주문 당시 단가도 같은 방식으로 연결에 속하는 사실이다.

같은 학생이 같은 강좌를 다음 학기에 다시 들을 수 있다면 `(student_id, course_id)`만 PK로 지정하면 너무 강한 제약이 된다. 학기나 강좌 개설 회차까지 포함해 한 행의 단위를 다시 정의해야 한다. 키 선택은 관계의 업무 의미를 따라야 한다. 최소·최대 참여의 모델링은 [Berkeley CS186 DB Design](https://cs186berkeley.net/notes/note13/)을 참고한다.

## 8. 복합 FK는 조합 전체를 참조한다

여러 회사가 같은 회원 번호를 각자 사용하는 시스템을 생각해 보자. 회사 1의 회원 7과 회사 2의 회원 7은 서로 다른 사람일 수 있다. 이때 `member_no`만으로 참조하면 다른 회사의 회원과 섞일 수 있다.

```sql
CREATE TABLE tenant_members (
    tenant_id integer NOT NULL,
    member_no integer NOT NULL,
    name text NOT NULL,
    PRIMARY KEY (tenant_id, member_no)
);

CREATE TABLE tenant_requests (
    id integer PRIMARY KEY,
    tenant_id integer NOT NULL,
    member_no integer NOT NULL,
    FOREIGN KEY (tenant_id, member_no)
        REFERENCES tenant_members (tenant_id, member_no)
);

INSERT INTO tenant_members VALUES (1, 7, '하나'), (2, 7, '준');
INSERT INTO tenant_requests VALUES (1000, 1, 7);
```

마지막 행은 회사 1의 회원 7을 참조한다. 두 열을 별도 FK로 나누어 “회사 번호가 어딘가 존재하고 회원 번호도 어딘가 존재한다”만 검사하면 의도한 조합의 유효성을 지킬 수 없다. **각 값의 존재와 그 값들의 조합의 존재는 다르다.**

참조하는 열과 참조 대상 열의 대응 순서도 맞춰야 한다. 복합 PK의 구성과 테이블의 표시 열 순서는 구분하고, DDL에서 실제 FK가 어느 열을 참조하는지 확인한다.

## 9. 삭제와 갱신의 파급 범위를 정하기

FK는 부모 삭제 때도 의미가 있다. 회원이 탈퇴했다고 그 회원의 모든 결제 이력까지 지울지, 회원 식별 정보만 별도로 처리할지는 데이터 보존 요구에 달려 있다.

| 동작 | 의미 | 검토할 상황 |
| --- | --- | --- |
| RESTRICT | 참조 중인 부모의 삭제·대상 변경을 제한 | 독립적인 이력을 보존해야 할 때 |
| NO ACTION | 검사 시점에 위반이 있으면 실패 | 지연 검사 등 트랜잭션 규칙과 함께 판단 |
| CASCADE | 부모의 변경을 자식으로 전파 | 부모에 종속되는 구성 요소 |
| SET NULL | 참조 열을 NULL로 변경 | 관계를 선택적으로 끊어도 되는 경우 |

`SET NULL`을 지정해도 해당 열의 NOT NULL 규칙이 없어지는 것은 아니다. 두 설정이 충돌하면 실제 작업이 실패할 수 있다. CASCADE도 “잘 지워 주는 기능” 정도로 생각하면 안 된다. 연결된 자식의 범위가 크면 한 번의 삭제가 많은 데이터를 바꿀 수 있다.

PostgreSQL에서 RESTRICT와 NO ACTION은 지연 검사 가능성 등의 차이가 있으므로 완전한 동의어로 외우지 않는다. 이 페이지의 기본 예는 검사를 미루지 않는 일반적인 상황이며, 복잡한 지연 제약 설계는 별도 범위다.

## 10. 조인과 cardinality를 숫자로 관찰하기

### 연결 개수와 결과 행 수

회원 1에게 주문 2건, 회원 2에게 주문 0건이 있다고 하자. 회원과 주문을 내부 조인하면 2행이다. 회원 1의 이름이 두 번 나타나는 것은 1:N 관계를 주문 단위로 펼친 결과다. 회원 수를 세려는데 조인 결과의 행 수를 그대로 세면 잘못된 답을 얻을 수 있다.

3절의 스키마에서 다음 데이터를 추가한 뒤 확인해 볼 수 있다.

```sql
INSERT INTO users VALUES
    (1, 'hana@example.com', '하나'),
    (2, 'jun@example.com', '준');
INSERT INTO orders (id, user_id, status) VALUES
    (100, 1, 'pending'), (101, 1, 'paid');

SELECT u.id, u.name, o.id AS order_id
FROM users AS u JOIN orders AS o ON o.user_id = u.id
ORDER BY o.id;

SELECT COUNT(*) AS joined_rows,
       COUNT(DISTINCT u.id) AS members_with_orders
FROM users AS u JOIN orders AS o ON o.user_id = u.id;
```

첫 조회는 회원 1에 대해 두 행을, 두 번째는 2와 1을 반환한다. 조인 조건을 생략한 곱집합은 이와 다른 조합을 만들 수 있으므로 항상 결과의 단위와 조건을 먼저 확인한다. 관계 결합의 정확한 의미는 [Table Expressions](https://www.postgresql.org/docs/18/queries-table-expressions.html)를 참고한다.

### 서로 다른 값 수와 선택되는 행 비율

주문 상태가 두 종류라는 말은 열의 값 cardinality가 작다는 뜻이다. 그러나 `pending`이 전체의 절반인지 0.1%인지는 별도 정보다. 같은 값 종류 수라도 어떤 조건이 선택하는 행의 비율은 크게 달라질 수 있다.

실행 계획의 `rows`는 보통 해당 연산이 반환할 행 수의 추정이다. 실제 결과와 차이가 크다면 분포 통계와 조건 사이의 관계 등을 살핀다. “회원과 주문이 1:N이다”라는 설계 제약만으로 모든 회원의 주문 수가 같다고 가정할 수 없다. 이 연결은 [PostgreSQL Row Estimation Examples](https://www.postgresql.org/docs/18/row-estimation-examples.html)에 설명되어 있다.

## 11. 규칙을 선언한 DB와 미리 조회만 한 코드

두 회원 가입 요청이 동시에 이메일의 존재 여부를 조회하면 둘 다 “없음”을 볼 수 있다. 그 뒤 각각 INSERT하면 사전 조회만으로는 중복을 막지 못한다. UNIQUE 제약이 있으면 둘 중 하나의 충돌을 DB가 검사할 수 있다.

사전 검사는 사용자에게 친절한 안내를 만드는 데 도움이 되지만 최종 저장 규칙의 대체물은 아니다. DB 제약 오류가 난 경우에도 적절한 사용자 응답을 만드는 애플리케이션 처리가 필요하다. 잘못된 값을 DB가 거절했다는 사실은 API 응답 설계까지 완료했다는 뜻이 아니다.

또한 FK는 모든 관련 업무 규칙을 검사하지 않는다. 주문 항목이 존재하는 상품을 참조하더라도 그 상품이 판매 중인지, 주문 가능한 날짜인지는 별도 조건일 수 있다. **참조 유효성, 값 범위, 업무 상태 규칙**을 한 가지 제약의 이름으로 뭉뚱그리지 않는다.

## 12. 흔한 오해와 주의점

### “FK가 있으면 조인을 따로 작성할 필요가 없다”

FK는 저장 상태를 제한하는 제약이다. 조회에서 어떤 열을 결합하고 어떤 행을 남길지는 SQL로 표현해야 한다. FK가 없는 열도 값 조건으로 조인할 수 있으나, 저장된 참조가 유효하다는 보장을 얻는 것은 별개다.

### “1:1이면 양쪽 행이 항상 동시에 존재한다”

FK와 UNIQUE로 최대 한 개를 제한한 것과 양쪽에 반드시 대응 행이 있어야 하는 것은 다른 제약이다. 어느 쪽의 선택 참여가 허용되는지 방향을 포함해 설명해야 한다.

### “PK를 추가하면 모든 의미상 중복이 제거된다”

서로 다른 대리 키로 같은 이메일을 저장하는 것은 PK 위반이 아닐 수 있다. 정말 유일해야 하는 업무 속성에 제약을 선언했는지 확인한다. 이름이나 표시 순서를 키처럼 사용하지 않는다.

## 13. 성능·운영·디버깅 관점

제약 위반이 나면 먼저 **어느 규칙이 무엇을 보호하는지** 확인한다. 중복 오류가 난다고 유일성 제약을 없애면 잘못된 쓰기 경로를 숨길 수 있다. FK 오류는 참조 대상이 없거나 삭제·갱신하려는 대상을 자식이 참조하는 상황일 수 있으므로 오류의 방향을 살핀다.

성능에서는 PK·UNIQUE를 위한 인덱스와 참조하는 FK 열의 인덱스를 구분한다. PostgreSQL은 FK 선언만으로 자식 열의 인덱스를 자동 생성하지 않는다. 부모 삭제나 자식 검색이 빈번하다면 실제 접근 패턴을 검토해야 한다. 다만 이 판단을 유일성 같은 데이터 규칙 자체와 혼동하지 않는다.

스키마를 검토할 때는 표본 데이터뿐 아니라 허용하면 안 되는 사례를 적어 본다. “같은 주문의 같은 상품 두 번”, “없는 회원의 주문”, “부모 삭제 후 남는 참조”, “NULL인 필수 값”을 실제로 넣어 보는 것이 DDL이 의도한 규칙을 표현하는지 확인하는 방법이다. 실패 예제는 정상 예제와 분리하여 실행한다.

## 14. 핵심 정리

<callout icon="✅" color="green_bg">
한 행의 의미를 먼저 정하고, 식별·참조·개수·필수 여부를 각각의 제약으로 표현한다. 같은 그림처럼 보여도 실제로 검사하는 규칙은 다를 수 있다.
</callout>

- 후보 키는 최소 슈퍼키이고 PK는 그중 선택한 하나다.
- 대리 키와 자동 생성 기능은 업무상 유일성 규칙을 대신하지 않는다.
- FK는 값 또는 값 조합의 참조 유효성을 검사하며 N쪽에서는 반복될 수 있다.
- 1:1·1:N·N:M의 최대 수와 참여의 최소 수를 함께 확인한다.
- NULL·CHECK·UNIQUE의 제품별 규칙을 알고 필수 값은 명확히 선언한다.
- cardinality는 관계의 행 수, 관계 제약, 서로 다른 값 수, 실행 계획 행 수를 문맥에 따라 뜻한다.

## 출처

- [Database System Concepts 2장](https://db-book.com/slides-dir/PDF-dir/ch2.pdf): 슈퍼키·후보 키·기본 키와 관계 구조.
- [PostgreSQL 18 — Table Basics](https://www.postgresql.org/docs/18/ddl-basics.html), [Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html), [CREATE TABLE](https://www.postgresql.org/docs/18/sql-createtable.html): 행·열·타입, 키, NULL, CHECK, 참조 동작.
- [PostgreSQL 18 — Identity Columns](https://www.postgresql.org/docs/18/ddl-identity-columns.html), [Comparison Functions](https://www.postgresql.org/docs/18/functions-comparison.html): 자동 생성과 유일성의 차이, NULL의 비교.
- [Berkeley CS186 — DB Design](https://cs186berkeley.net/notes/note13/): 관계의 최대 연결 수와 최소 참여 제약.
- [PostgreSQL 18 — Table Expressions](https://www.postgresql.org/docs/18/queries-table-expressions.html), [Row Estimation Examples](https://www.postgresql.org/docs/18/row-estimation-examples.html): 조인 결과와 cardinality 추정.

## 15. 이해도 확인 질문

1. 같은 상품이 옵션별로 여러 줄 나올 수 있는 주문에서 `(order_id, product_id)`를 PK로 선택해도 되는가? 한 행의 의미와 후보 키의 최소성을 근거로 설명하라.
2. 자동 생성 ID가 PK인 회원 테이블에서 이메일 중복과 NULL을 막으려면 어떤 제약이 필요한가? 자동 생성·PK·UNIQUE·NOT NULL의 책임을 나누어 설명하라.
3. 프로필의 `user_id`에 PK와 FK가 있으면 회원마다 프로필이 반드시 하나씩 존재하는가? 최소 참여와 최대 연결 수를 구분하라.
4. 회사마다 회원 번호 7을 사용할 수 있다. 두 열의 복합 FK가 각각 따로 만든 FK와 어떻게 다르며 어떤 잘못된 조합을 막는가?
5. 회원 한 명과 주문 100건을 조인했을 때 행 수, 서로 다른 회원 수, 상태 열의 서로 다른 값 수는 어떻게 달라질 수 있는가? 이들을 모두 cardinality라고만 말하면 왜 혼동하는가?
