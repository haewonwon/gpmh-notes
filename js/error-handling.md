---
track: A
notion: https://app.notion.com/p/415178c4afac8198a0cadd0a2284cab3
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · Async JavaScript · Error Handling

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 오류 값·예외 발생·포착·복구·재전파를 구분한다.
- 동기 호출과 나중 실행되는 콜백·Promise의 실패 경계를 설명한다.
- await·catch·finally를 사용해 성공·실패와 정리를 연결한다.
- 원인 보존·오류 분류·재시도 기준을 설계하고 누락된 실패를 추적한다.

## 2. 이 주제를 왜 배우는가

잘못된 입력, 실패한 파싱, 외부 통신 장애를 모두 같은 빈 배열로 바꾸면 호출자는 “결과가 없음”과 “읽지 못함”을 구분할 수 없다. 반대로 어느 경계에서도 처리하지 않으면 실패 이유와 복구 방법을 사용자에게 제공하기 어렵다.

오류 처리는 예외를 없애는 작업이 아니라 실패를 어디서 발견하고, 누가 무엇을 복구하며, 복구하지 못한 실패를 어떻게 전달할지 정하는 작업이다. 정상값과 실패의 의미를 보존해야 이후 코드가 올바른 판단을 할 수 있다.

### 실패를 숨기지 않는 반환 계약

데이터가 실제로 없어서 빈 배열을 반환하는 경우와 읽기에 실패해 아무 데이터도 얻지 못한 경우는 다르다. 호출자가 재시도·오류 표시·빈 화면 중 올바른 행동을 고르려면 이 의미가 보존되어야 한다. 포착한 뒤 무조건 정상 기본값을 반환하는 것은 정책을 임의로 정한 셈이다.

오류 처리 위치도 책임에 맞춰야 한다. 입력 함수는 유효성 문제를 발견할 수 있지만 전체 기능의 대체 화면을 선택할 정보는 없을 수 있다. 하위에서는 원인을 보존해 전달하고, 사용자에게 보이는 기능 경계에서 복구 여부를 결정하는 구조를 비교한다.

## 3. Error 객체와 throw

**Error 객체**는 실패를 설명하는 값이다. 일반적으로 `name`, `message`, 구현이 제공하는 `stack`, 원인을 연결하는 `cause` 등을 활용한다. **throw**는 현재의 정상 흐름을 중단하고 가장 가까운 처리 가능한 경계로 예외를 전달한다.

```javascript
function requirePositive(n) {
  if (!Number.isFinite(n) || n <= 0) {
    throw new RangeError("amount must be positive");
  }
  return n;
}
try {
  console.log(requirePositive(-1));
} catch (error) {
  console.log(error.name, error.message);
}
console.log("continued");
```

출력은 `RangeError amount must be positive`, `continued`다. 예외가 발생한 함수는 정상 반환하지 않으며 호출 도중의 계산도 중단된다. catch를 마치면 그 바깥 흐름으로 이어진다. 실패한 함수의 throw 다음 줄로 돌아가 재개하는 것은 아니다.

`new Error(...)`만으로는 throw하지 않는다. Error는 값이므로 반환하거나 Promise 거부 이유로 쓸 수도 있다. JavaScript는 문자열·숫자도 throw할 수 있지만 Error를 쓰면 원인과 추적 정보를 일관되게 남기기 쉽다.

### 오류를 만들기와 발생시키기의 차이

```javascript
function asValue() { return new Error("returned"); }
function asFailure() { throw new Error("thrown"); }
console.log(asValue() instanceof Error); // true
try {
  asFailure();
} catch (error) {
  console.log(error.message); // thrown
}
```

첫 함수는 Error 객체를 정상값으로 반환했고 둘째는 정상 흐름을 예외로 끝냈다. 오류 객체가 보인다는 사실만으로 프로그램이 실패 경로로 이동했다고 판단하지 않는다. `Promise.reject에` Error를 전달하는 경우도 해당 Promise의 상태 계약이 적용된다.

오류 이름은 원인 범주를 설명하지만 업무상의 복구 가능성을 자동 결정하지 않는다. 같은 TypeError라도 외부 입력 검증 실패일 수도 있고 프로그램의 잘못된 접근일 수도 있으므로 발생 맥락을 함께 본다.

## 4. try/catch의 동기 경계

```javascript
function inner() { throw new Error("broken"); }
function outer() { return inner(); }
try {
  outer();
  console.log("skipped");
} catch (error) {
  console.log(error.message); // broken
}
```

inner의 예외가 활성 호출을 거슬러 올라가 outer의 호출을 포함한 try에 도달한다. `skipped`는 실행되지 않는다. 이때 포착할 수 있는 것은 해당 try의 실행 흐름 안에서 전달된 예외다.

같은 소스에 구문 오류가 있어 실행 자체가 시작되지 않으면 그 소스 안에 적은 try가 이를 정상적으로 감쌀 수 없다. 반면 `JSON.parse`가 실행 중 잘못된 문자열을 만나 던지는 SyntaxError는 포착할 수 있다. 오류 이름만으로 “구문 오류는 언제나 catch 불가”라고 일반화하지 않는다.

### 잡힌 뒤에는 실패 지점으로 되돌아가지 않는다

```javascript
let stage = "start";
try {
  stage = "parse";
  JSON.parse("{");
  stage = "saved";
} catch (error) {
  console.log(stage, error.name); // parse SyntaxError
}
console.log("after"); // after
```

parse가 실패하면 try의 나머지 saved 대입은 실행되지 않는다. catch 후에는 경계 바깥으로 진행하며 자동 재시도가 되지 않는다. 복구 뒤 다시 실행해야 한다면 그 제어 흐름을 명시해야 한다.

실패할 수 있는 범위를 너무 크게 감싸면 어떤 작업이 실패했는지와 이미 실행한 효과가 무엇인지 모호해진다. 입력 해석·외부 호출·저장 등을 구분하고, 각 단계에서 원인 정보와 완료 여부를 보존한다.

## 5. 나중 실행되는 콜백의 경계

```javascript
try {
  setTimeout(() => {
    // 이 콜백 내부에서 나중에 던지는 오류는 바깥 try가 잡지 못한다.
    console.log("later");
  }, 0);
} catch (error) {
  console.log("registration failed", error.message);
}
console.log("scheduled");
```

출력은 `scheduled`, `later`다. 바깥 try는 타이머를 등록한 뒤 끝났다. 콜백은 나중에 별도의 실행 흐름으로 호출되므로 이전 try의 동기 호출 경계에 들어 있지 않다.

실패 가능 작업을 콜백 내부에서 수행한다면 그 내부에 처리 경계를 두거나, error-first 콜백·reject 등 호출자가 이해할 수 있는 완료 계약으로 전달한다. 타이머 안에서 throw한 것을 생성 당시의 Promise executor가 자동으로 잡아 준다고 기대해서도 안 된다.

### 실패 전달 통로와 예외 포착 범위를 맞춘다

동기 throw, error-first의 error 인자, Promise 거부는 서로 다른 전달 통로다. 콜백으로 받은 error 값은 그것만으로 throw되지 않고, 거부된 Promise도 주변 동기 try만으로 포착되지 않는다. API 문서에서 사용하는 통로에 맞춰 처리해야 한다.

타이머 안의 작업을 Promise로 감쌀 때도 미래 예외를 reject로 연결하는 책임이 남는다. 단지 타이머 등록 코드를 new Promise 안에 적었다는 사실만으로 콜백 실행 전체가 생성자의 동기 포착 범위 안에 들어가지 않는다. 등록 시점과 실행 시점의 호출 경계를 각각 그린다.

## 6. Promise 거부와 await

```javascript
async function read() {
  return Promise.reject(new Error("offline"));
}
async function run() {
  try {
    await read();
    console.log("not reached");
  } catch (error) {
    console.log("caught", error.message); // caught offline
  }
}
run();
```

async 함수의 실패는 반환 Promise의 거부가 된다. `try { read(); }`만 쓰면 그 Promise의 미래 실패를 기다려 포착하지 않는다. `await read()`로 연결하거나 `read().catch(...)`를 반환·관리해야 한다.

```javascript
Promise.reject(new Error("failed"))
  .catch(error => { console.log(error.message); })
  .then(value => console.log("success path", value));
```

출력은 `failed`, `success path undefined`다. catch가 정상 종료해 거부를 성공으로 바꾸었기 때문이다. 오류를 기록만 하고 계속 실패로 전달하려면 `throw error`가 필요하다. 복구 기본값을 반환하는 경우에는 호출자가 그 기본값을 정상 결과로 취급해도 되는지 정해야 한다.

### 호출자에게 반환하지 않은 실패

```javascript
function disconnected() {
  Promise.reject(new Error("background failed"))
    .catch(error => console.log(error.message));
}
console.log(disconnected()); // undefined
```

이 예제는 내부 실패를 출력하지만 호출자는 그 완료를 기다리거나 성공·실패를 구분할 수 없다. 반환 계약이 fire-and-forget이라면 의도일 수 있으나 호출자가 작업 완료에 의존한다면 Promise를 반환해 연결해야 한다.

모든 비동기 호출이 반드시 상위로 실패를 전달해야 하는 것은 아니다. 그러나 상위 기능에 중요한 작업인지, 독립적인 부가 작업인지 정책이 필요하다. 반환을 놓친 실수와 의도적인 독립 실행을 구분해 기록한다.

## 7. finally의 정리와 결과 변경

**finally**는 정상 종료·반환·예외에 관계없이 해당 경계를 빠져나갈 때 정리를 수행한다. 일반적인 코드에서는 finally에서 return·throw로 기존 결과를 덮어쓰지 않는다.

```javascript
function example() {
  try {
    return "original";
  } finally {
    return "overridden";
  }
}
console.log(example()); // overridden
```

finally의 새 return이 원래 반환을 가렸다. throw를 새로 하면 원래 오류도 가릴 수 있다. 정리는 원래 성공·실패를 보존하는 형태로 설계하고 정리 자체가 실패하면 어떤 이유를 전달할지도 정한다.

`Promise.prototype.finally`도 정상적인 정리 완료 시 기존 결과를 전달하지만, 정리 함수가 던지거나 거부되면 그 실패로 바뀔 수 있다. 비동기 자원 정리는 `await`하여 완료·실패를 관리할 필요가 있다. finally가 존재한다고 모든 비동기 작업의 종료를 자동으로 기다리는 것은 아니다.

### 정리가 원래 결과를 가릴 때의 선택

파일·잠금·임시 상태 같은 자원을 정리하다 실패하면 원래 작업 실패와 정리 실패가 동시에 존재할 수 있다. finally에서 새 오류를 무조건 던지면 원래 원인을 잃을 수 있고, 정리 오류를 모두 무시하면 자원 문제를 숨길 수 있다.

어느 실패를 호출자에게 대표로 전달할지, 다른 원인을 cause나 별도 기록으로 어떻게 남길지 정한다. 비동기 정리는 시작만 하고 반환하지 않으면 바깥 기능이 끝난 뒤 실패할 수 있다. 정리가 완료 조건의 일부라면 await 또는 체인의 반환으로 연결한다. finally는 정리 코드를 적는 위치이지 모든 자원 정책을 자동 해결하는 기능은 아니다.

## 8. 원인 보존과 오류 분류

```javascript
function readSettings(text) {
  try {
    return JSON.parse(text);
  } catch (cause) {
    throw new Error("settings could not be read", { cause });
  }
}
try {
  readSettings("{");
} catch (error) {
  console.log(error.message); // settings could not be read
  console.log(error.cause instanceof SyntaxError); // true
}
```

호출자가 이해할 수 있는 문맥을 더하되 `cause`로 원래 실패도 유지한다. 메시지를 계속 덮어쓰면 실제 파싱 오류의 근거가 사라진다. 모든 잡힌 값이 Error라는 보장은 없으므로 외부 경계에서 `instanceof Error` 등으로 필요한 정보를 확인하고 문자열 거부도 처리할지 정한다. 서로 다른 realm의 객체에는 추가 주의가 필요하다.

| 분류 | 처리 판단의 예 |
| --- | --- |
| 입력·검증 실패 | 입력 수정 요청, 같은 요청의 무조건 재시도는 피함 |
| 일시적 외부 실패 | 제한된 재시도·대체 결과 가능성 검토 |
| 코드 결함 | 추적 정보 보존, 정상 성공값으로 숨기지 않음 |
| 취소 | 사용자의 의도된 중단인지 실제 장애인지 구분 |

오류 문자열을 전부 비교해 분기하기보다 API가 제공하는 안정적인 코드·종류·업무 계약을 사용한다. 구현별 stack 문자열의 정확한 형식은 표준의 고정 출력이라고 가정하지 않는다.

### 오류를 감싸는 이유와 감싸지 않을 이유

하위 오류에 “어느 설정을 읽다가 실패했는지” 같은 문맥을 더하면 상위에서 원인을 이해하기 쉽다. 이때 원본을 cause로 보존하면 추적 정보가 끊기지 않는다. 반대로 메시지를 그대로 복사한 새 Error만 매 단계 만들면 의미 있는 정보 없이 원래 추적 경로를 가릴 수 있다.

같은 오류를 그대로 전파해도 충분한 계층에서는 다시 던지는 것으로 계약을 유지할 수 있다. 업무상 다른 범주의 실패로 바꾸어야 하는 계층에서만 변환하고, 호출자가 안정적으로 분기할 수 있는 코드·종류를 제공할지 결정한다. 오류 메시지 문장 전체를 영구 API로 사용하는 것은 피한다.

## 9. 흔한 오해·예외·주의점

- **catch가 있으면 문제가 해결됐다**: 포착과 실제 복구는 다르다.
- **async 함수를 try 안에서 호출했으니 실패를 잡는다**: Promise의 거부를 연결해야 한다.
- **catch에서 출력만 하면 실패 상태도 유지된다**: 일반 종료는 성공 경로로 복구된다.
- **finally는 결과에 영향을 주지 않는다**: return·throw·거부된 정리 작업이 기존 결과를 덮을 수 있다.
- **재시도하면 항상 안전하다**: 이미 외부 효과가 완료됐을 수 있다. 중복 실행이 가능한지와 횟수·간격·중단 조건이 필요하다.

### 재시도의 대상은 코드 한 줄이 아니라 효과다

응답을 못 받았다고 요청이 처리되지 않았다고 단정할 수 없다. 이미 저장됐지만 응답이 끊긴 작업을 재시도하면 중복 효과가 생길 수 있다. 재시도 전에는 해당 작업이 반복되어도 같은 결과를 유지하는지, 중복 식별을 지원하는지, 어디까지 완료됐는지 확인해야 한다.

재시도를 허용하더라도 횟수·간격·전체 대기 시간·사용자 취소 조건을 정한다. 입력 오류나 코드 결함을 무한 재시도하는 것은 복구가 아니다. 일시적인 외부 실패와 영구적인 계약 위반을 구별할 근거를 API 문서와 업무 규칙에서 찾는다.

## 10. 실무 적용과 디버깅 관점

실패를 처리할 수 있는 가장 적절한 경계를 정한다. 하위 함수는 필요한 문맥을 더해 전달하고, 상위 기능 경계에서 대체 값·오류 표시·재시도를 판단할 수 있다. 여러 계층이 같은 오류를 중복 기록하면 한 사건이 여러 장애로 보일 수 있으므로 기록 책임도 정한다.

오류 유형, 작업 ID, 원인, 발생 위치, 시도 횟수를 남긴다. 사용자 입력 전체·인증 토큰 같은 불필요한 데이터 대신 재현에 필요한 정보만 선택한다. 미처리 거부·전역 오류 수집은 놓친 실패를 발견하는 보조 장치이며 정상 경로의 반환 계약을 대신하지 않는다.

작업을 시작만 하고 반환하지 않은 Promise, catch 뒤의 성공 처리, race 종료 뒤 계속되는 부수 효과를 함께 조사한다. 앞서 배운 비동기 조합의 성공·실패 계약과 업무상 완료를 구분해야 원인을 끝까지 추적할 수 있다.

### 실패 경계별로 관찰 결과를 검증한다

동기 실패, await 거부, 반환 누락, catch 복구, finally 실패를 각각 작은 사례로 실행한다. 기대한 반환값뿐 아니라 어떤 다음 단계가 건너뛰어졌는지와 원래 원인이 남았는지도 확인한다. “catch 로그가 찍혔다” 하나만으로 처리 완료라고 보지 않는다.

실제 로그에는 작업을 식별할 ID와 단계·오류 범주·원인을 남겨 같은 사건을 연결한다. 입력 전체를 무작정 출력하는 대신 재현에 필요한 최소 정보를 선택한다. 한 계층은 기록하고 다른 계층은 사용자 행동을 결정하는 식으로 책임을 나누면 중복 경고와 원인 누락을 줄일 수 있다.

## 11. 핵심 정리

오류를 값으로 만들기, throw·reject로 전달하기, 경계에서 포착하기, 복구 또는 재전파하기를 구분한다. 동기 try와 미래 비동기 실행은 같은 경계가 아니다. 실패 이유를 보존하고 정리가 원래 결과를 가리지 않도록 설계한다.

## 출처

- [MDN — Control flow and error handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling): throw·오류 값·동기 제어 흐름.
- [MDN — try...catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch): 포착 범위·재전파·finally 우선 결과.
- [MDN — Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error): 오류 종류·cause·추적 정보.
- [MDN — Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises): 거부 전달·catch·체인 경계.
- [Node.js — Errors](https://nodejs.org/api/errors.html#error-propagation-and-interception): 동기 예외·error-first·비동기 실패의 구분.

## 12. 이해도 확인 질문

1. 타이머 등록을 감싼 try가 타이머 내부의 나중 예외를 잡지 못하는 이유를 호출 경계로 설명하라.
2. async 함수를 try 안에서 호출만 한 경우와 await한 경우의 실패 전달은 어떻게 다른가?
3. catch에서 오류를 출력하고 끝내면 호출자가 성공으로 오해할 수 있는 이유와 수정 기준은 무엇인가?
4. finally의 return이나 정리 작업의 거부가 기존 결과에 어떤 영향을 주는가?
5. 파싱 오류에 업무 문맥을 더해 전달하면서 원인을 보존하고, 재시도 여부를 판단하는 방법을 설명하라.
