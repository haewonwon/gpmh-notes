---
track: A
notion: https://app.notion.com/p/415178c4afac81abb162d8e79bcaf5ea
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · Async JavaScript · async / await

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- async 함수의 반환 계약을 Promise로 설명한다.
- await 앞의 동기 구간과 뒤의 재개 시점을 추적한다.
- 의존 작업과 독립 작업의 시작 순서를 설계한다.
- 반복문·오류 포착·반환 누락에서 생기는 버그를 고친다.

## 2. 이 주제를 왜 배우는가

Promise 체인이 길어지면 단계별 지역 값과 조건 분기를 표현하기 번거로울 수 있다. **async/await**는 Promise의 성공·실패 흐름을 일반적인 문장과 지역 변수로 표현하는 문법이다. JavaScript 전체를 동기 실행으로 바꾸거나 별도 스레드를 자동 생성하지 않는다.

`async`는 함수의 반환을 Promise 계약으로 만들고, `await`는 특정 결과를 얻은 뒤 현재 async 함수의 후속 실행을 이어 간다. 기다리는 것은 해당 함수의 흐름이며 다른 작업의 실행 기회를 막는 바쁜 대기가 아니다.

### 문장 형태가 의존 관계를 없애지는 않는다

await를 쓰면 값이 준비된 뒤의 코드를 아래 줄에 이어 적을 수 있다. 하지만 그 위에 있던 외부 대기와 실패 가능성이 사라지는 것은 아니다. async 함수를 사용하는 호출자도 반환 Promise를 관찰해야 전체 기능의 완료를 알 수 있다.

따라서 리팩터링 때는 문법이 짧아졌는지뿐 아니라 작업 시작 순서, 실패가 전달되는 경계, 최종 반환값이 기존 체인과 같은지 확인한다. then을 await로 바꾸면서 원래 겹쳐 시작하던 작업을 순차 시작하면 성능 계약도 달라질 수 있다.

## 3. async 함수는 Promise를 반환한다

```javascript
async function answer() { return 42; }
const result = answer();
console.log(result instanceof Promise); // true
result.then(value => console.log(value)); // 42
```

`return 42`라도 호출 결과는 숫자 자체가 아니라 그 값으로 이행되는 Promise다. 함수 안에서 잡지 않은 예외는 반환한 Promise의 거부가 된다. `async`를 붙인 함수의 실패를 단순한 동기 호출의 throw와 같다고 판단하지 않는다.

async 함수가 기존 Promise를 반환하면 최종 결과를 따르지만 호출마다 새 Promise를 반환한다. 결과를 따른다는 사실과 객체 식별성이 같다는 것은 다르다.

### 결과를 따르는 것과 같은 객체를 반환하는 것

```javascript
const original = Promise.resolve(3);
async function relay() { return original; }
const returned = relay();
console.log(returned === original); // false
returned.then(value => console.log(value)); // 3
```

async 함수 호출은 새 Promise를 반환하고 original의 최종 결과를 따른다. 값이 같다고 Promise 객체의 식별성까지 같은 것은 아니다. 결과 캐시나 동일성 검사에서 호출 횟수와 반환 객체를 섞지 않는다.

또한 첫 await 이전에 throw가 발생해도 async 함수의 계약상 반환 Promise는 거부된다. 호출한 쪽은 동기 throw만 포착하려고 하지 말고 반환된 실패 경로를 연결해야 한다.

## 4. await의 중단과 재개

```javascript
async function run() {
  console.log("A");
  const value = await Promise.resolve(3);
  console.log("C", value);
}
console.log("start");
run();
console.log("B");
```

```text
start
A
B
C 3
```

함수는 호출 직후 첫 await에 도달할 때까지 동기적으로 실행된다. await한 값이 이미 이행된 Promise여도 후속 코드는 지금 바로 이어 실행하지 않고 나중의 재개로 이어진다. 일반 값 `await 3`도 후속 실행을 분리한다.

```text
호출 → A 출력 → await할 값 평가 → 현재 호출자에게 Promise 반환
호출자 B 출력 → 재개 기회 → value에 3 연결 → C 출력
```

await 오른쪽 표현식을 계산하는 작업 자체는 그 시점에 실행된다. `await heavyCalculation()`은 계산 함수를 호출하기 전부터 CPU 작업을 다른 곳으로 보내는 문법이 아니다.

### 일반 값을 await해도 재개는 분리된다

```javascript
async function run() {
  console.log("A");
  await 0;
  console.log("C");
}
run();
queueMicrotask(() => console.log("M"));
console.log("B");
```

출력은 `A B C M`이다. await 0은 이미 준비된 일반 값이라도 현재 함수의 나머지 실행을 분리한다. 이 예제에서는 재개가 먼저 준비되고 그다음 M을 등록했으므로 C가 앞선다. “await가 있으면 모든 다른 microtask 뒤에 간다”는 규칙은 없다.

[ECMAScript Await](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#await)는 값을 Promise 방식으로 해석하고 현재 평가를 나중에 이어 가는 절차를 둔다. 소스의 줄 배치보다 재개가 준비되는 순서와 이미 대기 중인 작업을 함께 추적한다.

## 5. 의존 작업의 순차 흐름

```javascript
function loadUser() { return Promise.resolve({ id: 7 }); }
function loadOrders(id) { return Promise.resolve(["order-for-" + id]); }
async function load() {
  const user = await loadUser();
  const orders = await loadOrders(user.id);
  return orders;
}
load().then(orders => console.log(orders.join(","))); // order-for-7
```

주문 조회에는 `user.id`가 필요하므로 사용자 결과 뒤에 시작하는 것이 자연스럽다. `await`는 이 의존 관계를 드러낸다. 함수가 최종 결과를 호출자에게 전달하려면 `return`해야 하며, 출력만 하면 호출자가 받는 성공 값은 `undefined`다.

### await의 범위와 반환값 사용

await는 값을 얻은 뒤 현재 함수 안의 의존 계산을 이어 가게 한다. 호출자까지 자동으로 기다리게 만드는 전역 장치가 아니다. `const result = load()`를 한 호출자는 여전히 Promise를 받으므로 그 결과를 처리할 때 then 또는 자신의 await가 필요하다.

중간 단계에서 출력만 하고 최종 값을 return하지 않으면 상위 호출자는 undefined 성공값을 받는다. async 함수에서 값을 얻었다는 사실과 그 값을 호출자에게 전달했다는 사실을 나눠 검사한다. 함수 경계마다 입력·반환·실패를 명시하면 문법을 바꿔도 계약을 유지하기 쉽다.

## 6. 독립 작업은 언제 시작하는가

독립된 작업에 무조건 `await`를 차례로 붙이면 두 번째 시작이 첫 번째 완료 뒤로 미뤄진다. 작업이 겹치는지 판단할 때 await 개수보다 작업을 실제로 호출한 시점을 본다.

```javascript
function task(name) {
  console.log("start", name);
  return Promise.resolve(name);
}
async function serial() {
  const a = await task("A");
  const b = await task("B");
  return a + b;
}
serial().then(value => console.log(value));
```

출력은 `start A`, `start B`, `AB`다. B는 A를 기다리는 구간 뒤에 시작한다. 실제 작업에 지연이 있으면 대기 시간이 합쳐질 수 있다. 반면 이미 시작된 Promise에 나중에 await하는 것은 그때 작업을 새로 시작하는 것이 아니다.

**[미리보기: `Promise.all` / allSettled / race]** 독립 작업들을 함께 시작하고 성공·부분 실패·먼저 끝난 결과를 묶는 방법이 있다. 안전한 결과 수집과 실패 연결은 다음 통합 문서에서 다룬다.

### 시작 시점과 기다리는 시점은 분리된다

`const a = taskA(); const b = taskB();`를 실행하면 두 호출은 이 줄에서 수행된다. 나중에 a를 await하는 순간 b가 처음 시작되는 것은 아니다. 반면 `await taskA(); await taskB();`는 두 번째 호출을 첫 완료 뒤로 미룬다.

독립된 작업을 먼저 시작한 뒤 하나씩 await하는 코드에는 실패 관찰의 빈 구간도 생길 수 있다. 뒤에서 await할 Promise가 먼저 거부되면 즉시 실패 처리가 연결되어 있지 않을 수 있다. 이 때문에 독립 작업의 시작과 성공·실패 수집을 한꺼번에 연결하는 다음 문서의 조합 API가 중요하다.

전체 작업이 모두 필요한지 일부 실패를 허용하는지 정한 뒤 수집 방식을 고른다. 단순히 await를 적게 썼다는 사실만으로 올바른 동시 실행 설계가 되지는 않는다.

## 7. 반복문의 await

```javascript
async function processItems() {
  const output = [];
  for (const n of [1, 2, 3]) {
    const value = await Promise.resolve(n * 2);
    output.push(value);
  }
  return output;
}
processItems().then(values => console.log(values.join(","))); // 2,4,6
```

이 반복은 한 요소의 처리가 끝나야 다음 요소로 간다. 순서 의존이나 요청량 제한이 있을 때 적합할 수 있다.

`forEach(async item => { ... })`는 다르다. `forEach`는 콜백이 반환한 Promise를 모아 기다리지 않으며 `forEach` 자체의 반환값은 `undefined`다. `await items.forEach(...)`를 해도 내부 작업 전체 완료를 기다리는 계약이 생기지 않는다. 순차 처리라면 `for...of`, 함께 수집하려면 다음 문서의 조합 API를 사용한다.

### 반복 API가 콜백의 Promise를 소비하는가

```javascript
const promises = [1, 2].map(async n => n * 2);
console.log(promises.length, promises[0] instanceof Promise); // 2 true
```

async 콜백을 map에 넘기면 map은 반환한 Promise들을 요소로 모은다. 숫자 배열로 자동 변환하거나 전체 완료를 기다리는 것은 아니다. 반면 forEach는 반환값을 모으지 않으므로 결과 배열조차 얻지 못한다.

순차 처리는 for...of에서 각 단계를 await해 표현할 수 있다. 독립 작업을 묶을 때는 먼저 반환 계약을 보존한 Promise 목록을 만들고 적절한 조합을 선택한다. 배열 API의 동기 순회 규칙과 async 함수의 반환 계약이 함께 적용된 결과다.

## 8. 실패가 들어오는 경계

```javascript
async function load() {
  try {
    const value = await Promise.reject(new Error("offline"));
    return value;
  } catch (error) {
    console.log(error.message); // offline
    return "fallback";
  }
}
load().then(value => console.log(value)); // fallback
```

await한 Promise가 거부되면 재개 시점에 해당 이유가 throw된 것처럼 전달되므로 주변 `try/catch`가 처리할 수 있다. `try` 안에서 `return somePromise`만 하면 그 Promise의 나중 거부를 지역 catch가 기다려 잡는 것은 아니다. 지역 catch에서 처리해야 한다면 `return await somePromise`처럼 await 경계를 둔다.

**[미리보기: Error Handling]** try/catch는 실행 경계에서 실패를 포착하고 복구·재전파를 결정한다. finally와 오류 원인 보존까지 포함한 정책은 이 섹션 마지막 Error Handling에서 다룬다.

### 지역 catch가 관찰할 실패를 연결한다

```javascript
async function load() {
  try {
    return await Promise.reject(new Error("offline"));
  } catch (error) {
    return "handled:" + error.message;
  }
}
load().then(value => console.log(value)); // handled:offline
```

return await에서는 결과를 기다리는 재개 지점에서 거부가 전달되므로 지역 catch가 작동한다. `return Promise.reject(...)`만 하면 반환 Promise로 결과를 넘긴 뒤 지역 try의 기다림이 없어 나중 거부를 그 catch가 처리하지 않는다.

모든 return에 기계적으로 await를 붙여야 한다는 뜻은 아니다. 지역 복구·정리 경계가 결과 완료를 관찰해야 하는지에 따라 선택한다. 호출자에게 그대로 실패를 전달하는 것이 계약이면 단순 반환이 적합할 수도 있다.

## 9. 흔한 오해·예외·주의점

- **async를 붙이면 함수 본문 전체가 나중에 실행된다**: 첫 await 전은 지금 실행된다.
- **await하면 스레드 전체가 멈춘다**: 현재 함수의 후속 실행이 분리된다.
- **반환이 Promise여도 일반 값처럼 읽을 수 있다**: 호출자는 then·await로 결과를 연결해야 한다.
- **await를 반복하면 언제나 같은 총 시간이 든다**: 작업을 언제 시작하는지와 의존 관계가 중요하다.
- **await는 어떤 파일의 최상위에서도 된다**: 일반적으로 async 함수 안 또는 이를 지원하는 ESM 최상위에서 사용한다. CommonJS 파일 최상위에는 그대로 쓰지 않는다.

### await가 CPU 작업을 옮기지 않는 이유

await 오른쪽의 함수를 호출하는 표현식은 먼저 평가된다. 그 함수가 큰 동기 계산을 수행하면 Promise 또는 값을 반환하기 전까지 현재 스레드를 점유한다. async라는 이름이나 await 키워드만으로 계산을 병렬화하는 기능은 생기지 않는다.

또한 아주 많은 await로 작업을 잘게 나누어도 그 재개가 마이크로태스크로 연속 이어지면 다음 일반 작업의 기회가 늦어질 수 있다. 분할 횟수와 UI 응답성을 같은 지표로 보지 않는다. 실제 실행 자원을 넘겨야 하는 요구인지 호스트 API와 함께 판단한다.

## 10. 실무 적용과 디버깅 관점

“시작 로그 → await 직전 → 재개 직후 → 반환”을 구분해 기록한다. 상위 호출자가 async 함수의 반환을 기다리는지도 확인한다. 예상보다 느리면 독립 작업을 불필요하게 순차 시작했는지, CPU 작업이 await 오른쪽에서 오래 실행되는지 나누어 조사한다.

### 변환 전후의 동작 계약을 대조한다

then 체인을 async/await로 바꿀 때 각 작업의 시작 순서를 적고 최종 반환값·실패 전달이 같은지 확인한다. 작업은 끝났는데 화면만 기다린다면 상위 함수가 반환 Promise를 놓쳤는지 살핀다. 특정 입력에서만 undefined가 오면 분기별 return을 검사한다.

지연은 함수 진입, 첫 await 직전, 각 재개 직후, 최종 반환으로 나누어 측정한다. 외부 대기와 그 사이 CPU 계산을 구분하면 순차 시작 문제와 무거운 동기 계산 문제를 다르게 해결할 수 있다.

## 11. 핵심 정리

async 함수는 Promise를 반환하고 await는 현재 함수의 후속 실행을 연결한다. 첫 await 전의 동기 코드, 작업 시작 시점, 결과 의존성을 각각 추적한다. 반복문과 오류 처리도 사용하는 API의 반환 계약을 따라야 한다.

## 출처

- [MDN — async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function): 반환 계약·동기 구간·재개.
- [MDN — await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await): 일반 값·실패·return await와 지역 처리.
- [ECMAScript — Async Functions](https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-async-function-definitions): async 함수의 언어 규칙.

- [ECMAScript — Await](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#await): 값 해석·평가 중단·재개 절차.

## 12. 이해도 확인 질문

1. async 함수의 첫 await 전후와 호출자 다음 줄의 출력 순서를 설명하라.
2. async 함수가 숫자를 return해도 호출자가 Promise를 받는 이유는 무엇인가?
3. 사용자 ID가 필요한 요청과 독립된 요청의 시작 순서는 어떤 기준으로 정해야 하는가?
4. `await items.forEach(async item => ...)`가 전체 완료를 기다리지 않는 이유를 반환 계약으로 설명하라.
5. 지역 catch에서 실패를 처리해야 할 때 `return promise`와 `return await promise`의 차이는 무엇인가?
