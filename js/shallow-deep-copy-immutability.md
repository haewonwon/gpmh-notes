---
track: A
notion: https://app.notion.com/p/415178c4afac818085dae1d91a2fd083
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · Shallow Copy / Deep Copy · Immutability

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 대입·얕은 복사·깊은 복사의 공유 범위를 비교한다.
- 변경 경로를 따라 새 객체를 만드는 불변 갱신을 설계한다.
- structuredClone·JSON 왕복·`Object.freeze의` 한계를 구분한다.
- 객체 식별성과 공유 관계로 원본 변경 버그를 디버깅한다.

## 2. 이 주제를 왜 배우는가

설정 객체를 복사해 편집했는데 원본도 바뀌거나, 매번 모든 데이터를 깊게 복사해 성능이 나빠질 수 있다. 복사의 목적은 무조건 많은 객체를 만드는 것이 아니라 원하는 변경 경계를 확보하는 것이다.

**얕은 복사(shallow copy)** 는 바깥 구조를 새로 만들고 속성 값들을 복사한다. **깊은 복사(deep copy)** 는 지원되는 중첩 값도 독립된 구조로 복제한다. **불변성(immutability)** 은 이미 사용하는 값을 제자리에서 바꾸지 않고 새 값으로 변경을 표현하는 설계 원칙이다.

### 독립성을 어느 경계에서 요구하는가

편집 초안을 만들 때 모든 중첩 객체를 바꾸는지, 특정 주소 필드 하나만 바꾸는지에 따라 필요한 복사 범위가 다르다. “복사본”이라는 이름만으로 계약을 정하지 말고, 원본과 공유해도 되는 가지와 분리해야 하는 가지를 표시한다.

모든 객체를 깊게 복제하면 비용뿐 아니라 참조 동일성도 바뀐다. 다른 코드가 같은 객체를 키나 캐시 식별자로 쓰고 있다면 불필요한 복제가 추가 조회와 갱신을 일으킬 수 있다. 독립성·성능·식별성을 함께 고려해야 한다.

## 3. 대입과 얕은 복사의 차이

```javascript
const original = { count: 1, profile: { city: "Seoul" } };
const alias = original;
const copied = { ...original };
copied.count = 2;
copied.profile.city = "Busan";
console.log(alias === original, copied === original); // true false
console.log(original.count, original.profile.city); // 1 Busan
console.log(copied.profile === original.profile); // true
```

대입은 같은 바깥 객체를 공유한다. spread는 바깥 객체를 새로 만들지만 `profile`의 값은 여전히 같은 객체다. 따라서 원시 속성 재할당은 분리되지만 중첩 속성 변경은 원본에서도 보인다.

```text
original ─> 바깥 객체 A ─┐
                       ├─> profile 객체
copied ───> 바깥 객체 B ─┘
```

`Array.prototype.slice`, 배열 spread, `Object.assign`도 중첩 객체를 자동으로 전부 복사하지 않는다. 복사를 몇 번 했는지보다 실제 변경 경로의 어느 객체가 공유되는지 확인한다.

### 공유 관계는 깊이뿐 아니라 연결 모양이다

```javascript
const shared = { count: 1 };
const source = { left: shared, right: shared };
const copy = { ...source };
copy.left.count = 2;
console.log(source.right.count); // 2
console.log(copy.left === copy.right); // true
```

left와 right는 원본에서도 같은 객체를 가리킨다. 얕은 복사는 이 연결을 그대로 유지한다. 따라서 “깊이 1까지 복사했다”는 말만으로는 어떤 변경이 다른 가지에 보이는지 충분히 설명하지 못한다. 같은 객체로 합쳐지는 별칭도 그려야 한다.

복사 API가 새 바깥 객체를 반환하는지, 공유된 하위 객체를 그대로 두는지, 동일한 하위 객체가 여러 번 등장할 때 그 관계를 보존하는지 각각 질문한다.

## 4. 변경 경로만 새로 만드는 불변 갱신

```javascript
const previous = {
  user: { name: "Kim", address: { city: "Seoul" } },
  settings: { theme: "light" }
};
const next = {
  ...previous,
  user: {
    ...previous.user,
    address: { ...previous.user.address, city: "Busan" }
  }
};
console.log(previous.user.address.city, next.user.address.city); // Seoul Busan
console.log(previous.user === next.user); // false
console.log(previous.settings === next.settings); // true
```

`city`까지 도달하는 바깥 객체·user·address는 새로 만든다. 바뀌지 않은 settings는 공유한다. 이를 **구조적 공유(structural sharing)** 라고 부른다. 이미 공유된 settings도 이후 직접 바꾸지 않는 계약이 있어야 이전 상태를 보존할 수 있다.

배열에서는 `map`으로 수정 대상만 새 객체로 만들 수 있다. 목록을 새로 만들었다고 모든 요소를 복제할 필요는 없지만, 수정하는 요소까지 그대로 공유하면 원본이 바뀐다.

### 배열 요소 하나를 갱신하는 경로

```javascript
const previous = [{ id: 1, done: false }, { id: 2, done: false }];
const next = previous.map(item =>
  item.id === 2 ? { ...item, done: true } : item
);
console.log(previous[1].done, next[1].done); // false true
console.log(previous === next); // false
console.log(previous[0] === next[0], previous[1] === next[1]); // true false
```

배열과 변경 대상 요소는 새로 만들고 변경하지 않은 요소는 공유한다. 이 결과는 모든 요소가 독립된 깊은 복제본은 아니지만, 현재 변경 경로에 대해서는 이전 상태를 유지한다. 이후에도 공유 요소를 제자리 변경하지 않는 계약이 필요하다.

원본 변경을 찾을 때는 새 배열 여부만 검사하지 않는다. 갱신할 요소와 그 아래 변경할 필드의 부모 객체까지 순서대로 비교한다. 필요한 경로 하나를 빠뜨리면 바깥을 여러 번 복사해도 원본이 오염될 수 있다.

## 5. structuredClone과 객체 그래프

**객체 그래프**는 객체가 다른 객체를 참조하는 연결 전체다. 깊은 복사는 단순한 나무 모양뿐 아니라 반복 참조·순환도 어떻게 처리할지 정해야 한다.

```javascript
// structuredClone을 지원하는 현대 브라우저 또는 Node.js
const source = { nested: { n: 1 } };
source.self = source;
const copy = structuredClone(source);
copy.nested.n = 2;
console.log(source.nested.n, copy.nested.n); // 1 2
console.log(copy.self === copy); // true
console.log(copy.self === source); // false
```

`structuredClone`은 HTML 표준의 구조화 복제 알고리즘을 사용하는 호스트 API이며 ECMAScript의 Object 메서드는 아니다. `Node.js도` 제공하지만 환경 지원을 확인한다. 순환·Map·Set·Date 등 여러 값을 지원하며 지원되지 않는 값이 포함되면 `DataCloneError`로 실패할 수 있다.

함수·DOM 노드·사용자 클래스의 동작과 private 필드까지 일반적으로 그대로 복원하는 만능 복제기는 아니다. 사용자 정의 프로토타입·접근자·속성 기술자를 원형 그대로 유지한다고 기대하지 않는다. [구조화 복제 표준](https://html.spec.whatwg.org/multipage/structured-data.html#structured-cloning) 참조.

### 순환과 중복 참조를 보존하는 이유

```javascript
const shared = { n: 1 };
const source = { a: shared, b: shared };
const copy = structuredClone(source);
console.log(copy.a === copy.b); // true
console.log(copy.a === source.a); // false
```

구조화 복제는 새 그래프 안에서 원본의 반복 참조 관계를 보존한다. a와 b를 아무 생각 없이 각각 재귀 복사하면 두 개의 별도 객체가 되어 원본과 다른 관계를 만들 수 있다. 순환에서는 방문한 객체를 추적하지 않는 재귀 복사가 종료하지 못할 수도 있다.

HTML의 [StructuredSerializeInternal](https://html.spec.whatwg.org/multipage/structured-data.html#structuredserializeinternal)은 이미 처리한 객체를 기록하는 절차를 포함한다. 함수나 사용자 정의 클래스의 모든 동작을 보존하는 알고리즘은 아니며, 지원 값과 직렬화 계약 안에서 그래프를 다루는 기능이다.

전송 가능한 자원은 복사 대신 소유를 옮기는 transfer 계약도 있다. 이 경우 원본 자원을 계속 사용할 수 있다고 가정하면 안 되며, 구체적인 호스트 자원 처리는 별도 API의 계약을 확인한다.

## 6. JSON 왕복이 같은 복사가 아닌 이유

```javascript
const data = { count: 1, missing: undefined, when: new Date("2020-01-01T00:00:00Z") };
const copy = JSON.parse(JSON.stringify(data));
console.log(Object.hasOwn(copy, "missing")); // false
console.log(typeof copy.when); // string
```

JSON은 직렬화 형식이다. 객체의 `undefined` 속성은 빠지고 Date는 문자열이 된다. BigInt나 순환 구조는 기본 처리에서 실패하며 함수와 Map·Set도 일반적으로 기대한 객체 형태를 보존하지 못한다. 데이터가 JSON 형식으로 제한되어 있다는 명확한 계약이 있을 때만 왕복의 의미가 단순해진다.

### JSON의 손실을 값 종류별로 확인한다

```javascript
const source = { missing: undefined, list: [undefined], count: NaN };
const text = JSON.stringify(source);
console.log(text); // {"list":[null],"count":null}
const copy = JSON.parse(text);
console.log(Object.hasOwn(copy, "missing"), copy.list[0] === null); // false true
```

객체의 undefined는 빠지지만 배열 위치는 null로 표현될 수 있다. NaN도 JSON 숫자가 아니어서 위와 같은 변환이 발생한다. 복원 후 타입·필드 존재 여부가 달라지므로 JSON 왕복을 일반적인 객체 보존 연산으로 쓰면 안 된다.

업무 데이터가 JSON 형식으로 제한되어 있다면 오히려 직렬화 경계를 명시할 수 있다. 그때도 저장 형식과 런타임 객체 복제 목적을 구별하고, 복원 시 검증과 클래스 동작 복구가 필요한지 결정한다.

## 7. const·freeze·불변 설계

`const`는 변수 재할당을 제한하고 `Object.freeze`는 대상 객체의 자신의 속성 추가·삭제·데이터 속성 재할당 등을 막는다. `freeze`는 원본 객체를 그대로 반환하며 기본적으로 얕다.

```javascript
"use strict";
const value = Object.freeze({ nested: { count: 1 } });
value.nested.count = 2;
console.log(value.nested.count); // 2
```

중첩 객체까지 동결한 것은 아니므로 위 변경은 가능하다. freeze된 객체의 금지된 속성 변경은 strict mode에서 오류가 될 수 있다. 또한 Map·Set의 내부 컬렉션 상태까지 `Object.freeze`가 자동으로 잠근다고 생각하지 않는다.

불변 갱신은 코드를 통해 지키는 계약이고 freeze는 특정 변경을 막는 런타임 장치다. 두 개념은 함께 쓸 수 있지만 같은 말은 아니다.

### freeze가 막는 대상과 내장 내부 상태

```javascript
const map = Object.freeze(new Map([["a", 1]]));
map.set("a", 2);
console.log(map.get("a"), Object.isFrozen(map)); // 2 true
```

freeze는 대상 객체의 일반적인 자신의 속성 구조를 제한하지만 Map의 내부 항목 저장소를 자동으로 잠그지 않는다. `Map.set이` 다루는 내부 상태와 객체의 일반 속성 대입은 다른 연산이다. 같은 이유로 “`Object.isFrozen이` 참이면 어떤 관찰 결과도 바뀌지 않는다”는 해석은 과도하다.

모든 중첩 객체를 재귀 동결하려면 순환·공유·내장 타입·함수 등을 어떻게 다룰지 설계해야 한다. 보편적인 deepFreeze 함수를 몇 줄로 만들어 모든 데이터를 안전하게 다룬다고 단정하지 않는다. 필요한 데이터 종류를 제한한 뒤 변경 계약을 정한다.

## 8. 흔한 오해·예외·주의점

- **깊은 복사가 항상 더 안전하고 빠르다**: 복제 비용·식별성 변경·미지원 타입을 고려한다.
- **새 객체면 불변이다**: 이전 객체를 계속 직접 바꾸면 과거 상태도 오염될 수 있다.
- **JSON 왕복은 어떤 데이터든 보존한다**: 형식 변환과 손실이 있다.
- **freeze하면 객체 그래프 전체가 불변이다**: 기본 동작은 얕고 내장 객체 내부 상태에도 한계가 있다.

### 복사와 검증은 별개의 책임

깊은 복제가 성공해도 데이터가 올바른 상태라는 뜻은 아니다. 잘못된 가격이나 누락된 사용자 ID도 그대로 복제할 수 있다. 복제는 공유 관계를 바꾸는 도구이고 입력 검증은 값의 의미를 확인하는 도구다.

반대로 원본을 변경하지 않는 함수가 항상 깊은 복제를 해야 하는 것도 아니다. 읽기만 하거나 변경 경로를 정확히 새로 만드는 경우에는 구조적 공유가 충분할 수 있다. 반환값을 유지한 채 불필요한 전체 복제를 줄이는 것이 성능과 계약을 함께 지키는 방법이다.

## 9. 실무 적용과 디버깅 관점

문제가 생긴 중첩 경로를 따라 `===`를 비교한다. 바깥 객체는 달라도 수정 대상이 공유될 수 있다. 업데이트 함수의 입력을 보관한 뒤 호출 후에도 원래 값이 같은지 확인하면 제자리 변경을 찾을 수 있다.

편집 초안·외부 경계 전달처럼 완전 분리가 필요한지, 이전 상태를 유지하며 작은 경로만 바꾸면 되는지 먼저 판단한다. 사용자 클래스라면 생성자·명시적 변환 함수로 복원할 동작을 정하는 편이 임의 복제보다 분명할 수 있다.

### 원본 오염을 최소 사례로 줄인다

원본 한 개, 얕은 복사 한 개, 문제의 중첩 객체 한 개만 남긴다. 갱신 전후에 바깥·중간·수정 대상 각각의 === 결과를 기록한다. 그다음 값을 바꾸는 한 줄만 실행해 어느 원본 경로가 함께 바뀌는지 확인한다.

해결 후에는 변경 값의 정확성, 원본 값의 보존, 바뀌어야 할 경로의 새 식별성, 공유해도 되는 경로의 유지라는 네 가지를 확인한다. 단순히 반환 객체가 원본과 다르다는 검사만으로는 불변 갱신을 검증하지 못한다.

## 10. 핵심 정리

얕은 복사는 바깥만 분리하고 깊은 복사는 지원 범위 안의 중첩 그래프도 분리한다. 불변 갱신은 바뀌는 경로를 새로 만들고 나머지를 공유할 수 있다. 복제·직렬화·동결의 목적과 한계를 각각 확인한다.

## 출처

- [MDN — Spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax): 얕은 복사.
- [HTML Standard — Structured data](https://html.spec.whatwg.org/multipage/structured-data.html#structured-cloning): 구조화 복제와 지원 값.
- [MDN — structuredClone](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone): 순환 구조·실패 조건.
- [MDN — Object.freeze](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze): 얕은 동결·동일 객체 반환.

- [MDN — JSON.stringify](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify): 값 종류별 직렬화 손실과 변환.

## 11. 이해도 확인 질문

1. `{ ...original }`을 만든 뒤 중첩 필드를 변경했을 때 원본이 바뀌는 과정을 그려라.
2. 깊이 세 단계의 필드를 불변 갱신하려면 어느 객체들을 새로 만들어야 하는가?
3. 변경하지 않은 가지를 공유해도 안전하려면 이후 코드가 어떤 계약을 지켜야 하는가?
4. 함수·Date·순환 참조가 섞인 데이터를 JSON 왕복으로 복사하려 할 때 무엇을 확인해야 하는가?
5. `const`·`Object.freeze`·불변 갱신이 각각 제한하거나 보존하는 대상을 비교하라.
