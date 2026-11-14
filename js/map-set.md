---
track: A
notion: https://app.notion.com/p/415178c4afac811b8577e5409e00b5f1
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · Map / Set

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 키·값 저장소와 중복 없는 집합의 차이를 설명한다.
- Map의 키 비교와 Set의 중복 판정 결과를 예측한다.
- 일반 객체·배열·Map·Set을 요구에 맞게 선택한다.
- 존재 확인, 객체 식별성, 직렬화 오류를 디버깅한다.

## 2. 이 주제를 왜 배우는가

사용자별 방문 횟수는 키로 값을 찾는 문제이고, 선택한 항목의 중복 제거는 값의 존재를 확인하는 문제다. 배열을 매번 검색하거나 일반 객체를 억지로 사전처럼 쓰기보다 `Map`과 `Set`으로 의도를 표현할 수 있다.

둘 다 표준 JavaScript의 컬렉션이다. DOM이나 서버 API가 없어도 사용할 수 있으며, 저장되는 객체 자체를 복제하지는 않는다.

### 동일한 항목의 의미부터 결정한다

“중복 사용자 제거”에서 같은 항목은 같은 객체인가, 같은 ID인가, 같은 이메일인가? Set은 언어가 정한 값 비교로 중복을 판단하므로 이 업무 의미를 대신 정해 주지 않는다. 저장소 종류를 고르기 전에 동일성 기준을 적어야 한다.

Map도 마찬가지다. 조회할 때 매번 새 객체를 만들어 키로 사용하면 내용이 같아도 저장한 값을 찾지 못한다. 객체 자체의 수명에 연결된 정보를 저장할지, 재생성해도 유지되는 식별자로 저장할지부터 결정한다.

## 3. Map: 키에서 값으로 연결하기

**Map**은 키와 값의 쌍을 보관한다. 키에는 문자열뿐 아니라 숫자·객체 등 모든 값을 사용할 수 있다. `set`은 추가·갱신, `get`은 조회, `has`는 존재 확인, `delete`는 제거, `size`는 항목 수다.

```javascript
const visits = new Map();
visits.set("Kim", 1);
visits.set("Lee", 2);
visits.set("Kim", 3);
console.log(visits.size, visits.get("Kim")); // 2 3
console.log(visits.has("Park"), visits.get("Park")); // false undefined
console.log(visits.delete("Lee"), visits.size); // true 1
```

같은 키의 갱신은 항목 수를 늘리지 않는다. 없는 키의 `get`은 `undefined`이므로, 실제로 `undefined`를 저장한 경우와 구분할 때는 `has`를 쓴다. `map.key = value`는 Map 항목을 추가하는 방식이 아니다.

### 존재 여부와 집계 값을 나누어 처리한다

```javascript
const counts = new Map();
for (const name of ["Kim", "Lee", "Kim"]) {
  const previous = counts.has(name) ? counts.get(name) : 0;
  counts.set(name, previous + 1);
}
console.log(counts.get("Kim"), counts.get("Lee")); // 2 1
counts.set("unknown", undefined);
console.log(counts.has("unknown"), counts.get("unknown")); // true undefined
```

집계에서는 키를 찾는 과정, 기본 횟수를 정하는 과정, 새 횟수를 저장하는 과정이 분리된다. 이 예제는 없는 키와 값이 undefined인 키가 서로 다르다는 사실도 보여 준다. 캐시에 undefined를 정상 결과로 저장할 수 있다면 get 결과만 보고 미조회로 판단하면 같은 작업을 계속 수행할 수 있다.

`size`도 일반 객체의 키 개수를 매번 계산한 값과는 별도의 컬렉션 API다. 항목 조작은 set·get·delete로 일관되게 수행해야 보통 속성과 Map 항목이 뒤섞이지 않는다.

## 4. Set: 값의 존재를 저장하기

**Set**은 같은 값의 중복을 허용하지 않는 컬렉션이다. `add`로 넣고 `has`로 확인하며 `delete`로 제거한다. 값마다 별도의 개수나 부가 정보를 저장하려면 Map이 필요하다.

```javascript
const selected = new Set(["a", "b", "a"]);
selected.add("c");
console.log(selected.size); // 3
console.log([...selected].join(",")); // a,b,c
selected.delete("b");
console.log(selected.has("b")); // false
```

배열을 Set으로 만든 뒤 다시 펼치면 원시 값 목록의 중복을 제거할 수 있다. 단, “내용이 같은 객체”까지 하나로 합친다고 기대하면 안 된다.

### Set을 사용할 때 잃는 정보

```javascript
const values = ["A", "B", "A", "C"];
const unique = new Set(values);
console.log(unique.size, unique.has("A")); // 3 true
console.log([...unique].join(",")); // A,B,C
```

Set은 중복 횟수를 보관하지 않는다. 위 결과만으로 A가 두 번 등장했다는 사실을 복원할 수 없다. 선택 여부에는 알맞지만 투표 수·재고 수량처럼 중복의 개수가 의미 있는 문제에는 Map 등의 집계가 필요하다.

Set에 넣기 전에 대소문자·공백을 정규화하면 어떤 값들을 같게 볼지 바뀐다. 이것은 Set의 비교 규칙을 바꾸는 것이 아니라 입력을 바꾸는 설계다. 원래 표시값도 필요하다면 정규화 키와 원본 표시값을 함께 관리한다.

## 5. 키와 중복을 비교하는 규칙

Map의 키와 Set의 값은 **SameValueZero** 규칙으로 비교한다. 기본적으로 `===`와 비슷하지만 `NaN`은 자기 자신과 같은 것으로 보고, `+0`과 `-0`도 같다. 객체는 내용이 아니라 식별성으로 비교한다. [MDN의 비교 규칙](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections#key_and_value_equality_of_map_and_set) 참조.

```javascript
const a = { id: 1 };
const b = { id: 1 };
const records = new Map([[a, "first"]]);
console.log(records.get(a)); // first
console.log(records.get(b)); // undefined
const values = new Set([a, a, b, NaN, NaN, 0, -0]);
console.log(values.size); // 4
```

`a`가 두 번 들어가도 같은 객체이므로 하나다. `b`는 새 객체라 별개다. 두 `NaN`은 하나, 두 0도 하나다. 따라서 총 네 값이다. API에서 매번 새로 만들어 받은 객체의 중복을 ID 기준으로 제거하려면 객체 자체 대신 `id`를 키로 삼는 등 업무상 동일성 기준을 정해야 한다.

### 객체를 키로 쓰는 경우의 실행 순서

```javascript
const key = { id: 1 };
const metadata = new Map([[key, "saved"]]);
key.id = 2;
console.log(metadata.get(key)); // saved
console.log(metadata.get({ id: 2 })); // undefined
console.log(new Map([[1, "number"], ["1", "string"]]).size); // 2
```

키 객체 내부가 바뀌어도 그 객체의 식별성은 유지된다. 따라서 Map은 같은 키로 조회한다. 반면 새 객체의 필드가 같아도 별도 객체다. 숫자 1과 문자열 "1"도 서로 다른 값이므로 Map에서 구분된다.

일반 객체의 숫자 속성 키는 문자열로 변환될 수 있어 같은 설계가 다른 결과를 낼 수 있다. Map의 장점은 모든 키를 문자열로 바꾸지 않고 언어 값의 구분을 유지한다는 데 있다. [ECMAScript Map Objects](https://tc39.es/ecma262/multipage/keyed-collections.html#sec-map-objects)의 키 비교 계약을 기준으로 해석한다.

## 6. 삽입 순서와 순회의 실행 흐름

```javascript
const scores = new Map([["b", 2], ["a", 1]]);
scores.set("b", 20);
for (const [key, value] of scores) {
  console.log(key, value);
}
scores.delete("b");
scores.set("b", 30);
console.log([...scores.keys()].join(",")); // a,b
```

첫 반복은 `b 20`, `a 1` 순서다. 기존 키 갱신은 처음 삽입 위치를 유지하고, 삭제 후 재삽입하면 뒤로 간다. Map·Set은 삽입 순서로 순회하지만 키의 정렬 순서를 자동 유지하지 않는다.

순회 도중 컬렉션을 변경하면 남은 순회 대상에 영향을 줄 수 있다. 안정된 한 시점의 목록이 필요하면 별도의 배열을 만들어 순회할지 판단한다.

### 순회는 자동 스냅샷이 아니다

Map·Set의 반복 중 추가와 삭제는 남은 방문 대상에 영향을 줄 수 있다. “반복 시작 때의 항목만 보겠다”는 요구라면 그때 엔트리 배열을 만들어 순회할지 정한다. 다만 배열로 만든 엔트리 안의 객체까지 깊게 복사되는 것은 아니다.

삽입 순서가 보존된다는 계약도 시간순 로그와 같지는 않다. 기존 키의 값 갱신은 처음 위치를 유지한다. 최신 갱신 항목을 뒤에 놓고 싶다면 삭제 후 재삽입 같은 정책을 별도로 선택해야 하며, 그 정책이 다른 순회 코드에 주는 영향도 검토한다.

## 7. 선택 기준과 변환

| 요구 | 자연스러운 구조 | 이유 |
| --- | --- | --- |
| 이름·나이 등 고정된 레코드 | 일반 객체 | 알려진 필드를 속성으로 표현 |
| 위치·중복·순서가 있는 목록 | 배열 | 인덱스와 목록 변환 |
| 동적으로 늘어나는 키별 집계 | Map | 다양한 키·존재 확인·항목 수 |
| 중복 없는 선택 목록 | Set | 값의 존재를 직접 표현 |

Map·Set의 조회가 어떤 엔진에서도 반드시 해시 테이블의 O(1)이라고 명세가 요구하지는 않는다. 명세는 평균적으로 요소 수보다 낮은 차수의 접근 시간을 요구하지만 내부 자료 구조까지 고정하지 않는다.

`JSON.stringify(new Map(...))`나 Set의 결과를 저장하면 기본적으로 항목이 기대대로 직렬화되지 않는다. Map은 `Array.from(map)` 같은 엔트리 배열로 바꾸고, 복원 시 `new Map(entries)`를 사용하는 등 저장 형식을 명시한다. 객체 키가 있는 Map을 JSON으로 왕복하면 객체 식별성도 별도로 고려해야 한다.

### 저장과 복원은 별도의 데이터 형식이다

```javascript
const original = new Map([["a", 1], ["b", 2]]);
const text = JSON.stringify([...original]);
const restored = new Map(JSON.parse(text));
console.log(restored.get("a"), restored.size); // 1 2
console.log(JSON.stringify(new Set([1, 2]))); // {}
```

이 예제는 문자열 키와 숫자 값으로 제한된 저장 계약이다. 객체 키를 JSON으로 바꾸어 복원하면 새 객체가 만들어지므로 원래 키 객체로 조회할 수 없다. Date·undefined 등 값의 표현도 JSON 규칙의 영향을 받는다.

컬렉션을 파일에 저장한다면 항목 형식, 키 타입, 허용 값, 중복 키가 입력에 있을 때의 정책까지 정해야 한다. 런타임 자료 구조를 정한 것만으로 영속 형식까지 결정되는 것은 아니다.

## 8. 흔한 오해·예외·주의점

- `new Set([{ id: 1 }, { id: 1 }])`은 내용 기준 중복 제거가 아니다.
- Map에서 `get(key) === undefined`는 키 부재의 충분한 증거가 아니다.
- `map["x"] = 1`과 `map.set("x", 1)`은 다른 저장 공간을 사용한다.
- 삽입 순서를 유지한다고 키를 정렬해 주지는 않는다.
- Map에 객체를 넣으면 컬렉션이 그 객체를 계속 참조한다. 캐시 항목의 제거 정책 없이 데이터를 무한히 축적하지 않는다.

### 캐시의 성능과 수명은 함께 본다

빠른 조회를 위해 Map을 캐시로 사용해도 항목을 계속 추가하면 보관 메모리가 늘어난다. 한 번의 get 비용만 줄였다고 전체 성능 문제가 해결된 것은 아니다. 최대 항목 수, 제거 조건, 데이터의 유효 기간이 필요할 수 있다.

Map이 객체 키를 참조한다는 점은 객체 수명과도 연결된다. 더 이상 사용할 항목을 제거할 책임을 정하고, 자동 수거 시점에 의존해 업무 처리를 예약하지 않는다.

## 9. 실무 적용과 디버깅 관점

카운터를 구현할 때 현재 값이 없으면 0으로 시작하고, 이후 합산한다. 키 조회가 실패하면 키의 값·타입·식별성을 확인한다. 화면에는 같은 ID로 보이더라도 원래 키로 넣은 객체와 새 응답 객체는 다를 수 있다.

컬렉션 종류를 바꿀 때는 성능 기대만 보지 말고 저장·복원·중복 정의·순회 순서를 함께 적는다. 데이터 모델과 접근 패턴이 단순해지면 버그 원인도 좁히기 쉽다.

### 실패한 조회를 재현하는 절차

첫째, 저장과 조회 시점의 키를 원시 값이면 값·타입으로, 객체면 식별성 비교로 확인한다. 둘째, 키를 직렬화하거나 다시 생성하는 경로가 있는지 찾는다. 셋째, 삭제·clear·재할당으로 다른 Map을 읽고 있지 않은지 검사한다.

ID 기준 중복 제거를 구현한다면 Map에 `id → record`를 저장한 뒤 중복 ID를 처음 값으로 유지할지 마지막 값으로 갱신할지 정한다. 이 결정이 없으면 자료 구조는 정확히 동작해도 화면에는 기대와 다른 데이터가 남을 수 있다.

### 통합 사례: ID별 목록과 선택 상태

같은 사용자가 여러 번 수집되는 목록에서는 “마지막에 받은 레코드 유지”와 “선택 여부 유지”를 별도로 요구할 수 있다. 객체 자체를 Set에 넣으면 응답마다 새 객체가 들어와 중복 제거 기준이 맞지 않을 수 있다.

```javascript
const incoming = [{ id: 1, name: "old" }, { id: 2, name: "Lee" }, { id: 1, name: "new" }];
const byId = new Map();
for (const user of incoming) byId.set(user.id, user);
const selectedIds = new Set([1]);
console.log(byId.size, byId.get(1).name); // 2 new
console.log([...byId.keys()].join(",")); // 1,2
console.log(selectedIds.has(byId.get(1).id)); // true
```

Map은 같은 ID의 값을 갱신하므로 마지막 레코드가 남는다. 그러나 키를 갱신해도 처음 삽입 위치를 유지하므로 순서는 1,2다. “최신 갱신 항목을 뒤로 이동”이 요구라면 별도 순서 정책이 필요하다.

선택 상태는 ID를 Set에 저장했으므로 레코드 객체를 새로 받아도 유지된다. 다만 숫자 1과 문자열 "1"은 다른 키이므로 입력 경계에서 ID 타입을 일관되게 정해야 한다. 자료 구조가 중복과 조회를 처리하더라도 동일성 기준·값 갱신 정책·표시 순서는 업무 코드가 정한다. 이 세 결정을 분리하면 객체 식별성 문제를 자료 구조의 고장으로 오해하지 않는다.

## 10. 핵심 정리

Map은 키와 값을 연결하고 Set은 값의 존재를 저장한다. 비교 규칙과 객체 식별성을 이해해야 조회·중복 제거가 의도대로 동작한다. 직렬화 형식과 삭제 정책까지 컬렉션 사용의 일부로 설계한다.

## 출처

- [MDN — Keyed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections): Map·Set API와 키·값 비교.
- [ECMAScript — Keyed Collections](https://tc39.es/ecma262/multipage/keyed-collections.html): 순서와 평균 접근 시간 요구.

- [MDN — Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map): 키 식별성·객체와의 차이·직렬화 고려.

## 11. 이해도 확인 질문

1. 같은 내용의 객체 두 개를 Map의 키로 넣으면 같은 항목이 되는가? 어떤 기준으로 판단하는가?
2. Map에 `undefined`를 저장한 키와 없는 키를 구분하는 코드를 설계하라.
3. 사용자 목록을 ID 기준으로 중복 제거하려면 Set에 무엇을 넣거나 Map을 어떻게 사용해야 하는가?
4. 기존 키 갱신과 삭제 후 재삽입은 순회 순서에 어떤 차이를 만드는가?
5. Map을 JSON으로 저장하려는 코드에서 저장·복원 양쪽에 무엇을 명시해야 하는가?
