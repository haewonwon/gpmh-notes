---
track: CS
notion: https://app.notion.com/p/416178c4afac814a8d41e40f63ad99b5
---

# Normalization 기초

<callout icon="📍" color="gray_bg">
**학습 범위**: Database · 3/7 · 정규화의 목적과 1NF·2NF·3NF

중복을 발견하는 데서 끝내지 않고 함수 종속성·후보 키·분해 후 복원 가능성을 근거로 설계를 설명한다. 주문 예제에서 개념을 익힌 뒤 수강 신청 예제를 처음부터 끝까지 정리한다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 갱신·삽입·삭제 이상을 구체적인 데이터의 변화로 설명한다.
- 함수 종속성을 현재 데이터의 우연한 패턴과 구분한다.
- 후보 키와 부분·이행 종속성을 찾아 1NF·2NF·3NF를 판단한다.
- 분해한 테이블이 원래 사실을 보존하는지와 제약을 유지하는지를 확인한다.
- 현재 사실의 중복, 이력 보존, 성능을 위한 파생 데이터의 목적을 구분한다.

## 2. 이 주제를 왜 배우는가

회원 이메일을 바꾸었는데 주문 화면 일부만 옛 이메일을 보여 주거나, 마지막 수강 기록을 지웠더니 강좌 정보까지 사라지는 문제는 단순한 화면 오류가 아닐 수 있다. 서로 다른 사실을 하나의 행에 묶어서 저장한 설계가 원인일 수 있다.

정규화는 이런 문제를 “중복이 많다”는 느낌으로 다루지 않고 어떤 속성이 어떤 속성을 결정하는지 분석한다. 목표는 테이블 수를 최대한 늘리는 것이 아니라, 변경의 의미가 분명하고 잘못된 상태를 만들기 어려운 구조를 얻는 것이다. 이 페이지에서는 키와 참조 관계를 이미 배웠다고 보고 그 위에서 설계를 검토한다.

## 3. 주문 예제로 이해하는 정규화의 기본

### 중복이 왜 문제가 되는가

다음처럼 모든 정보를 주문 항목 한 테이블에 넣었다고 하자. 한 행은 한 주문에 포함된 상품 하나이고 후보 키는 `(order_id, product_id)`라고 가정한다. `product_name`은 상품의 현재 이름, `user_email`은 회원의 현재 이메일을 뜻한다.

| order_id | product_id | quantity | user_id | user_email | product_name |
| --- | --- | --- | --- | --- | --- |
| 100 | 10 | 2 | 1 | hana@example.com | 키보드 |
| 100 | 20 | 1 | 1 | hana@example.com | 마우스 |
| 101 | 10 | 1 | 1 | hana@example.com | 키보드 |

중복은 단순한 저장 공간 낭비를 넘어 **이상 현상(anomaly)** 을 만든다.

| 이상 현상 | 이 테이블에서 생기는 문제 |
| --- | --- |
| 갱신 이상 | 회원의 이메일을 바꿀 때 모든 주문 항목을 함께 고쳐야 함 |
| 삽입 이상 | 아직 주문되지 않은 새 상품을 주문 키 없이 자연스럽게 저장하기 어려움 |
| 삭제 이상 | 상품의 마지막 주문 항목을 삭제하면 그 상품 이름 정보까지 사라짐 |

**정규화(normalization)** 는 함수 종속성과 키를 기준으로 데이터를 구성하여 이런 불필요한 중복과 이상 현상을 줄이는 과정이다. 단순히 테이블 수를 늘리거나 같은 문자열을 모두 별도 테이블로 옮기는 작업은 아니다. [Microsoft 정규화 기초](https://learn.microsoft.com/en-us/previous-versions/troubleshoot/microsoft-365/microsoft-365-apps/access/database-normalization-description)를 참고한다.

### 함수 종속성: 어떤 값이 다른 값을 결정하는가

**함수 종속성(Functional Dependency, FD)** `X -> Y`는 같은 X 값을 가진 튜플은 반드시 같은 Y 값을 가져야 한다는 규칙이다. 특정 샘플에서 우연히 성립하는 상관관계가 아니라 데이터가 지켜야 할 의미상의 제약이다.

위 예제의 업무 규칙은 다음과 같다.

```text
(order_id, product_id) -> quantity
order_id              -> user_id
user_id               -> user_email
product_id            -> product_name
```

회원 ID는 회원 이메일을 결정하지만, 이메일 앞부분이나 이름이 우연히 같다고 회원 ID를 결정하지는 않는다. FD의 방향을 거꾸로 해석하지 않는다.

### 1NF: 한 속성에 반복 집합을 밀어 넣지 않는다

**제1정규형(1NF)** 에서는 각 튜플의 속성 값이 해당 도메인의 단일 값이어야 한다. 주문의 `product_ids`에 `'10,20,30'`을 넣어 여러 상품을 한 문자열로 관리하면 상품별 FK·수량·검색을 표현하기 어려워진다. 주문 항목을 별도 행으로 표현하면 이 관계를 직접 다룰 수 있다.

원자적이라는 말은 문자열을 더 작은 글자로 나눌 수 없어야 한다는 뜻이 아니다. 현재 모델에서 하나의 값으로 다루는 단위라는 뜻이다. SQL이 배열이나 JSON을 지원한다는 사실만으로 모든 반복 관계를 그 안에 저장해야 하는 것도 아니다. 관계로 조회하고 제약을 걸 대상인지 판단한다.

### 2NF: 복합 키의 일부에만 의존하는 사실을 분리한다

**제2정규형(2NF)** 은 1NF를 만족하고, 어떤 후보 키에도 포함되지 않는 속성이 후보 키의 진부분집합에 부분적으로 종속되지 않는 상태다. 입문 예제에서는 “비키 속성이 복합 키 전체에 의존하는가”로 확인할 수 있다.

위 테이블에서 `quantity`는 주문과 상품을 모두 알아야 결정된다. 반면 `product_name`은 `product_id`만으로 결정되고, `user_id`와 `user_email`은 `order_id`만으로도 결정된다. 이런 사실을 분리하면 다음 구조가 된다.

```text
order_items(order_id, product_id, quantity)
products(product_id, product_name)
orders(order_id, user_id, user_email)
```

단일 열 PK를 추가했다고 기존의 함수 종속성이 없어지지는 않는다. 대리 키를 새로 만들더라도 원래 `(order_id, product_id)`가 후보 키라면 그 키에 대한 부분 종속성도 검사해야 한다.

### 3NF: 다른 사실을 거쳐 결정되는 속성을 검토한다

`orders`에는 아직 `order_id -> user_id -> user_email`이라는 **이행 종속(transitive dependency)** 이 남는다. 회원의 현재 이메일은 주문보다 회원에 속하는 사실이므로 다음처럼 분리한다.

```text
users(user_id, user_email)
orders(order_id, user_id)
products(product_id, product_name)
order_items(order_id, product_id, quantity)
```

**제3정규형(3NF)** 을 입문 수준에서는 “2NF를 만족하고 비키 속성 사이의 이행 종속을 제거한다”로 이해할 수 있다. 정확한 조건은 모든 비자명한 FD `X -> A`에 대해 **X가 슈퍼키이거나 A가 어떤 후보 키에 포함되는 속성**이어야 한다는 것이다. 비자명하다는 것은 A가 원래 X에 포함된 속성이 아니라는 뜻이다. 복잡한 후보 키가 있는 경우에는 구호보다 이 조건으로 판단한다. [Juniata 대학 정규화 강의](https://jcsites.juniata.edu/faculty/rhodes/dbms/normalization.htm)를 참고한다.

### 나눈 뒤에도 같은 사실을 복원할 수 있어야 한다

**무손실 분해(lossless decomposition)** 는 나눈 관계를 올바르게 조인하면 원래 관계를 정확히 복원할 수 있다는 뜻이다. 행을 빠뜨리지 않는 것뿐 아니라 원래 없던 조합이 생기지 않는 것도 중요하다. 예제는 주문·회원·상품의 식별자를 남겨 어떤 사실끼리 연결되는지 보존한다.

또한 **종속성 보존(dependency preservation)** 은 기존의 FD를 분해된 관계들에서 검사할 수 있는지에 관한 조건이다. 테이블을 나누었다는 사실만으로 무손실성과 종속성 보존이 자동으로 모두 성립한다고 가정하지 않는다. 이 둘의 차이는 [Berkeley DB Design의 분해 설명](https://cs186berkeley.net/notes/note13/)에 정리되어 있다.

### 정규화의 이득과 비용

정규화하면 한 사실을 수정할 지점이 줄고 제약을 표현하기 쉬워진다. 대신 필요한 정보를 모으기 위해 조인이 늘어날 수 있다. 조인이 있다는 이유만으로 설계가 느리다고 단정하거나, 속도를 예상해 처음부터 모든 데이터를 중복 저장하지 않는다.

중복 저장이 필요하다면 원본과 동기화 방법을 함께 정한다. 특히 **주문 당시 단가**와 **현재 상품 가격**은 서로 다른 사실이다. `order_items.unit_price`를 남겨 결제 내역을 보존하는 것은 현재 상품 가격을 불필요하게 복제하는 문제와 구분해야 한다.

## 4. 함수 종속성을 읽는 정확한 방법

### 결정한다는 말은 함수 코드를 실행한다는 뜻이 아니다

`student_id -> student_name`은 ID가 같으면 이름도 같아야 한다는 규칙이다. 이름을 계산하는 JavaScript 함수가 있다는 뜻도, 이름이 영원히 바뀌지 않는다는 뜻도 아니다. 특정 시점의 유효한 데이터에서 같은 학생 ID에 서로 다른 현재 이름이 동시에 있어서는 안 된다는 뜻이다.

반대 방향인 `student_name -> student_id`는 별도 규칙이다. 동명이인이 있으면 성립하지 않는다. `X -> Y`와 `Y -> X`를 한 문장으로 섞지 않아야 어떤 속성이 키인지 정확히 판단할 수 있다.

또한 FD는 많은 행이 있어야만 성립하는 통계 법칙이 아니다. 행이 한 개라 모든 열이 우연히 유일해 보여도 업무 규칙을 확인해야 한다. 샘플 데이터는 위반 사례를 찾는 데 도움이 되지만 앞으로의 모든 적법한 상태에서 FD가 성립한다는 근거를 대신하지 않는다.

### 시간의 범위가 바뀌면 FD도 달라질 수 있다

현재 부서만 기록하는 직원 표에서는 `employee_id -> department_id`를 가정할 수 있다. 직원의 모든 부서 이동 이력을 같은 표에 넣으면 한 직원에게 여러 부서가 대응하므로 같은 가정을 그대로 쓰면 안 된다. 이때는 발령 시점이나 이력 행의 식별 범위를 함께 생각해야 한다.

따라서 정규화 전에 “현재 값인가, 어느 시점의 사실인가, 한 번 정하면 고정되는 값인가”를 문장으로 적는 것이 중요하다. 같은 이름의 열이라도 의미가 다르면 같은 FD를 적용할 수 없다. 이런 의미 기반 접근은 [Database System Concepts 7장](https://db-book.com/slides-dir/PDF-dir/ch7.pdf)의 함수 종속성 정의와 연결된다.

## 5. 수강 신청 예제의 규칙을 먼저 정하기

한 학생이 한 강좌 개설 회차에 한 번 신청한다고 하자. `class_id`는 추상적인 과목명이 아니라 **특정 학기의 특정 분반**을 식별한다. 같은 과목을 다음 학기에 다시 열면 다른 `class_id`를 부여한다.

처음에는 다음 속성을 모두 하나의 `registration_raw`에 넣었다고 하자.

```text
registration_raw(
  student_id, class_id, grade,
  student_name, department_id, department_name,
  class_title, instructor_id, instructor_name
)
```

업무에서 다음 규칙을 채택한다고 명시한다. 강좌 한 회차에 여러 강사를 두거나 학생의 복수 전공을 다루는 확장은 이번 모델의 범위 밖이다.

```text
student_id -> student_name, department_id
department_id -> department_name
class_id -> class_title, instructor_id
instructor_id -> instructor_name
(student_id, class_id) -> grade
```

이 규칙 아래에서 `(student_id, class_id)`로 행의 모든 속성을 결정할 수 있다. 학생 ID를 빼면 같은 강좌의 여러 학생을 구분하지 못하고, 강좌 ID를 빼면 같은 학생의 여러 신청을 구분하지 못한다. 따라서 이 조합을 후보 키로 사용할 수 있다.

한 이름이 여러 번 나타난다는 사실보다 위 규칙이 중요하다. 학생 하나가 신청한 강좌마다 이름을 반복 저장하면 이름 갱신이 여러 행으로 퍼진다. 반면 성적은 학생만 알아도, 강좌만 알아도 결정되지 않는다. 이 차이가 분해의 근거다.

## 6. 1NF에서 확인할 것은 표현의 단위다

다음처럼 한 행의 `class_ids`에 여러 강좌를 문자열로 붙이면 강좌마다 다른 성적을 연결하기 어렵다.

```text
student_id = 1
class_ids  = 'JS-1,DB-1'
grades     = 'A,B'
```

두 목록의 순서가 어긋나면 어느 성적이 어느 강좌에 속하는지 불분명해진다. 문자열에 구분자를 더 정교하게 넣는 것보다 신청 사실을 행으로 표현하면 식별과 참조를 직접 다룰 수 있다.

| student_id | class_id | grade |
| --- | --- | --- |
| 1 | JS-1 | A |
| 1 | DB-1 | B |
| 2 | JS-1 | B |

이 표는 학생 1의 강좌가 두 개여도 한 행에 반복 목록을 만들지 않는다. 다만 1NF를 만족했다고 중복 문제를 모두 해결한 것은 아니다. 여기에 학생 이름과 부서 이름을 붙이면 여전히 같은 학생 사실이 여러 신청 행에 반복된다.

문자열의 내부를 기술적으로 나눌 수 있는지와 모델의 값 단위는 다른 문제다. 주소 문자열을 반드시 글자 단위 테이블로 나누라는 규칙이 아니다. 어떤 속성을 독립적으로 검색·검사·변경해야 하는지에 맞춰 도메인을 정한다. 반복 그룹을 피하는 기초 설명은 [Microsoft 정규화 설명](https://learn.microsoft.com/en-us/previous-versions/troubleshoot/microsoft-365/microsoft-365-apps/access/database-normalization-description)을 참고한다.

## 7. 2NF로 분해할 때 전체 키와 부분 키를 비교하기

후보 키가 `(student_id, class_id)`인데 이름은 학생 ID만으로 결정된다. 학생의 부서 정보도 학생 ID에서 결정된다. 강좌 제목과 강사 정보는 강좌 ID만으로 결정된다. 성적만 두 ID를 함께 필요로 한다.

따라서 먼저 세 관계로 나눈다.

```text
registrations(student_id, class_id, grade)
students(student_id, student_name, department_id, department_name)
classes(class_id, class_title, instructor_id, instructor_name)
```

이 단계는 아무 열이나 비슷한 것끼리 묶는 작업이 아니다. 학생 ID로 결정되는 사실을 학생 관계로, 강좌 ID로 결정되는 사실을 강좌 관계로 옮긴 것이다. 신청 관계에는 연결을 복원할 두 키와 그 연결 전체에 속하는 성적을 남긴다.

대리 키 `registration_id`를 추가해도 `(student_id, class_id)`의 업무 유일성은 남을 수 있다. 이 조합을 후보 키로 유지한다면 부분 종속성 검토도 남는다. “PK가 한 열이니까 2NF는 무조건 통과”라는 설명은 다른 복합 후보 키가 있는 경우를 놓친다.

비키 속성이라는 말도 명확히 한다. 여기서는 **어떤 후보 키에도 포함되지 않는 속성**을 뜻한다. 선택한 PK에 없다는 이유만으로 모두 비키라고 부르면 다른 후보 키를 가진 설계에서 잘못 판단할 수 있다. 완전 함수 종속성과 2NF의 관계는 [Juniata 정규화 강의](https://jcsites.juniata.edu/faculty/rhodes/dbms/normalization.htm)를 참고한다.

## 8. 3NF로 분해하여 사실의 소유를 정하기

2NF의 `students`에는 학생 ID가 부서 ID를, 부서 ID가 부서 이름을 결정하는 사슬이 남아 있다. 같은 부서의 학생이 100명이면 부서 이름 변경을 100행에 반영해야 한다. 부서 이름은 학생에게 반복 저장할 사실보다 부서 자체에 속하는 사실이다.

강좌도 마찬가지다. 강사 이름은 여러 강좌에서 반복할 현재 정보가 아니라 강사 ID로 결정되는 정보다. 다음처럼 정리한다.

```text
departments(department_id, department_name)
students(student_id, student_name, department_id)
instructors(instructor_id, instructor_name)
classes(class_id, class_title, instructor_id)
registrations(student_id, class_id, grade)
```

이제 학생 이름은 학생 한 곳, 부서 이름은 부서 한 곳, 강사 이름은 강사 한 곳에서 바꾼다. 수강 신청은 학생과 강좌를 참조한다. 부서에 아직 학생이 없어도 부서 행을 먼저 만들 수 있으며 마지막 신청을 지워도 강좌나 학생의 독립적인 사실은 사라지지 않는다.

이 예에서 모든 FD를 말로 검토할 수 있지만, 더 복잡한 후보 키가 있으면 3절의 형식적 조건을 사용한다. “모든 속성이 PK만 보면 된다”라는 구호는 후보 키와 슈퍼키의 조건을 정확히 대체하지 못한다.

<callout icon="🔭" color="gray_bg">
**[미리보기: BCNF와 고차 정규형]**

3NF보다 강한 조건을 사용하는 BCNF나 다른 종류의 종속성을 다루는 정규형도 있다. 이번 범위에서는 1NF·2NF·3NF와 무손실성·종속성 보존을 우선 정확히 이해한다.
</callout>

## 9. 분해가 잘못되면 없던 조합이 생길 수 있다

실제로 다음 두 배정만 존재한다고 하자. 여기서는 같은 과목에 여러 강사가 가능하므로 `subject -> teacher`라는 FD를 가정하지 않는다.

| learner | subject | teacher |
| --- | --- | --- |
| 하나 | JS | 민 |
| 준 | JS | 수 |

이를 `(learner, subject)`와 `(subject, teacher)`로 나누면 각각의 표에는 두 행씩 남는다. 하지만 과목 JS로 조인하면 하나-민, 하나-수, 준-민, 준-수의 **네 조합**이 생긴다. 원래 존재하지 않은 하나-수와 준-민도 구분하지 못하게 된 것이다.

따라서 lossy라는 말은 단순히 행을 삭제해 버렸다는 뜻으로만 읽으면 안 된다. 원래 어떤 값들이 함께 연결되어 있었는지에 관한 정보가 사라져, 재결합 때 가짜 튜플이 생길 수도 있다.

이항 분해에서는 공통 속성이 한쪽 관계의 모든 속성을 결정할 수 있는지가 무손실성 판단의 중요한 기준이다. 예를 들어 학생 ID는 학생 표의 한 행을 식별하므로 신청과 학생을 해당 ID로 연결할 근거가 있다. 모든 분해를 현재 샘플에서만 조인해 보는 것으로 증명하지 말고 선언한 FD 아래에서 성립하는지 확인한다. [Berkeley의 Lossless Decomposition](https://cs186berkeley.net/notes/note13/)을 참고한다.

## 10. 정규화한 구조를 SQL로 구현하고 복원하기

다음 SQL은 별도의 PostgreSQL 학습용 스키마에서 실행한다. 이 예의 키·FK는 위에서 정한 업무 가정을 표현한다.

```sql
CREATE SCHEMA db_normalization_lab;
SET search_path TO db_normalization_lab;

CREATE TABLE departments (
    id integer PRIMARY KEY,
    name text NOT NULL
);
CREATE TABLE students (
    id integer PRIMARY KEY,
    name text NOT NULL,
    department_id integer NOT NULL REFERENCES departments(id)
);
CREATE TABLE instructors (
    id integer PRIMARY KEY,
    name text NOT NULL
);
CREATE TABLE classes (
    id text PRIMARY KEY,
    title text NOT NULL,
    instructor_id integer NOT NULL REFERENCES instructors(id)
);
CREATE TABLE registrations (
    student_id integer REFERENCES students(id),
    class_id text REFERENCES classes(id),
    grade text,
    PRIMARY KEY (student_id, class_id)
);

INSERT INTO departments VALUES (10, '컴퓨터공학');
INSERT INTO students VALUES (1, '하나', 10), (2, '준', 10);
INSERT INTO instructors VALUES (20, '민');
INSERT INTO classes VALUES ('JS-1', 'JavaScript 기초', 20);
INSERT INTO registrations VALUES (1, 'JS-1', 'A'), (2, 'JS-1', 'B');

SELECT s.id AS student_id, s.name AS student_name,
       d.name AS department_name, c.id AS class_id,
       c.title, i.name AS instructor_name, r.grade
FROM registrations AS r
JOIN students AS s ON s.id = r.student_id
JOIN departments AS d ON d.id = s.department_id
JOIN classes AS c ON c.id = r.class_id
JOIN instructors AS i ON i.id = c.instructor_id
ORDER BY s.id, c.id;
```

결과에는 학생 1과 학생 2의 신청 행이 각각 나타난다. 두 학생의 부서 이름이 조회 결과에서 반복되더라도 부서 사실을 두 번 저장한 것은 아니다. 저장 중복과 읽기 때 조합한 결과의 반복을 구분한다.

이어서 부서 이름을 한 번만 바꾸면 두 학생을 조회한 결과에 함께 반영된다.

```sql
UPDATE departments SET name = '소프트웨어학과' WHERE id = 10;
```

실습에서는 이름 변경 후 같은 조인을 다시 실행하여 결과를 확인한다. 또한 신청 두 행을 없애더라도 학생과 강좌가 남는지 확인하면 삭제 이상의 제거를 이해할 수 있다. 다만 운영 데이터에서 삭제를 시험하는 것이 아니라 위의 독립된 학습 스키마를 사용한다.

## 11. 종속성 보존과 제약 구현은 별도로 점검한다

무손실 분해는 정보를 복원할 수 있는지에 대한 성질이고, 종속성 보존은 원래 FD를 분해된 관계에서 검증할 수 있는지에 대한 성질이다. 둘을 하나로 합쳐 “조인만 잘 되면 끝”이라고 판단하지 않는다.

위 예에서는 부서 ID가 부서 이름을 결정하는 사실을 부서 테이블의 PK로 표현한다. 학생이 유효한 부서를 참조한다는 사실은 학생 테이블의 FK로 검사한다. 어느 규칙이 어느 표에서 검사되는지 설명할 수 있어야 한다.

정규화 이론에서 말하는 모든 FD가 단일 SQL 문법으로 자동 선언되는 것은 아니다. DBMS가 직접 제공하는 PK·UNIQUE·FK·CHECK로 표현할 수 있는 규칙과 애플리케이션 또는 다른 제약 메커니즘이 필요한 규칙을 구분한다. 설계 문서의 화살표만 남기고 실제 쓰기 경로에서 검사하지 않는다면 그 FD를 DB가 지켜 준다고 말할 수 없다.

## 12. 현재 사실, 역사적 사실, 파생 결과를 구분하기

### 주문 당시 단가는 현재 상품 가격과 다른 사실이다

오늘 상품 가격이 50000이고 지난달 주문 단가가 45000이었다면 둘을 같은 값으로 맞추면 오히려 이력을 훼손한다. `product_id -> current_price`와 `(order_id, product_id) -> unit_price_at_order`는 다른 의미의 규칙이다.

“비슷한 숫자가 두 군데 있으므로 정규화 위반”이라고 판단하기 전에 시간과 업무 의미를 확인한다. 현재 회원 이메일을 주문마다 반복 저장한 문제와 당시 배송지나 결제 단가를 기록하는 요구는 같지 않다.

### 계산 결과를 보관하면 갱신 책임이 생긴다

수량과 단가에서 계산할 수 있는 금액을 별도로 보관하면 값이 서로 어긋날 수 있다. 항상 같은 행의 값으로 계산하는지, 당시 확정 금액이라는 독립적인 기록인지 먼저 결정한다. PostgreSQL에는 같은 행의 식으로 관리하는 생성 열 같은 기능도 있지만 사용 가능 조건을 확인해야 한다. [Generated Columns](https://www.postgresql.org/docs/18/ddl-generated-columns.html)를 참고한다.

조회 비용 때문에 집계 결과를 보관하기로 했다면 원본, 갱신 시점, 허용할 지연, 재계산 방법을 정한다. PostgreSQL의 materialized view는 결과를 보관하지만 갱신 전에는 최신 원본과 다를 수 있다. 이를 즉시 동기화되는 일반 뷰와 혼동하지 않는다. [Materialized Views](https://www.postgresql.org/docs/18/rules-materializedviews.html)를 참고한다.

## 13. 정규화와 성능을 함께 판단하는 순서

1. 행의 의미와 시점, 유일성, FD를 문장으로 기록한다.
2. 현재 구조의 이상 현상을 실제 입력·수정·삭제 사례로 적는다.
3. 키와 FD를 근거로 분해하고 무손실성과 규칙 보존을 확인한다.
4. 필요한 조회를 작성하고 실제 데이터 분포에서 비용을 측정한다.
5. 인덱스·조회 범위·불필요한 결과량 등을 먼저 검토한다.
6. 중복된 파생 구조가 필요하면 원본과 동기화 규칙을 명시한다.

정규화가 모든 조회를 빠르게 만드는 방법은 아니다. 반대로 조인이 있다는 이유만으로 느리다는 결론도 성립하지 않는다. 한 사실을 100곳에 반복 저장하는 비용과 조인으로 읽는 비용을 데이터 변경 빈도·조회 빈도·정확성 요구와 함께 비교한다.

앱에서 쓰기 경로가 늘수록 중복 값의 동기화 누락 위험도 늘어난다. “현재는 한 API만 수정한다”는 가정이 관리 도구·배치·추가 서비스 이후에도 유지되는지 검토해야 한다. 성능 최적화가 원본 사실의 위치를 모호하게 만들지 않도록 한다.

## 14. 흔한 오해와 검토 사례

| 주장 | 검토할 내용 |
| --- | --- |
| 값이 반복되면 모두 잘못된 중복이다 | 동일한 사실의 반복인지, 다른 대상에 우연히 같은 값인지 확인 |
| PK를 하나 추가하면 정규화가 끝난다 | 기존 후보 키와 FD가 그대로 남는지 확인 |
| 문자열은 글자까지 나눠야 원자적이다 | 모델에서 독립적으로 다룰 값의 단위를 확인 |
| 나눠서 행이 줄었으므로 무손실이다 | 다시 결합했을 때 원래 없던 조합이 생기지 않는지 확인 |
| 정규화하면 업무 규칙도 모두 자동 보장된다 | 실제 DDL과 쓰기 로직에서 무엇을 검사하는지 확인 |
| 이력 값도 현재 원본과 같게 유지해야 한다 | 그 값이 어느 시점의 사실을 표현하는지 확인 |

정규화 설명에서 가장 중요한 결과물은 “몇 정규형인가”라는 이름 하나보다 **왜 이 속성을 이 테이블에 보관하는가**에 대한 근거다. 설계를 바꾸면 그 근거를 다시 적용할 수 있어야 한다.

## 15. 핵심 정리

<callout icon="✅" color="green_bg">
정규화는 함수 종속성과 후보 키를 근거로 사실의 저장 위치를 정리하는 과정이다. 분해한 뒤에도 원래 사실과 지켜야 할 규칙을 보존해야 한다.
</callout>

- 갱신·삽입·삭제 이상은 서로 다른 사실을 한 구조에 묶었을 때 생길 수 있다.
- FD는 표본에서 우연히 관찰한 상관관계와 다르며 방향과 시점이 중요하다.
- 2NF는 모든 후보 키에 대한 부분 종속성을 검토한다.
- 3NF는 형식적 조건을 갖고 있으며 비키 속성 간 종속 제거라는 입문 설명의 한계를 안다.
- 무손실성과 종속성 보존은 구분하여 확인한다.
- 역사적 사실을 보존하는 값과 원본의 파생 복사본은 갱신 책임이 다르다.

## 출처

- [Database System Concepts 7장 — Relational Database Design](https://db-book.com/slides-dir/PDF-dir/ch7.pdf): FD, 키, 정규형, 분해와 시간에 따른 모델링.
- [Microsoft — Database Normalization Basics](https://learn.microsoft.com/en-us/previous-versions/troubleshoot/microsoft-365/microsoft-365-apps/access/database-normalization-description): 반복 그룹과 기초 정규화의 목적.
- [Juniata — Normalization](https://jcsites.juniata.edu/faculty/rhodes/dbms/normalization.htm): 완전·부분·이행 종속과 2NF·3NF의 조건.
- [Berkeley CS186 — DB Design](https://cs186berkeley.net/notes/note13/): FD에 따른 분해, 무손실성과 종속성 보존.
- [PostgreSQL 18 — Constraints](https://www.postgresql.org/docs/18/ddl-constraints.html): 예제의 PK·FK와 무결성 검사.
- [PostgreSQL 18 — Generated Columns](https://www.postgresql.org/docs/18/ddl-generated-columns.html), [Views](https://www.postgresql.org/docs/18/tutorial-views.html), [Materialized Views](https://www.postgresql.org/docs/18/rules-materializedviews.html): 계산 결과와 파생 조회 구조의 관리 차이.

## 16. 이해도 확인 질문

1. 한 행에 학생·강좌·부서·강사의 현재 정보를 모두 저장하면 어떤 갱신·삽입·삭제 이상이 생기는가? 각각 실제 변경 예를 들어라.
2. 샘플에서 학생 이름이 모두 다를 때 이름을 후보 키로 결론 내릴 수 있는가? FD의 방향과 모든 적법한 상태라는 조건을 이용해 설명하라.
3. 수강 신청에 대리 키를 추가해도 `(student_id, class_id)`가 후보 키로 남는다면 부분 종속성 검토가 왜 필요한가? 2NF에서 3NF로 나누는 단계도 설명하라.
4. 학습자·과목·강사 표를 두 개로 나눈 뒤 원래 없던 조합이 생겼다. 무슨 정보가 사라졌으며 무손실 분해와 종속성 보존은 무엇이 다른가?
5. 주문 당시 단가, 현재 상품 가격, 미리 계산해 보관한 월별 매출은 각각 어떤 사실인가? 원본 위치와 갱신 책임을 어떻게 구분할 것인가?
