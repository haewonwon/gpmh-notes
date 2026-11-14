---
track: A
notion: https://app.notion.com/p/415178c4afac81ddbd94fa82b2e7b61a
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · Hoisting · TDZ

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 선언 바인딩의 생성·초기화·대입을 구분한다.
- 함수 선언문·`var`·`let`·`const`의 선언 전 접근을 비교한다.
- TDZ와 섀도잉이 결합된 오류를 추적한다.
- `typeof`와 함수 표현식의 예외를 정확히 설명한다.

## 2. 이 주제를 왜 배우는가

선언보다 앞의 함수를 호출할 수 있는데 어떤 변수는 `undefined`이고 다른 변수는 오류가 난다. 코드를 물리적으로 위로 옮긴다고 설명하면 왜 결과가 다른지 빠진다. 핵심은 이름을 준비하는 시점과 값을 사용할 수 있게 초기화하는 시점이다.

**호이스팅(hoisting)** 은 선언이 실행 전 준비되는 현상을 설명할 때 사용하는 말이다. 소스 텍스트가 실제로 이동하는 것은 아니다. **TDZ(Temporal Dead Zone, 일시적 사각지대)** 는 어휘 바인딩이 존재하지만 아직 초기화되지 않아 접근할 수 없는 시간 구간이다.

### 오류 이름의 차이를 설명하는 모델

선언 전 호출이 모두 같은 방식으로 실패한다면 단순히 “위에서 쓰지 마라”라고 외워도 될 것이다. 그러나 var 함수 표현식의 TypeError와 const 함수 표현식의 ReferenceError는 서로 다른 상태를 알려 준다. 전자는 이름은 읽었지만 호출할 함수 값이 없고, 후자는 이름을 읽는 것부터 금지된 상태다.

이 차이는 디버깅 방향을 정한다. 호출 대상이 undefined인 문제인지, 초기화 순서가 잘못된 문제인지 구분해야 바인딩을 바꾸는 임시 처방 대신 의존 관계를 고칠 수 있다.

## 3. 선언별 준비 상태

| 선언 | 본문 실행 전 일반적인 준비 상태 | 초기화 전 읽기 |
| --- | --- | --- |
| 함수 선언문 | 이름에 함수 객체 연결 | 함수 호출 가능 |
| `var` | 바인딩을 `undefined`로 초기화 | `undefined` |
| `let` | 바인딩은 있으나 미초기화 | `ReferenceError` |
| `const` | 바인딩은 있으나 미초기화 | `ReferenceError` |
| `class` | 어휘 바인딩, 미초기화 | `ReferenceError` |

표는 일반적인 함수·모듈 범위를 설명한다. 비엄격 스크립트의 블록 함수 선언에는 역사적인 추가 규칙이 있으므로 모든 위치에 똑같이 적용하지 않는다.

`let value;`도 실행이 선언 위치에 도달하면 `undefined`로 초기화된다. 선언 전에 있는 미초기화 상태와 선언 후의 `undefined`는 다르다.

### 생성·초기화·대입은 같은 연산이 아니다

바인딩 생성은 이름과 그 선언의 규칙을 마련하는 것이고, 초기화는 처음 사용 가능한 값을 연결하는 것이며, 이후 대입은 그 연결을 갱신하는 것이다. const의 재할당 금지는 초기화 자체가 없다는 뜻이 아니다. 생성된 바인딩을 선언 위치에서 초기화해야 한다.

[FunctionDeclarationInstantiation](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-functiondeclarationinstantiation)은 함수 호출을 준비할 때 매개변수와 선언을 처리하는 규칙이다. 전역 스크립트·모듈·블록은 각자의 준비 절차도 갖는다. 모든 코드가 함수 준비 알고리즘 하나만으로 동작한다고 일반화하지 않는다.

학습용 실행 표에는 `미생성`, `미초기화`, `undefined`, `실제 값`을 별도로 적는다. 특히 미초기화와 undefined를 한 칸으로 합치면 TDZ를 설명할 수 없게 된다.

## 4. var는 선언과 대입을 나누어 본다

```javascript
function show() {
  console.log(total); // undefined
  var total = 10;
  console.log(total); // 10
}
show();
```

함수 실행 준비에서 `total`은 `undefined`로 초기화된다. 첫 줄이 이를 읽고, `var total = 10`에 도달하면 초기값 표현식을 평가하여 `10`을 대입한다. `10`이라는 값 자체가 위로 올라가는 것은 아니다.

`var`가 함수 범위라는 사실과 선언 전 `undefined`를 읽을 수 있다는 사실을 함께 적용해야 한다. 코드의 블록 모양만 보고 다른 지역 변수라고 판단하지 않는다.

### 조건문을 실행하지 않아도 선언의 범위는 준비된다

```javascript
function show() {
  console.log(value); // undefined
  if (false) {
    var value = 10;
  }
  console.log(value); // undefined
}
show();
```

조건은 거짓이므로 10을 대입하는 문장은 실행되지 않는다. 하지만 var 선언의 함수 범위 바인딩은 호출 준비에 포함되어 undefined로 초기화된다. 선언 준비와 조건 분기 실행을 나누면 두 출력이 같은 이유가 분명해진다.

반면 블록 안의 let을 그 블록 밖에서 읽는 문제는 범위가 다르다. 선언의 존재 범위, 초기화 상태, 분기가 실제 실행되었는지를 차례로 확인해야 한다.

## 5. let·const와 TDZ

```javascript
let amount = 10;
{
  // console.log(amount); // 이 줄을 활성화하면 ReferenceError
  let amount = 20;
  console.log(amount); // 20
}
console.log(amount); // 10
```

블록 안의 `amount`는 블록 진입부터 안쪽 이름으로 해석되지만 선언 위치에 도달하기 전에는 초기화되지 않았다. 그때 접근하면 바깥의 `10`으로 대신 넘어가지 않고 오류가 난다. TDZ는 단순히 “코드 몇 번째 줄”이 아니라 실행 시 초기화 전후의 문제다.

```javascript
function run() {
  const read = () => value;
  let value = 3;
  return read();
}
console.log(run()); // 3
```

함수 본문의 `value` 사용 코드가 선언보다 앞에 작성되어 있어도 그 코드를 실행하는 호출 시점에는 초기화가 끝났다. 함수를 만드는 시점과 그 안에서 값을 읽는 시점이 다르기 때문이다. [TDZ 설명](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz) 참조.

### TDZ는 코드 위치만으로 판정하지 않는다

```javascript
function laterRead() {
  const read = () => value;
  const value = 7;
  return read;
}
const read = laterRead();
console.log(read()); // 7
```

함수 값을 만드는 시점에는 내부의 value 읽기가 실행되지 않았다. 실제 read 호출은 value가 초기화된 뒤다. 따라서 소스에서 식별자가 선언보다 앞에 보인다는 이유만으로 TDZ 오류라고 판단하지 않는다.

반대로 기본 매개변수도 왼쪽부터 초기화되므로 `function f(a = b, b = 1) {}`에 인자를 생략하면 앞의 기본값에서 아직 초기화되지 않은 뒤쪽 매개변수를 읽을 수 있다. TDZ를 선언문 앞의 몇 줄이라는 고정 영역보다 실행 순서의 상태로 이해해야 하는 이유다.

## 6. 함수 선언문과 함수 표현식

```javascript
console.log(declared()); // ready
function declared() { return "ready"; }

var expression = function () { return "later"; };
console.log(expression()); // later
```

`expression` 선언 전에 `expression()`을 호출하면 이름 자체는 있으나 값이 `undefined`여서 `TypeError`가 난다. `const expression = ...`을 선언 전에 호출하면 TDZ 때문에 먼저 `ReferenceError`가 난다. 둘 다 “아직 호출 못 한다”는 결과만 외우면 오류 원인을 놓친다.

화살표 함수도 표현식이므로 이를 담는 변수의 초기화 규칙을 따른다. `const`로 만든 함수를 “함수니까 선언 전 호출 가능하다”고 추론하지 않는다.

### 같은 이름을 다른 선언으로 바꾸면 준비 상태도 달라진다

```javascript
function run() {
  console.log(typeof action); // function
  function action() { return 1; }
  action = 3;
  console.log(typeof action); // number
}
run();
```

함수 선언 준비로 호출 가능한 값이 연결되어 있어도 이후 코드가 그 바인딩에 다른 값을 대입할 수 있다. 함수 선언이 준비됐다는 사실은 그 이름이 영원히 같은 함수를 가리킨다는 보장이 아니다.

함수를 const 표현식으로 바꾸면 재할당을 막을 수 있지만 선언 전 호출 규칙도 함께 바뀐다. 리팩터링에서는 코드 모양만 비교하지 말고 초기화 시점·호출 위치·재할당 가능성 세 가지를 함께 확인한다.

## 7. typeof의 예외와 자기 참조

선언하지 않은 이름에 대한 `typeof missingName`은 보통 `"undefined"`다. 그러나 TDZ의 이름에 `typeof`를 적용하면 `ReferenceError`다. `typeof`가 모든 잘못된 접근을 보호하는 연산자는 아니다.

`let value = value;`의 오른쪽 `value`도 이제 만들 안쪽 바인딩을 가리킨다. 오른쪽을 평가할 때 아직 초기화되지 않았으므로 TDZ 오류가 난다. 바깥의 같은 이름을 복사하고 싶었다면 지역 이름을 바꾸거나 입력을 명시해야 한다.

재선언 오류는 또 다른 문제다. 같은 어휘 범위의 중복 `let`·`const` 선언은 구문 분석 단계에서 실패할 수 있다. 초기화 전 읽기와 같은 런타임 오류로 섞지 않는다.

### 존재하지 않음과 존재하지만 사용할 수 없음

```javascript
console.log(typeof neverDeclaredHere); // undefined
{
  // typeof blocked; // 활성화하면 ReferenceError
  let blocked;
  console.log(typeof blocked, blocked); // undefined undefined
}
```

첫 검사는 이름이 해석되지 않는 특별한 typeof 경로다. 블록의 blocked는 이미 그 범위에서 선언된 이름이므로 초기화 전에는 이 특별 경로로 우회하지 못한다. 선언 위치를 지난 뒤에는 실제 undefined 값이므로 같은 typeof 결과라도 의미가 다르다.

`let x = x`에서 오른쪽 x가 바깥 값을 읽지 않는 것도 가장 가까운 선언의 존재 때문이다. 기본값·초기화 식 안의 이름을 해석할 때에도 스코프 검색 규칙이 먼저 적용된다는 점을 기억한다.

## 8. 흔한 오해·예외·주의점

- **let은 호이스팅되지 않는다**: 미초기화 바인딩이 바깥 이름을 가리는 현상을 설명하지 못한다.
- **var의 초기값도 위로 올라간다**: 선언 준비와 대입은 다르다.
- **선언 이전 줄에 함수가 있으면 무조건 오류다**: 실제 읽기 시점이 초기화 이후라면 가능하다.
- **TDZ가 있으니 변수는 실행 준비에 없다**: 이름은 존재하되 사용 가능한 값이 아직 없다.

### 초기화 순서를 숨기는 수정은 피한다

ReferenceError를 없애려고 const를 var로 바꾸면 undefined가 후속 연산으로 흘러가 오류가 더 늦게 나타날 수 있다. 프로그램이 실행된다는 사실과 올바른 값이 사용된다는 사실은 다르다. 초기화가 필요했다면 그 준비가 끝난 뒤 호출하도록 흐름을 바꾸는 것이 먼저다.

클래스 선언도 어휘 바인딩의 초기화 규칙을 갖는다. 함수 선언과 이름 있는 코드 단위라는 점이 비슷하다고 선언 전 new 호출까지 가능하다고 추론하지 않는다. 클래스 문법은 뒤의 Class에서 구체적으로 다룬다.

## 9. 실무 적용과 디버깅 관점

오류를 `ReferenceError`와 `TypeError`로 구분하고 현재 이름이 어느 스코프의 선언을 가리키는지 찾는다. 초기화 전에 실행되는 함수가 있는지, 순환 import로 미초기화 export를 읽는지 확인한다. 문제를 숨기려고 `var`로 바꾸기보다 의존 순서와 초기화 책임을 분명히 한다.

함수와 변수의 선언을 사용하는 곳 가까이에 명확하게 배치하면 실행 준비 규칙에 기대는 코드가 줄어든다. 선언 전 호출을 사용할 때에도 모듈의 초기화 순서가 이해 가능해야 한다.

### 한 이름의 상태를 시간표로 기록한다

문제의 이름을 어느 환경에서 찾았는지 적고, 그 환경 진입·선언 준비·초기화·첫 읽기의 순서를 나열한다. 오류가 발생한 줄에서 값이 undefined였는지 아직 읽을 수 없었는지를 구별한다. 호출된 함수 내부의 읽기라면 함수 생성 줄이 아니라 실제 호출 줄을 기준으로 분석한다.

모듈 순환에서 같은 오류가 나면 해당 모듈이 로딩되지 않았다고 바로 단정하지 않는다. 모듈은 연결되어도 export가 초기화되기 전일 수 있다. import 그래프와 최상위 평가 순서를 작은 두 파일로 줄여 재현하면 초기화 의존을 찾기 쉽다.

### 통합 사례: 선언 방식을 바꿀 때의 상태표

먼저 있던 함수 선언문을 const 표현식으로 바꿨더니 실행이 실패하는 경우를 생각하자. 함수 본문은 똑같더라도 초기화 규칙이 달라졌으므로 호출 위치를 다시 검토해야 한다.

```javascript
function successful() {
  const result = read();
  function read() { return 7; }
  return result;
}
function initializedFirst() {
  const read = () => 7;
  return read();
}
console.log(successful(), initializedFirst()); // 7 7
```

successful의 read는 본문 평가 전에 함수 값이 연결된다. initializedFirst는 const 초기화 뒤에 호출하기 때문에 안전하다. 두 번째 함수의 return 호출을 const 선언 앞에 옮기면 초기화 전 읽기에서 실패하며, var로 바꾸면 undefined를 호출하는 다른 실패가 될 수 있다.

| 선언 방식 | 호출 직전 필요한 상태 |
| --- | --- |
| 함수 선언문 | 해당 범위의 선언 준비 완료 |
| const 함수 표현식 | 선언 준비뿐 아니라 초기화 문장 실행 완료 |
| var 함수 표현식 | undefined 준비 상태를 넘어 함수 값의 대입 완료 |

리팩터링 검증은 같은 입력의 최종 값만 비교하지 않는다. 선언 이전 호출, 조건부 초기화, 다른 함수에서 간접으로 읽는 경로도 확인한다. 초기화가 끝나야 사용할 수 있는 의존 관계를 코드 순서나 명시적 생성 단계로 드러내야 한다.

## 10. 핵심 정리

호이스팅은 텍스트 이동이 아니라 선언 준비다. `var`의 `undefined`, 함수 선언문의 함수 객체, `let`·`const`의 미초기화 상태를 구분한다. TDZ 오류는 이름 검색과 실제 읽는 시점을 함께 추적해야 설명할 수 있다.

## 출처

- [MDN — let: TDZ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz): 초기화 전 접근, 섀도잉, typeof.
- [ECMAScript — FunctionDeclarationInstantiation](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-functiondeclarationinstantiation): 함수 호출의 선언 준비.
- [MDN — Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions#function_hoisting): 선언문과 표현식의 차이.

## 11. 이해도 확인 질문

1. `var f = () => 1`과 `const f = () => 1`을 선언 전에 호출할 때 오류가 다른 이유를 설명하라.
2. 바깥 `x`가 있어도 블록 안 `let x` 이전에 읽으면 실패하는 이유는 무엇인가?
3. 선언 이전에 만든 함수가 초기화 이후 호출되면 TDZ 오류가 나지 않을 수 있는 이유는 무엇인가?
4. `typeof missing`이 안전한 경우와 TDZ 때문에 실패하는 경우를 구분하라.
5. 선언·초기화·대입을 구분해 `var n = 5` 이전의 읽기 결과를 설명하라.
