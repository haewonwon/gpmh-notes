---
track: A
notion: https://app.notion.com/p/415178c4afac8112a460d464c3c926d7
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · Destructuring · Spread / Rest

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 구조 분해, 펼치기, 나머지 수집의 방향을 구분한다.
- 이름 변경·기본값·중첩 패턴의 결과를 예측한다.
- 배열과 객체에서 `...`의 처리 대상을 비교한다.
- 옵션 병합 순서와 중첩 데이터 공유 버그를 찾는다.

## 2. 이 주제를 왜 배우는가

함수에 넘길 옵션을 합치거나 응답 객체에서 필요한 필드만 꺼내는 코드는 자주 반복된다. 매번 긴 속성 경로를 쓰는 대신 구조 분해를 사용할 수 있고, 값을 새 목록·객체에 펼쳐 넣을 수도 있다. 다만 문법이 짧아지면서 무엇을 읽고 무엇을 새로 만드는지가 숨겨지기 쉽다.

세 문법은 함께 쓰이지만 역할이 다르다. **구조 분해(destructuring)** 는 구조에서 값을 꺼내 이름에 연결하고, **펼치기(spread)** 는 여러 위치에 값을 공급하며, **나머지(rest)** 는 남은 값을 하나로 모은다.

### 문법을 줄여도 데이터 계약은 남는다

`const { user: { name } } = response`는 간결하지만 response와 user가 존재한다는 전제를 갖는다. 잘못된 응답이 오면 어떤 단계의 구조가 없었는지부터 확인해야 한다. 축약 문법은 검증을 대신하지 않는다.

반대로 매번 `response.user.name`을 반복해서 쓰면 속성 경로를 잘못 적기 쉽다. 필요한 깊이까지만 구조 분해하고 입력 경계의 존재 확인은 따로 하면 반복을 줄이면서 실패 지점도 드러낼 수 있다.

## 3. 배열과 객체 구조 분해

```javascript
const [first, , third = 30] = [10, 20];
const { name: userName, role = "reader" } = { name: "Kim" };
console.log(first, third); // 10 30
console.log(userName, role); // Kim reader
```

배열 패턴은 순서를 기준으로 값을 소비하고, 객체 패턴은 속성 키로 조회한다. `name: userName`은 `name` 속성을 읽어 `userName`이라는 변수를 만드는 문법이다. `name`이라는 변수를 만드는 것도, 객체의 키 이름을 변경하는 것도 아니다.

기본값은 읽은 결과가 `undefined`일 때만 적용된다. `null`, `0`, `false`, 빈 문자열은 그대로 남는다. 중첩 구조 분해는 중간 객체가 실제로 존재하는지도 확인해야 한다.

### 선언 패턴과 대입 패턴

```javascript
let left = 1;
let right = 2;
[left, right] = [right, left];
console.log(left, right); // 2 1
const source = { name: "Kim", age: 20 };
let name;
({ name } = source);
console.log(name, source.name); // Kim Kim
```

첫 대입은 오른쪽 배열을 먼저 만든 뒤 왼쪽 이름들에 순서대로 연결한다. 새로운 변수를 선언하지 않고 기존 바인딩을 갱신한다. 객체 대입의 바깥 괄호는 문장 시작의 중괄호를 블록으로 해석하지 않도록 한다.

객체 구조 분해는 지정한 키로 값을 읽는다. 배열 패턴은 반복 가능한 값의 순서를 사용한다. 예를 들어 문자열을 배열 패턴으로 분해할 수 있어도 일반 객체가 같은 방식으로 반복 가능하다고 볼 수는 없다. [명세의 구조 분해 대입](https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment)은 두 경로를 별도로 정의한다.

## 4. 기본값이 적용되는 층위

```javascript
function describe({ name = "guest" } = {}) {
  return name;
}
console.log(describe()); // guest
console.log(describe({})); // guest
console.log(describe({ name: null })); // null
```

인자를 생략하면 바깥의 `= {}`가 적용된다. 빈 객체에는 `name`이 없으므로 안쪽 기본값이 적용된다. 반면 `{ name: null }`은 속성 값이 명시되어 있어 그대로 `null`이다. `describe(null)`은 바깥 기본값도 적용되지 않아 구조 분해 과정에서 `TypeError`가 난다.

```text
입력 인자 undefined? → 인자 기본값 적용
그 객체에서 name 읽기 → 결과 undefined? → 속성 기본값 적용
```

이 두 단계를 섞어서 “기본값이 있으니 어떤 입력도 안전하다”고 판단하면 안 된다. [구조 분해 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring)에서 매개변수 기본값의 적용 위치를 확인할 수 있다.

### 기본값을 놓을 위치를 결정한다

```javascript
function city({ address = {} } = {}) {
  const { name = "unknown" } = address;
  return name;
}
console.log(city(), city({ address: {} })); // unknown unknown
console.log(city({ address: { name: "Seoul" } })); // Seoul
```

이 함수는 인자 생략과 address 생략을 허용하지만 `address: null`까지 빈 객체로 해석하지 않는다. 기본값을 여러 단계에 적었다고 중간의 null이 모두 사라지지는 않는다. 어떤 결손을 정상적인 기본값으로 볼지 데이터 계약부터 정해야 한다.

기본값 식은 실제로 필요한 경우에 평가된다. 비용이 큰 함수나 부수 효과를 기본값 안에 넣으면 특정 입력에서만 실행된다. 패턴이 복잡해졌다면 중간 값을 꺼내 유효성을 확인한 뒤 다음 단계로 가는 편이 읽기 쉽다.

## 5. Spread: 값을 펼쳐 공급하기

```javascript
const numbers = [3, 5, 1];
console.log(Math.max(...numbers)); // 5
const combined = [0, ...numbers, 9];
console.log(combined.join(",")); // 0,3,5,1,9

const defaults = { theme: "light", size: 10 };
const options = { ...defaults, size: 20 };
console.log(options.theme, options.size); // light 20
```

호출·배열 위치의 spread는 **이터러블(iterable)**, 즉 순서대로 값을 꺼낼 수 있는 대상을 사용한다. 배열과 문자열은 대표적인 예다. 일반 객체를 `Math.max(...object)`로 펼칠 수 있다고 가정하지 않는다.

객체 spread는 자신의 열거 가능한 속성 값을 새 객체에 넣는다. 배열 spread와 동작 대상이 다르다. 같은 키가 여러 번 나오면 뒤에서 공급한 값이 덮어쓴다. `{ size: 20, ...defaults }`는 `size`가 다시 `10`이 된다.

### 객체 펼치기는 속성 값을 읽어서 새로 넣는다

```javascript
const source = {
  get count() { console.log("read count"); return 2; }
};
const copied = { ...source, count: 3 };
console.log(copied.count); // 3
```

출력은 `read count` 다음 `3`이다. 나중 속성이 덮어쓰더라도 앞의 spread에서 값을 읽는 효과는 발생한다. 복사 과정이 getter를 그대로 옮기는 것이 아니라 값을 읽는다는 점을 보여 준다. 따라서 객체 펼치기를 부수 효과 없는 저수준 메모리 복사로 해석하지 않는다.

여러 설정을 병합할 때는 기본 설정·사용자 입력·강제 설정의 순서를 먼저 적는다. 마지막에 보안상 중요한 값을 고정해야 하는지, 오히려 사용자 선택을 최종 우선으로 둘 것인지 요구에 따라 다르다. 키가 존재하고 값이 undefined여도 뒤쪽에서 같은 키를 공급하면 앞의 값을 덮을 수 있다.

## 6. Rest: 남은 값 모으기

```javascript
function sum(first, ...others) {
  let total = first;
  for (const value of others) total += value;
  return total;
}
const [head, ...tail] = [1, 2, 3];
const { token, ...publicData } = { token: "secret", name: "Kim", age: 20 };
console.log(sum(1, 2, 3)); // 6
console.log(head, tail.join(",")); // 1 2,3
console.log(Object.keys(publicData).join(",")); // name,age
```

나머지 매개변수는 아직 받지 않은 인자들을 실제 배열로 모은다. 배열 패턴의 rest도 남은 요소를 배열로 만들고, 객체 패턴의 rest는 선택한 키를 제외한 자신의 열거 가능한 속성을 새 객체로 모은다.

rest 요소·매개변수는 해당 패턴의 마지막에 온다. `...`를 보자마자 “복사”라고 읽기보다, 값을 밖으로 공급하는지 안으로 모으는지 주변 문법으로 판별한다.

### 수집 결과는 새 구조지만 값의 관계는 유지된다

```javascript
const shared = { n: 1 };
const source = { id: 7, payload: shared };
const { id, ...rest } = source;
console.log(rest === source, rest.payload === shared); // false true
function collect(...values) { return values; }
const result = collect(shared);
console.log(Array.isArray(result), result[0] === shared); // true true
```

rest는 새 객체 또는 배열을 만들지만 그 안의 객체 값은 그대로 연결된다. “모았다”는 것은 소유권을 옮기거나 깊은 복제를 했다는 뜻이 아니다. 원본 보존이 필요하면 수집 뒤 어떤 경로를 변경할지까지 판단한다.

매개변수의 rest는 남은 인자를 배열로 만들어 일반적인 배열 메서드를 사용할 수 있게 한다. 일반 함수의 arguments와 화살표 함수의 인자 처리 차이를 피하고 싶을 때도 매개변수에 수집 의도를 명시할 수 있다.

## 7. 실행 흐름과 공유 관계

```javascript
const source = { settings: { theme: "light" }, count: 1 };
const { settings } = source;
const copied = { ...source };
copied.count = 2;
settings.theme = "dark";
console.log(source.count, copied.count); // 1 2
console.log(source.settings.theme); // dark
console.log(copied.settings === source.settings); // true
```

구조 분해는 `settings`에 같은 객체를 연결한다. spread는 바깥 객체만 새로 만들고 `settings` 속성 값은 공유한다. 따라서 `count` 재할당은 분리되지만 중첩 객체 변경은 양쪽에서 보인다.

객체 spread는 프로토타입·비열거 속성·속성 기술자까지 보존하는 완전 복제가 아니다. 속성 값을 읽을 때 getter가 실행될 수도 있으므로 단순한 메모리 복사라고 이해하지 않는다.

### 제외와 병합의 처리 순서

공개 응답을 만들 때 `{ token, ...rest }`는 최상위 token 키를 제외한다. 그러나 `rest.profile.token`처럼 중첩된 값까지 검색해 없애지는 않는다. 공개 데이터 형식이 정해져 있다면 허용할 필드를 직접 골라 새 레코드를 만드는 방식도 비교해야 한다.

또한 rest가 가져오는 속성의 범위와 구조 분해로 직접 조회하는 범위는 동일하다고 단정하지 않는다. 직접 지정한 속성은 일반적인 속성 조회 규칙을 따르지만 객체 rest는 자신의 열거 가능한 나머지 속성을 수집한다. 이전에 상속된 값을 섞어 받은 데이터라면 결과가 달라질 수 있다. 프로토타입의 세부 원리는 JavaScript Core의 해당 주제에서 다룬다.

## 8. 흔한 오해·예외·주의점

- 구조 분해는 원본 객체를 수정하거나 자동으로 깊게 복사하지 않는다.
- 이미 선언한 변수에 객체 구조 분해로 대입할 때는 `({ name } = user);`처럼 문장 시작의 중괄호와 구별해야 한다.
- 옵션 병합에서 뒤쪽 객체를 신뢰하지 않으면 중요한 설정이 덮어써질 수 있다. 허용 필드를 명시하는 편이 나을 수 있다.
- 일부 키를 rest로 제외하는 것만으로 모든 민감 데이터가 제거되는 것은 아니다. 중첩 객체는 별도로 검사한다.

### 중첩 문법의 오류를 전체 입력 오류로 뭉개지 않는다

깊은 패턴 하나에서 실패하면 어떤 중간 값이 결손이었는지 오류 메시지만으로 불분명할 수 있다. 이때 패턴 전체를 try로 감싸 빈 객체를 반환하기보다 입력 구조를 단계별로 확인한다. 기본값은 허용된 누락을 채우는 용도이고 잘못된 구조를 모두 정상화하는 용도가 아니다.

배열·함수 호출의 spread는 값을 전부 공급할 수 있어 매우 큰 목록을 인자 목록으로 펼치면 환경의 인자 수 제한 등에 걸릴 수 있다. 대용량 목록의 최댓값을 계산하는 데 반드시 `Math.max(...values)`를 써야 하는 것은 아니다. 반복 계산과 비교해 입력 크기에 맞는 방식을 고른다.

## 9. 실무 적용과 디버깅 관점

기본값이 적용되지 않으면 실제 값이 `null`인지 `undefined`인지 확인한다. 병합 결과가 이상하면 속성 공급 순서를 왼쪽에서 오른쪽으로 적어 본다. 코드가 지나치게 깊게 중첩되면 한 번에 구조 분해하기보다 입력 확인과 꺼내기를 분리한다.

**[미리보기: Shallow Copy / Deep Copy]** 바깥 구조를 새로 만들어도 중첩 객체는 공유될 수 있다. 복사 깊이와 변경 경로 설계는 JavaScript Core의 Shallow Copy / Deep Copy · Immutability에서 다룬다.

### 옵션 덮어쓰기 버그를 재현한다

설정값이 예상과 다르면 병합 식을 값의 공급 순서대로 펼친 표로 바꾼다. `defaults.size → input.size → forced.size`처럼 같은 키를 공급한 모든 지점을 적고, 누락과 undefined도 구분한다. 그다음 문제를 단일 키와 작은 객체로 줄여 어떤 값이 마지막에 남는지 확인한다.

중첩 변경 문제에서는 결과 객체의 바깥 식별성과 변경 대상의 식별성을 함께 비교한다. “새 객체를 반환했다”만 확인하면 내부 공유를 놓친다. 이 두 검사를 나누면 병합 순서 오류와 공유 객체 변경을 서로 다른 원인으로 다룰 수 있다.

### 통합 사례: 옵션의 기본값과 강제값

설정 병합 요구가 “크기는 사용자가 선택할 수 있지만 내부 경로는 바꾸지 못한다”라고 하자. 단순히 defaults와 input을 아무 순서로 펼치면 두 요구 중 하나가 깨질 수 있다. 먼저 어떤 필드를 입력으로 허용할지 정하고, 기본값과 최종 우선순위를 구분한다.

```javascript
function configure({ size = 10, label = "default" } = {}) {
  return { size: size, label: label, path: "/internal" };
}
const a = configure({ size: 20, path: "/outside" });
const b = configure({ size: undefined, label: "custom" });
console.log(a.size, a.path); // 20 /internal
console.log(b.size, b.label); // 10 custom
```

이 함수는 입력 구조 분해에서 필요한 두 필드만 읽고 결과 객체의 path를 직접 정한다. path를 나중에 덮어썼다는 이유로만 안전한 것이 아니라 아예 허용 입력으로 취급하지 않은 설계다. size의 기본값은 undefined일 때만 적용되므로 null 같은 값까지 정상 크기로 보정한 것은 아니다.

그다음 업무 검증을 붙일 위치를 결정한다. 크기가 양의 정수여야 한다면 구조 분해가 끝난 뒤 별도 검사해야 한다. 문법이 값을 꺼내고 기본값을 채웠다는 사실은 유효한 설정이라는 증거가 아니다. 실패를 조사할 때에는 입력의 존재·기본값 적용·병합 우선순위·업무 검증 중 어느 단계가 어긋났는지 나누어 본다.

## 10. 핵심 정리

구조 분해는 꺼내기, spread는 펼치기, rest는 모으기다. 기본값의 조건과 병합 순서를 명시적으로 추적한다. 문법이 새 구조를 만들었다고 내부의 객체까지 독립된 것은 아니다.

## 출처

- [MDN — Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring): 패턴, 별칭, 기본값, 나머지 속성.
- [MDN — Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax): 펼치는 위치, 얕은 복사와 병합 순서.

- [ECMAScript — Destructuring Assignment](https://tc39.es/ecma262/multipage/ecmascript-language-expressions.html#sec-destructuring-assignment): 대입 패턴·이터레이터·속성 읽기.

## 11. 이해도 확인 질문

1. `{ name: nickname = "guest" }`에서 생성되는 변수 이름과 기본값 적용 조건을 설명하라.
2. `function f({ n = 1 } = {})`에 인자 생략, `{ n: null }`, `null`을 전달하는 경우를 비교하라.
3. `{ enabled: false, ...input }`에서 외부 입력이 설정을 덮어쓸 수 있는 이유는 무엇인가?
4. `[...items]`와 `{ ...items }`는 어떤 자료 구조와 키를 만드는가?
5. 중첩 객체를 구조 분해한 뒤 수정했을 때 원본도 바뀌는 이유를 설명하라.
