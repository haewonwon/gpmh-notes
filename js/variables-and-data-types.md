---
track: A
notion: https://app.notion.com/p/400178c4afac80c6a04bedec2bad011a
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · 변수와 자료형 · Primitive / Reference

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 값의 타입과 변수에 저장된 값을 구분한다.
- 선언·초기화·재할당을 설명하고 `let`과 `const`를 선택한다.
- 원시 값의 불변성과 객체의 식별성을 대입·비교 예제로 설명한다.
- 자료형 검사와 명시적 변환을 사용해 입력 오류를 추적한다.

## 2. 이 주제를 왜 배우는가

장바구니를 복사한 뒤 수량을 수정했는데 원본도 바뀌거나, 입력한 `"10"`과 `1`을 더했더니 `"101"`이 되는 문제는 서로 다른 원인에서 출발한다. 앞의 문제는 같은 객체를 공유했기 때문이고, 뒤의 문제는 문자열과 숫자의 연산 규칙 때문이다.

변수 이름만 보고 데이터를 추측하면 이 둘을 구분하기 어렵다. 어떤 값이 들어 있는지, 어떤 연산을 수행했는지, 그 값이 다른 코드와 공유되는지를 나누어 보면 변경과 계산의 결과를 예측할 수 있다.

### 버그를 세 가지 질문으로 나누기

`copied.quantity += 1`에서 예상과 다른 결과가 나오면 먼저 `copied`가 새 객체인지 확인한다. 다음으로 `quantity`가 숫자인지 문자열인지 확인한다. 마지막으로 그 값을 바꾸는 것이 함수의 계약에 맞는지 확인한다. 같은 증상을 “복사가 안 됐다” 하나로 묶으면 숫자 변환 오류를 놓친다.

이 페이지에서는 변수 이름의 변경, 값의 종류, 객체 내부 변경을 각각 관찰한다. 같은 객체를 공유하는 것 자체는 정상적인 동작이다. 공유를 의도했는지와 어느 코드가 변경할 책임을 갖는지가 버그 여부를 결정한다.

## 3. 값·변수·바인딩

**값(value)** 은 프로그램이 다루는 데이터다. `3`, `"note"`, `true`, `{ count: 1 }`은 모두 값이다. **변수**는 값을 읽거나 저장할 때 사용하는 이름이며, 이름과 값의 연결을 **바인딩(binding)** 이라고 한다.

**선언**은 이름을 도입하는 것, **초기화**는 처음 값을 연결하는 것, **재할당**은 이미 초기화된 연결을 다른 값으로 바꾸는 것이다. 객체 내부의 속성을 바꾸는 **변경(mutation)** 은 변수 재할당과 별개다.

```javascript
let count;           // 선언문 실행 시 undefined로 초기화
console.log(count);  // undefined
count = 2;           // 재할당
const cart = { count: 1 };
cart.count = 3;       // 객체 변경
console.log(count, cart.count); // 2 3
```

`const`는 초기화 후 변수의 재할당을 금지한다. 객체 전체를 얼리거나 그 안의 값을 상수로 만들지는 않는다. 기본적으로 `const`를 쓰고, 같은 이름에 새 값을 넣어야 할 때 `let`을 선택하면 변경 지점이 드러난다.

| 선언 | 재할당 | 일반적인 유효 범위 | 기본 사용 기준 |
| --- | --- | --- | --- |
| `const` | 불가 | 블록 | 재할당이 없는 이름 |
| `let` | 가능 | 블록 | 반복 횟수 등 값이 바뀌는 이름 |
| `var` | 가능 | 함수 또는 전역 | 기존 코드 해석 시 구분 |

### 읽기·계산·다시 연결하기를 분리한다

`count = count + 1`은 기존 숫자 값의 내부를 고치는 연산이 아니다. 오른쪽의 `count`를 읽고, 덧셈 결과를 계산한 뒤, 왼쪽 이름에 결과를 다시 연결한다. 따라서 `const count = 1`이라면 계산은 가능해도 그 이름에 결과를 다시 대입하는 단계는 금지된다.

```javascript
let count = 1;
const before = count;
count = count + 1;
console.log(before, count); // 1 2
const settings = { count: 1 };
settings.count = settings.count + 1;
console.log(settings.count); // 2
```

두 번째 경우에는 `settings`의 연결이 유지되고 객체의 `count` 속성 값이 바뀐다. “변수를 바꿨다”라는 말 대신 연결을 교체했는지 속성을 변경했는지까지 설명해야 `const`의 허용 범위를 정확히 판단할 수 있다.

## 4. 원시 값과 객체

ECMAScript 언어 타입은 `Undefined`, `Null`, `Boolean`, `String`, `Symbol`, `Number`, `BigInt`, `Object`다. 앞의 일곱 타입을 **원시 타입(primitive type)** 이라고 한다. 원시 값은 그 자체를 내부에서 수정할 수 없다.

| 종류 | 예 | 주의할 점 |
| --- | --- | --- |
| Undefined / Null | `undefined`, `null` | 미지정과 명시적 비어 있음의 의도를 구분 |
| Boolean | `true`, `false` | 조건 판단의 결과 |
| String | `"hello"` | 문자 일부를 직접 덮어쓸 수 없음 |
| Number | `2`, `0.5`, `NaN` | 정수와 소수를 같은 타입으로 표현 |
| BigInt | `123n` | Number와 산술 연산 시 무작정 혼합하지 않음 |
| Symbol | `Symbol("id")` | 같은 설명이어도 호출마다 별개의 값 |
| Object | `{}`, `[]`, 함수 | 속성·식별성·공유 관계를 가짐 |

“참조형”이라는 교육용 분류를 쓰더라도 별도의 `Reference` 언어 타입이 있다고 이해하면 안 된다. 배열과 함수도 객체이며, 객체의 대입은 객체 내부를 복제하지 않는다. 명세의 내부 용어인 Reference Record와도 구분한다. [ECMAScript 타입 정의](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-ecmascript-language-types)에서 언어 타입을 확인할 수 있다.

### 숫자와 문자열의 표현 한계

Number는 일반적으로 IEEE 754 배정밀도 표현을 사용한다. 저장 가능한 수 사이의 간격은 수의 크기에 따라 달라지고, `Number.MAX_SAFE_INTEGER`를 넘는 정수는 이웃한 정수와 정확히 구분되지 않을 수 있다. BigInt는 큰 정수를 위한 별도 타입이지 소수를 임의 정밀도로 계산하는 타입이 아니다.

문자열은 UTF-16 코드 단위의 나열이다. `length`를 곧 사용자가 보는 글자 수라고 해석하면 일부 이모지 등에서 틀릴 수 있다. 또한 `text[0] = "A"`처럼 원래 문자열의 일부를 교체하는 방식으로 수정하지 못한다. 필요한 결과 문자열을 새로 계산해 사용한다. 이러한 타입별 성질은 [ECMAScript의 String·Number 정의](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html#sec-ecmascript-language-types)를 기준으로 읽는다.

`undefined`는 선언 후 값을 정하지 않은 상황 등에서 자주 나타나고 `null`은 명시적으로 비어 있음을 표현하는 데 흔히 쓴다. 다만 이 사용 의도는 업무 계약이다. API가 어느 값을 사용하는지는 실제 스키마를 확인해야 하며 두 값을 무조건 같은 정상값으로 바꾸지 않는다.

## 5. 대입과 비교의 실행 흐름

```javascript
let first = 10;
let second = first;
second = 20;
console.log(first, second); // 10 20

const original = { score: 10 };
const alias = original;
alias.score = 20;
console.log(original.score); // 20
console.log(alias === original); // true
console.log({ score: 20 } === original); // false
```

원시 값 예제에서는 `second`에 다른 값을 재할당해도 `first`의 연결은 바뀌지 않는다. 객체 예제에서는 두 이름이 같은 객체를 가리키므로 어느 이름으로 속성을 수정하든 결과가 보인다. 마지막의 객체 리터럴은 내용이 같아도 새 객체를 만들었기 때문에 `false`다.

```text
original ─┐
          ├─> 하나의 객체 { score: 20 }
alias ────┘
```

이 그림은 관찰 가능한 공유 관계를 나타내는 모델이다. 실제 주소, 스택·힙 배치, 엔진의 최적화 방식까지 보장하는 메모리 배치도는 아니다.

### 원시 값의 동일성과 객체의 동일성

```javascript
const a = "note";
const b = "no" + "te";
const first = { name: "note" };
const second = { name: "note" };
console.log(a === b); // true
console.log(first === second); // false
console.log(NaN === NaN, Object.is(NaN, NaN)); // false true
console.log(0 === -0, Object.is(0, -0)); // true false
```

문자열은 만드는 식이 달라도 같은 문자열 값이면 엄격 비교에서 같다. 반면 객체는 생성할 때 별도의 식별성을 얻는다. 같은 키와 값으로 만들었다고 객체가 합쳐지지 않는다. `===`는 객체 내부를 재귀적으로 순회하는 깊은 비교가 아니다.

`NaN`과 부호 있는 0은 비교 규칙을 구분해야 하는 대표적인 경계다. `Object.is`가 모든 목적에 더 좋은 비교는 아니다. 어떤 종류의 동일성을 확인하는지 먼저 정한다. 명세는 [IsStrictlyEqual과 SameValue](https://tc39.es/ecma262/multipage/abstract-operations.html#sec-testing-and-comparison-operations)를 별개의 연산으로 정의한다.

## 6. 타입 검사와 변환 예제

JavaScript는 **동적 타입 언어**다. 변수 이름에 고정된 타입을 붙이는 대신, 현재 값이 타입을 갖는다. 같은 `let` 변수에 숫자 다음 문자열을 넣을 수 있다.

```javascript
console.log(typeof 10, typeof "10"); // number string
console.log(typeof null); // object
console.log(typeof []); // object
console.log(Array.isArray([])); // true
console.log(Number.isNaN(NaN)); // true
console.log("10" + 1); // 101
console.log(Number("10") + 1); // 11
console.log(Boolean("false")); // true
```

`typeof null`의 결과는 역사적으로 `"object"`이지만 `null`은 원시 값이다. 배열 여부는 `Array.isArray`, `NaN` 여부는 `Number.isNaN`처럼 목적에 맞는 검사를 사용한다. `Boolean("false")`는 문자열의 뜻을 해석하지 않는다. 비어 있지 않은 문자열이므로 참이다.

`+`는 문자열 연결과 숫자 덧셈을 모두 수행한다. 입력 경계에서 숫자로 바꾼 후 유효성을 검사해야 계산 중간의 암묵적 변환을 줄일 수 있다.

### 변환 성공과 유효한 입력은 다른 단계다

```javascript
const raw = " ";
const converted = Number(raw);
console.log(converted, Number.isFinite(converted)); // 0 true
console.log("5" - 1, "5" + 1); // 4 51
console.log(null == undefined, null === undefined); // true false
```

빈 내용이 숫자 `0`으로 변환될 수 있으므로 `Number.isFinite`만으로 필수 입력 누락을 찾지 못한다. 먼저 입력을 허용하는지 확인하고, 그다음 타입을 변환하며, 마지막으로 업무 범위를 검사한다. 예를 들어 수량이라면 비어 있지 않음·유한한 숫자·정수·최솟값을 별도로 판단한다.

`-`와 `+`의 변환 규칙도 다르다. 문자열에서 온 숫자를 자동 변환에 맡긴 채 여러 연산을 섞으면 계산과 연결이 뒤섞인다. 일반 코드에서는 명시적 변환과 `===`를 기본으로 사용하고, `==`가 필요하다면 허용할 변환의 의미를 설명할 수 있어야 한다.

## 7. 흔한 오해·예외·주의점

- **`const` 객체는 수정할 수 없다**: 금지하는 것은 변수 재할당이다.
- **객체 내용이 같으면 `===`도 참이다**: 객체 비교는 식별성을 비교한다.
- **원시 값 불변이면 변수 값도 바꿀 수 없다**: 변수 재할당과 값 자체의 변경은 다르다. 문자열을 이어 붙이면 새 문자열 값이 생긴다.
- **모든 숫자를 정확하게 계산한다**: Number는 이진 부동소수점이다. `0.1 + 0.2 === 0.3`은 거짓이며, 금액·정밀도 요구는 별도로 설계한다.
- **`undefined`와 미선언은 같다**: 값이 없는 초기화된 변수와 존재하지 않는 이름은 다르다. 미선언 이름을 직접 읽으면 보통 `ReferenceError`다.

### 동적 타입과 약한 입력 검증을 혼동하지 않는다

변수가 다른 타입의 값을 가질 수 있다는 언어 성질은 잘못된 값도 모두 허용하라는 설계 원칙이 아니다. 함수 경계에서는 필요한 타입과 값의 범위를 정할 수 있다. 반대로 문자열을 숫자로 변환했다고 원래 입력의 단위·통화·부호 의미까지 보존했다고 단정해서도 안 된다.

객체를 `console.log`로 출력할 때는 출력 문자열과 개발자 도구가 보관하는 객체 참조를 구분한다. 로그 시점에는 `count: 1`이었어도 나중에 펼친 객체에서 `count: 2`를 볼 수 있다. 공유 변경을 추적하는 실험에서는 현재 원시 속성 값과 객체끼리의 동일성 결과를 함께 출력한다.

## 8. 실무 적용과 디버깅 관점

문제를 재현할 때 변수 이름, `typeof`, 실제 값, 다른 객체와의 `===` 결과를 함께 기록한다. 개발자 도구의 객체 펼침 화면은 나중 시점의 상태를 보여 줄 수 있으므로 원시 속성을 즉시 출력하거나, JSON으로 표현 가능한 데이터라면 `JSON.stringify`로 그 시점의 값을 남긴다.

외부 입력은 문자열일 수 있고 빈 문자열을 숫자로 바꾸면 `0`이 될 수 있다. “변환에 성공했는가”와 “업무상 유효한 값인가”를 각각 검사한다. 객체가 함께 바뀌면 먼저 복사 여부보다 공유 경로부터 찾는다.

### 입력 경계를 좁히는 사례

수량 입력이 `"2"`인데 합계가 `"21"`이 됐다면 데이터가 들어오는 지점부터 순서대로 추적한다. 첫째, 화면·파일·API에서 받은 실제 값과 타입을 기록한다. 둘째, `+` 직전 값이 숫자로 정규화됐는지 확인한다. 셋째, 실패한 변환을 정상적인 0으로 숨기는 기본값 처리가 있는지 확인한다.

객체가 함께 바뀌는 문제에서는 처음 만들어진 객체를 중심으로 별칭을 표시한다. 새 변수의 이름이 `copy`여도 대입만 했다면 복제는 없다. 데이터를 읽기만 하는 코드와 변경하는 코드를 분리하고, 의도된 공유라면 그 변경 책임을 문서화한다. 이 절차는 엔진의 메모리 주소를 몰라도 관찰 가능한 값만으로 수행할 수 있다.

## 9. 필요한 미리보기

**[미리보기: Scope · Hoisting · TDZ]** `let`·`const`·`var`는 이름이 보이는 범위와 선언 전 접근 규칙도 다르다. 자세한 실행 준비 과정은 JavaScript Core의 Scope와 Hoisting · TDZ에서 다룬다.

**[미리보기: Shallow Copy / Deep Copy]** 객체를 새 변수에 대입하는 것과 복사하는 것은 다르다. 중첩 객체의 공유를 끊는 범위는 JavaScript Core의 Shallow Copy / Deep Copy · Immutability에서 다룬다.

## 10. 핵심 정리

변수의 연결, 값의 타입, 객체의 공유 관계를 따로 추적한다. 원시 값은 불변이고 객체는 식별성을 가지며, 대입만으로 객체 내부가 복제되지는 않는다. 타입 검사와 명시적 변환은 입력을 해석하는 도구이지 업무 규칙을 대신하는 검증이 아니다.

## 출처

- [ECMAScript — Data Types and Values](https://tc39.es/ecma262/multipage/ecmascript-data-types-and-values.html): 원시 타입·객체 타입과 값의 성질.
- [MDN — Grammar and types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types): 변수 선언, 동적 타입, 리터럴과 변환.

- [ECMAScript — Testing and Comparison Operations](https://tc39.es/ecma262/multipage/abstract-operations.html#sec-testing-and-comparison-operations): 엄격 비교와 SameValue의 경계.

## 11. 이해도 확인 질문

1. `const a = { n: 1 }; const b = a; b.n = 4;` 실행 후 두 변수가 읽는 값과 `a === b`를 예측하고 설명하라.
2. `let a = "hi"; let b = a; b += "!";`에서 `a`가 바뀌는지 값의 불변성과 재할당으로 설명하라.
3. 입력 `"0"`, `""`, `"false"`를 각각 숫자·불리언으로 사용할 때 어떤 검증이 필요한가?
4. `typeof value === "object"`만으로 배열과 일반 객체를 판별하는 코드에는 어떤 반례가 있는가?
5. 객체 내용이 같은지 검사하려고 `===`를 사용했다. 이 검사가 실제로 확인하는 것은 무엇인가?
