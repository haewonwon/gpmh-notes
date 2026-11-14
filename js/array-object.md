---
track: A
notion: https://app.notion.com/p/415178c4afac8198ab19eb65e0e987bd
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · Array / Object

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 객체의 속성과 배열의 인덱스·길이를 구분한다.
- 조회·추가·변경·삭제의 결과를 예측한다.
- 반복과 변환에서 순서, 원본 변경, 반환값을 추적한다.
- 누락된 속성·배열 구멍·정렬 오류를 디버깅한다.

## 2. 이 주제를 왜 배우는가

사용자 한 명은 이름과 나이처럼 서로 다른 성격의 속성을 갖고, 사용자 목록은 여러 항목의 순서를 갖는다. 전자에는 객체, 후자에는 배열이 자연스럽다. 둘 다 객체라는 이유로 같은 방식으로 처리하면 목록의 길이와 실제 항목 수를 혼동하거나, 정렬 중 원본 데이터를 바꿀 수 있다.

자료를 어떤 구조에 담는지는 이후 읽기·갱신·순회의 기준을 정한다. 문법을 외우기보다 “이 값은 이름으로 찾는가, 위치로 찾는가”부터 판단한다.

### 레코드와 목록은 접근 방식이 다르다

사용자 한 명을 `["Kim", 20, true]`로 표현하면 각 위치가 무엇인지 모든 코드가 기억해야 한다. `{ name: "Kim", age: 20, active: true }`는 의미를 키로 드러낸다. 반대로 사용자 여러 명을 키 이름만 계속 늘린 객체로 표현하면 목록 순회와 순서 변경을 직접 관리해야 한다.

자료 구조를 고를 때는 중복 허용·순서·접근 키·갱신 방식부터 적는다. 배열도 객체지만, 이 언어상의 공통점이 두 구조의 사용 목적까지 같게 만들지는 않는다.

## 3. 객체: 이름을 가진 속성의 모음

**속성(property)** 은 키와 값의 연결이다. 일반적인 객체 키는 문자열 또는 Symbol이다. 숫자를 대괄호에 사용하면 보통 문자열 키로 변환된다. 객체 리터럴 `{ ... }`은 새 객체를 만든다.

```javascript
const user = { name: "Kim", age: 20 };
const field = "name";
console.log(user.name, user[field]); // Kim Kim
user.age = 21;
user.role = "reader";
delete user.role;
console.log(user.age, user.role); // 21 undefined
```

점 표기 `user.field`는 글자 그대로 `field`라는 키를 찾는다. 대괄호 `user[field]`는 변수의 값을 키로 사용한다. 외부에서 선택한 항목을 조회할 때 둘을 혼동하면 `undefined`가 생긴다.

속성이 없는 경우와 속성이 있지만 값이 `undefined`인 경우는 다르다. 현대 환경에서는 `Object.hasOwn(object, key)`로 객체 자신의 속성 존재를 확인할 수 있다.

### 값 읽기와 속성 존재 확인

```javascript
const item = { value: undefined, count: 0 };
console.log(item.value, item.missing); // undefined undefined
console.log(Object.hasOwn(item, "value"), Object.hasOwn(item, "missing")); // true false
console.log(Object.keys(item).join(",")); // value,count
```

속성 읽기 결과만으로는 명시적인 `undefined`와 누락을 구분하지 못한다. 입력 스키마에서 “필드 생략은 유지, 필드 제공은 갱신”이라는 의미를 사용한다면 존재 검사와 값 검사를 나누어야 한다. `if (item.count)`는 0도 거짓으로 취급하므로 수량 존재 검사가 되지 않는다.

객체 속성에는 값뿐 아니라 열거 가능 여부 같은 성질이 있다. 따라서 `Object.keys`가 모든 종류의 내부 정보를 보여 주는 것은 아니다. 현재 단계에서는 일반적인 리터럴의 필드를 중심으로 보되, 키 목록과 속성 읽기 결과가 항상 같은 정보라고 생각하지 않는다.

## 4. 배열: 인덱스와 length

**배열(Array)** 은 위치 기반 목록을 위한 객체다. 인덱스는 0부터 시작한다. `length`는 항목을 직접 세어 저장한 개수라기보다 배열 인덱스 범위와 연결된 특별한 속성이다.

```javascript
const items = ["pen", "book"];
console.log(items[0], items.length); // pen 2
items.push("bag");
console.log(items.pop(), items.length); // bag 2
items[5] = "cup";
console.log(items.length, items[3]); // 6 undefined
console.log(Object.hasOwn(items, 3)); // false
```

인덱스 5에 쓰면 `length`가 6이 되지만 중간 2~4에 실제 속성이 생기지는 않는다. 이런 빈 위치를 **구멍(hole)** 이라고 한다. `undefined`라는 값이 들어 있는 요소와도 구분한다. 일반적인 목록에서는 연속된 요소를 유지하는 편이 다루기 쉽다.

`delete items[0]`은 뒤 요소를 당겨 채우지 않는다. 요소를 제거하고 이후 위치를 이동하려면 `splice` 같은 목록 연산을 선택한다.

### length 변경과 실제 항목 수

```javascript
const list = ["a", "b", "c"];
list.length = 1;
console.log(list.join(","), Object.hasOwn(list, 2)); // a false
list.length = 3;
console.log(list.length, Object.hasOwn(list, 2)); // 3 false
```

길이를 줄이면 범위 밖 요소가 제거된다. 다시 늘린다고 이전 값이 복원되지는 않으며 새 범위에 구멍이 생긴다. `length`를 직접 조정하는 코드가 있으면 “배열을 확장했으니 기본값이 채워졌다”는 가정을 버려야 한다.

배열에 `list.label = "items"`처럼 일반 속성을 붙여도 그 속성이 목록 요소나 길이에 자동으로 포함되지 않는다. 목록 데이터와 부가 정보를 함께 보관하려면 `{ items: [...], label: ... }`처럼 역할을 분리하는 편이 읽기 쉽다.

## 5. 순회·변환·선택의 실행 흐름

```javascript
const products = [
  { name: "pen", price: 1000 },
  { name: "book", price: 5000 },
  { name: "cup", price: 3000 }
];
const names = products.map(product => product.name);
const cheap = products.filter(product => product.price <= 3000);
const found = products.find(product => product.name === "book");
console.log(names.join(",")); // pen,book,cup
console.log(cheap.length, found.price); // 2 5000
```

`map`은 각 요소를 변환한 결과로 새 배열을 만든다. `filter`는 조건을 통과한 요소로 새 배열을 만들고, `find`는 처음 일치한 요소 또는 `undefined`를 반환한다. 새 배열을 만들었다고 내부 객체가 자동 복제되지는 않는다.

이 예제의 `product => ...`는 각 요소를 받아 결과를 계산하는 함수다. 각 API가 이 함수를 언제 어떤 인자로 부르는지 확인해야 한다. `map`에서 반환을 빠뜨리면 새 배열에 `undefined`가 들어갈 수 있다.

### 구멍과 undefined는 순회에서도 다르다

```javascript
const sparse = [1, , 3];
let calls = 0;
const mapped = sparse.map(value => { calls += 1; return value * 2; });
console.log(calls, mapped.length); // 2 3
console.log(Object.hasOwn(mapped, 1)); // false
console.log([...sparse].join("|")); // 1||3
```

`map`은 없는 요소를 콜백에 전달하는 대신 그 위치를 건너뛰고 결과에도 빈 위치를 남긴다. 배열을 펼칠 때의 반복은 빈 위치를 읽은 `undefined`를 값으로 전달할 수 있다. API마다 구멍을 다루는 방식이 다르므로 같은 길이라고 같은 처리 횟수가 보장되지는 않는다.

[ECMAScript의 map 알고리즘](https://tc39.es/ecma262/multipage/indexed-collections.html#sec-array.prototype.map)은 각 위치의 속성 존재를 확인한 뒤 콜백을 호출한다. 구멍의 차이는 이 존재 확인 과정에서 나온다. 보통의 업무 목록에서는 데이터 생성 지점에서 연속 배열을 유지해 이런 예외를 줄인다.

## 6. 변경하는 메서드와 새 값을 만드는 메서드

| 목적 | 원본을 바꾸는 예 | 새 배열을 만드는 예 |
| --- | --- | --- |
| 정렬 | `sort` | `toSorted` |
| 뒤집기 | `reverse` | `toReversed` |
| 일부 삭제·교체 | `splice` | `toSpliced` |
| 일부 읽기 | 해당 없음 | `slice` |

`toSorted` 등은 ES2023 API이므로 실행 환경의 지원을 확인한다. 숫자 정렬에는 비교 함수를 제공한다.

```javascript
const scores = [2, 10, 3];
scores.sort();
console.log(scores.join(",")); // 10,2,3
scores.sort((a, b) => a - b);
console.log(scores.join(",")); // 2,3,10
```

기본 정렬은 숫자의 크기 순서를 의도한 비교가 아니다. 비교 함수가 음수면 앞 값, 양수면 뒤 값이 먼저 오도록 한다. 위에서는 첫 `sort`가 원본을 바꾸고 두 번째 `sort`가 같은 배열을 다시 바꾼다.

### 정렬 함수는 비교 규칙과 효과를 함께 갖는다

```javascript
const rows = [{ score: 2 }, { score: 1 }];
const sorted = rows.slice().sort((a, b) => a.score - b.score);
console.log(rows[0].score, sorted[0].score); // 2 1
sorted[0].score = 99;
console.log(rows[1].score); // 99
```

`slice`로 만든 바깥 배열은 독립되어 원본의 순서는 유지된다. 그러나 요소 객체는 공유하므로 정렬 결과에서 객체를 변경하면 원본에서도 보인다. 순서 분리와 요소 분리는 다른 계약이다.

비교 함수는 두 값을 일관되게 비교해야 한다. 비교 중 원본을 바꾸거나 비교 결과를 무작위로 반환하면 정렬이 전제하는 순서 관계가 무너진다. 같은 점수의 상대 순서도 업무적으로 중요하다면 안정성 요구와 추가 비교 키를 명시한다. 정렬 결과를 확인할 때는 값뿐 아니라 원본 순서가 바뀌어도 되는지도 검사한다.

## 7. 객체 순회와 누락된 데이터

```javascript
const record = { title: "JS", pages: undefined };
console.log(Object.keys(record).join(",")); // title,pages
console.log(Object.hasOwn(record, "pages")); // true
console.log(Object.hasOwn(record, "author")); // false
for (const [key, value] of Object.entries(record)) {
  console.log(key, value);
}
```

마지막 반복은 `title JS`, `pages undefined`를 출력한다. `Object.keys`·`values`·`entries`는 자신의 열거 가능한 문자열 키 속성을 기준으로 한다. 모든 종류의 속성을 가져오는 만능 조회가 아니다.

배열 값은 `for...of`로 순회한다. `for...in`은 키를 순회하므로 배열 요소 순회와 혼동하지 않는다. 정수 모양 키를 포함한 객체의 열거 순서는 단순히 모든 키의 삽입 순서와 같다고 가정하지 않는다.

### 객체 열거 순서에 업무 순서를 맡기지 않는다

```javascript
const record = { b: 1, "10": "ten", "2": "two", a: 2 };
console.log(Object.keys(record).join(",")); // 2,10,b,a
```

일반 객체의 키 순회에는 배열 인덱스 모양의 문자열 키를 먼저 다루는 등의 규칙이 있다. 전체가 작성 순서대로 나오는 것으로 기억하면 숫자 모양 ID가 들어왔을 때 순서가 바뀌어 보인다. 화면의 명시적 순서가 필요하면 배열이나 별도 순서 필드를 사용한다.

또한 `Object.entries`가 만드는 것은 키·값 쌍의 배열이며 내부 객체 값의 깊은 복사는 아니다. 열거·배열 변환·복제를 같은 작업으로 섞지 않는다.

## 8. 흔한 오해·예외·주의점

- `typeof []`는 `"object"`다. 배열 검사는 `Array.isArray`를 쓴다.
- 배열 범위를 벗어나 읽어도 항상 예외가 나지는 않는다. `undefined`를 받은 뒤 속성에 접근하는 지점에서 실패할 수 있다.
- `slice`와 `splice`는 이름이 비슷하지만 원본 변경 여부가 다르다.
- `map`·`filter`가 반환한 배열 속 객체를 바꾸면 원본 목록의 같은 객체에서도 변경이 보일 수 있다.

### 반환값을 사용하지 않는 변환

`map`은 결과 배열을 얻기 위한 API다. 출력이나 외부 저장만 하면서 결과를 버리면 읽는 사람은 새 목록이 필요한 줄로 오해할 수 있다. 반대로 `forEach`의 반환값을 새 배열처럼 사용하면 `undefined`를 받는다. 메서드를 선택할 때 “어떤 요소를 방문하는가” 외에 “무엇을 반환하는가”도 확인한다.

반복 중 배열에 요소를 추가·삭제하면 현재 위치와 이후 방문 대상의 관계가 복잡해진다. 가능한 경우 입력을 읽어 결과 배열을 따로 구성하거나, 제자리 변경이 필요한 이유와 인덱스 이동 규칙을 명확히 정한다.

## 9. 실무 적용과 디버깅 관점

변환 전후의 배열 식별성, 길이, 요소 식별성을 따로 검사한다. 목록이 갑자기 재정렬되면 `sort`·`reverse` 호출을 찾고, 누락된 요소가 생기면 잘못된 인덱스 대입과 `delete`를 확인한다.

**[미리보기: Shallow Copy / Deep Copy]** 새 목록과 그 안의 중첩 객체는 서로 다른 수준의 공유 문제다. 내부 객체까지 복사할 범위는 JavaScript Core의 Shallow Copy / Deep Copy · Immutability에서 다룬다.

### 사라진 항목을 추적하는 순서

목록의 길이는 맞는데 일부 처리가 빠졌다면 `length`와 실제 키를 나란히 출력한다. 다음으로 목록을 만드는 코드에서 큰 인덱스 대입·길이 확장·delete 사용을 찾는다. 변환 결과에 빈 위치가 유지되는지, 명시적 undefined가 생기는지 확인하면 생성 오류와 반환 누락을 구분할 수 있다.

원본 순서가 바뀌었다면 가장 가까운 `sort`만 찾지 말고 공유된 별칭도 추적한다. 어떤 함수가 같은 배열을 받아 정렬했을 수 있다. 갱신 함수의 계약에 원본 변경 여부를 적어 두면 조회 전용 코드가 실수로 정렬하는 문제를 줄일 수 있다.

### 통합 사례: 목록 순서는 보존하고 한 항목만 갱신하기

상품 목록을 정렬해 보여 주고 선택한 상품의 재고만 바꾸는 기능을 생각하자. 요구는 원래 목록 순서를 유지하고, 원본 상품 데이터도 직접 바꾸지 않는 것이다. 정렬과 항목 갱신을 한 작업으로 뭉치면 어느 수준의 복사가 필요한지 놓치기 쉽다.

```javascript
const original = [{ id: 1, stock: 2 }, { id: 2, stock: 1 }];
const displayed = original.slice().sort((a, b) => a.stock - b.stock);
const updated = displayed.map(item =>
  item.id === 2 ? { id: item.id, stock: item.stock + 1 } : item
);
console.log(original.map(item => item.id).join(",")); // 1,2
console.log(updated.map(item => item.id).join(",")); // 2,1
console.log(original[1].stock, updated[0].stock); // 1 2
```

실행을 세 단계로 나눈다. 첫째 slice가 순서를 바꿀 새 바깥 배열을 만든다. 둘째 sort는 그 배열만 정렬하며 내부 상품 객체는 여전히 공유한다. 셋째 map은 수정 대상 상품만 새 객체로 만들어 원본의 stock을 보존한다.

검증도 세 단계와 대응한다. 원본 배열의 ID 순서, 표시 배열의 ID 순서, 수정 전후의 재고를 따로 확인한다. 정렬된 화면이 맞다는 확인 하나로 원본 보존까지 검증할 수 없고, 새 배열이라는 확인 하나로 요소 변경까지 안전하다고 결론 내릴 수 없다. 이 사례에서 사용한 얕은 구조 분리는 뒤의 복사·불변성 문서에서 일반적인 중첩 경로로 확장한다.

## 10. 핵심 정리

객체는 속성 이름으로, 배열은 위치와 순서로 데이터를 다룬다. 메서드의 반환값과 원본 변경 여부를 함께 읽어야 한다. 새 배열과 새 요소 객체는 같은 의미가 아니다.

## 출처

- [MDN — Indexed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections): 배열 인덱스·길이·메서드.
- [MDN — Working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects): 속성 조회와 열거.

- [ECMAScript — Indexed Collections](https://tc39.es/ecma262/multipage/indexed-collections.html): 배열 메서드·구멍·반복의 명세.

## 11. 이해도 확인 질문

1. 길이가 2인 배열의 인덱스 7에 값을 넣으면 길이와 중간 요소는 어떻게 되는가?
2. `const key = "name"`일 때 `user.key`와 `user[key]`의 차이를 설명하라.
3. 숫자 목록 `[30, 4, 100]`의 기본 정렬에 어떤 문제가 생기는가?
4. `filter`로 선택한 객체를 수정했더니 원본 목록에도 반영됐다. 실행 흐름을 설명하라.
5. 값이 `undefined`인 속성과 존재하지 않는 속성을 어떻게 구분할 수 있는가?
