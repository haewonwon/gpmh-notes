---
track: A
notion: https://app.notion.com/p/415178c4afac818faed2cddb3d66a5f0
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · Async JavaScript · Promise

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- Promise의 상태와 resolved·settled의 차이를 설명한다.
- executor와 then 후속 처리의 실행 시점을 구분한다.
- 반환값·예외·다른 Promise가 다음 체인에 미치는 영향을 추적한다.
- 반환 누락·끊어진 체인·중복 resolve·불필요한 생성자를 찾는다.

## 2. 이 주제를 왜 배우는가

콜백만으로 여러 비동기 단계를 연결하면 완료 경로와 오류 경로가 중첩되어 흐름을 읽기 어렵다. **Promise**는 나중에 제공될 성공 값 또는 실패 이유를 나타내는 객체다. 호출자는 이 객체에 후속 함수를 연결하여 단계별 계산을 조합할 수 있다.

Promise 자체가 외부 작업을 다른 스레드로 보내거나 무거운 코드를 자동으로 나누지는 않는다. 외부 작업을 시작하는 API와 그 작업의 결과를 나타내는 Promise를 구분한다.

### 결과 객체와 실행 중인 작업

Promise는 완료를 관찰하고 후속 계산을 연결하는 객체다. 이미 시작한 타이머나 요청이 이 객체와 연결될 수 있지만 Promise가 작업의 모든 자원을 직접 소유하는 것은 아니다. 결과를 더는 기다리지 않는 것과 실제 작업을 중단하는 것은 별도 계약이다.

이 구분은 함수를 언제 호출하는지에도 중요하다. Promise를 만드는 생성자나 Promise 반환 함수를 호출하는 순간 일부 작업이 시작될 수 있다. 나중에 then을 붙이는 행위가 언제나 작업의 시작 스위치는 아니다.

## 3. 상태와 운명

| 상태 | 의미 |
| --- | --- |
| pending | 성공·실패가 아직 확정되지 않음 |
| fulfilled | 성공 값으로 이행됨 |
| rejected | 실패 이유로 거부됨 |

fulfilled 또는 rejected이면 **settled**다. **resolved**는 다른 Promise나 thenable의 결과를 따르도록 결정된 경우까지 포함하므로 지금 당장 fulfilled라는 뜻은 아니다. **thenable**은 호출 가능한 `then`을 통해 결과를 전달하는 객체다.

```javascript
const inner = new Promise(resolve => {
  setTimeout(() => resolve("ready"), 0);
});
const outer = new Promise(resolve => {
  resolve(inner);
});
outer.then(value => console.log(value)); // ready
```

outer는 inner의 결과를 따르도록 resolved되었지만 inner가 끝날 때까지 pending일 수 있다. 성공을 전달하는 데 `resolve`라는 이름을 쓰더라도 두 용어를 완전히 같은 상태로 외우지 않는다.

### resolve를 두 번 부를 때와 코드 실행

```javascript
const p = new Promise((resolve, reject) => {
  resolve(1);
  reject(new Error("ignored"));
  resolve(2);
  console.log("executor still runs");
});
p.then(value => console.log(value));
```

출력은 `executor still runs` 다음 `1`이다. 첫 완료 결정이 뒤의 호출로 교체되지 않지만 executor의 나머지 코드가 자동 중단되는 것도 아니다. resolve가 return이나 throw처럼 제어 흐름을 종료한다고 생각하면 완료 뒤 부수 효과를 놓친다.

다른 pending Promise로 resolve하면 그 결과를 따르도록 결정된 뒤에도 아직 settled가 아닐 수 있다. 이후 reject를 호출해 결정을 뒤집는 방식으로 취소를 구현할 수는 없다. 상태 전이와 작업 중단 신호를 분리한다.

## 4. 생성자와 executor의 실행 시점

```javascript
console.log("A");
const promise = new Promise((resolve, reject) => {
  console.log("B");
  resolve(10);
  console.log("C");
});
promise.then(value => console.log("E", value));
console.log("D");
```

```text
A
B
C
D
E 10
```

생성자에 넘긴 **executor** 함수는 즉시 동기적으로 실행된다. `resolve(10)`은 뒤의 `console.log("C")`를 중단하지 않는다. 반면 `then`에 등록한 성공 후속 함수는 현재 동기 코드가 끝난 뒤 Job·마이크로태스크 규칙에 따라 실행된다.

executor의 `return` 값은 Promise의 성공 값이 아니다. 성공·실패는 `resolve`·`reject`를 통해 전달한다. 아무 경로에서도 완료 함수를 호출하지 않으면 pending으로 남을 수 있다. 같은 resolve/reject 쌍의 첫 호출로 결과를 따를 대상이 정해지면 이후 호출은 그 결정을 바꾸지 못한다.

### executor에서 발생한 동기 예외

```javascript
const p = new Promise(() => { throw new Error("setup failed"); });
p.catch(error => console.log(error.message)); // setup failed
console.log("constructed");
```

출력은 `constructed` 다음 `setup failed`다. 생성자는 executor의 동기 예외를 거부로 연결하지만 catch 후속 실행은 나중에 이루어진다. 반면 executor가 등록한 타이머 안의 미래 예외까지 같은 동기 포착 범위에 들어오는 것은 아니다.

executor의 return은 제어 흐름을 끝낼 뿐 성공 값을 전달하지 않는다. 콜백 API를 감쌀 때 성공은 resolve, 실패는 reject로 연결하고, 모든 완료 경로가 그 계약에 도달하는지 확인한다.

## 5. then이 만드는 새 Promise

```javascript
const first = Promise.resolve(2);
const second = first.then(value => value * 3);
console.log(first === second); // false
second.then(value => console.log(value)); // 6
```

`then`은 원본 Promise를 결과 `6`으로 바꾸는 것이 아니라 새 Promise를 반환한다. 원본의 성공 값은 `2`다. 후속 함수가 무엇을 반환하거나 던지는지에 따라 새 Promise의 결과가 정해진다.

| 후속 함수의 결과 | 다음 Promise |
| --- | --- |
| 일반 값 반환 | 해당 값으로 fulfilled |
| 반환 없이 종료 | `undefined`로 fulfilled |
| 예외 발생 | 해당 이유로 rejected |
| Promise·thenable 반환 | 그 최종 결과를 따름 |
| 해당 처리 함수가 없음 | 이전 성공·실패를 전달 |

```javascript
Promise.resolve(2)
  .then(value => Promise.resolve(value + 3))
  .then(value => { console.log("step", value); })
  .then(value => console.log("next", value));
```

출력은 `step 5`, `next undefined`다. 두 번째 후속 함수는 출력만 하고 반환하지 않았기 때문이다. 새로운 비동기 작업을 시작했다면 그 Promise를 `return`해야 다음 단계가 완료를 따를 수 있다.

### 연속 체인과 가지를 나눈 체인

```javascript
const root = Promise.resolve(2);
root.then(value => {
  console.log("A", value);
  return value + 1;
}).then(value => console.log("A2", value));
root.then(value => console.log("B", value));
console.log("S");
```

출력은 `S A 2 B 2 A2 3` 순서다. A와 B는 같은 root의 값 2를 각각 받는다. A2만 A가 반환한 새 Promise의 결과 3을 받는다. B는 A의 결과를 자동으로 기다리거나 전달받지 않는다.

[PerformPromiseThen](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-performpromisethen)은 반응을 연결하고 결과 Promise를 준비한다. 후속 처리가 큐에 들어갈 때는 앞 Promise의 상태도 중요하므로 소스의 then 호출을 모두 같은 시점의 큐 항목으로 펼치지 않는다.

## 6. 오류 전파와 복구

```javascript
Promise.resolve(1)
  .then(() => { throw new Error("failed"); })
  .then(value => console.log("skipped", value))
  .catch(error => {
    console.log(error.message); // failed
    return 0;
  })
  .then(value => console.log("recovered", value)); // recovered 0
```

실패는 성공 처리 단계를 건너뛰고 다음 실패 처리로 전파된다. `catch(handler)`는 `then(undefined, handler)`와 같은 역할이다. catch에서 일반 값을 반환하면 그 다음 체인은 성공으로 복구된다. 복구할 수 없으면 다시 던지거나 실패 Promise를 반환해야 한다.

`p.then(success, failure)`의 failure는 원본 p의 거부를 처리한다. 같은 `then`의 success 안에서 생긴 예외까지 형제 failure가 잡는 것은 아니다. 그 예외는 `then`이 반환한 Promise에 전달되므로 뒤의 catch가 처리해야 한다.

### 실패 처리의 위치가 바꾸는 후속 경로

```javascript
Promise.reject(new Error("first"))
  .catch(error => { console.log(error.message); throw new Error("second"); })
  .then(() => console.log("skipped"))
  .catch(error => console.log(error.message));
```

출력은 `first` 다음 `second`다. 첫 catch가 새 오류를 던졌으므로 체인은 여전히 실패 상태로 이어지고 성공 단계는 건너뛴다. catch가 일반 값을 반환하면 복구, 던지면 실패 전파라는 계약을 명시적으로 선택한다.

오류 로그를 한 번 남겼다는 사실과 실패 상태를 유지했다는 사실은 다르다. 처리할 수 없는 실패를 undefined 성공값으로 바꾸면 다음 단계에서 더 멀리 떨어진 오류가 발생할 수 있다. 반대로 대체 값이 업무상 유효하다면 복구를 통해 후속 흐름을 계속하는 것이 맞다.

## 7. finally와 콜백 API 감싸기

`finally`는 성공·실패와 관계없이 정리 작업을 이어 붙인다. 일반 값을 반환한다고 기존 결과를 바꾸지는 않지만, 예외를 던지거나 거부되는 Promise를 반환하면 새 실패로 바뀔 수 있다. 정리 실패도 최종 결과에 영향을 준다.

```javascript
function delayedValue(value) {
  return new Promise(resolve => {
    setTimeout(() => resolve(value), 0);
  });
}
delayedValue(5)
  .then(value => value + 1)
  .finally(() => console.log("cleanup"))
  .then(value => console.log(value)); // cleanup 다음 6
```

생성자는 콜백 기반 완료를 Promise로 연결할 때 유용하다. 이미 Promise를 반환하는 작업을 불필요하게 `new Promise`로 다시 감싸면 오류 전달 누락이 생기기 쉽다. 단순 값은 `Promise.resolve`, 실패는 `Promise.reject`로 표현할 수 있다.

### 콜백 API를 감쌀 때의 책임

일회성 콜백 API의 성공·실패를 각각 resolve·reject에 연결한다. 원본 API가 동기 throw와 비동기 error 인자 중 어느 실패 방식을 사용하는지 확인하고, 생성자만으로 포착되지 않는 미래 실패도 완료 계약으로 전달해야 한다.

finally는 대체 결과를 계산하는 곳보다 정리 작업을 연결하는 곳으로 사용한다. 정리가 Promise를 반환하면 그 완료를 기다리므로 정리 자체가 끝나야 뒤 체인으로 간다. 정리 실패가 원래 실패를 가릴 수 있다는 점도 설계에 포함한다. 필요한 맥락을 보존하는 방법은 Error Handling에서 더 다룬다.

## 8. 흔한 오해·예외·주의점

- executor를 `async` 함수로 쓰면 반환한 Promise를 생성자가 기다려 주지 않는다. 바깥 Promise가 pending으로 남거나 오류가 분리될 수 있다.
- executor가 나중에 실행하는 타이머 안에서 던진 예외는 생성자의 동기 executor 예외 포착과 다르다. 콜백의 실패를 `reject`로 연결해야 한다.
- 하나의 Promise에 `then`을 두 번 붙이면 독립된 두 후속 체인이다. 첫 체인의 반환을 둘째가 자동으로 기다리지 않는다.
- Promise가 실패했다고 외부 요청·타이머·다른 Promise가 자동 취소되지는 않는다.

### 떠 있는 Promise와 유지되는 반응

then 안에서 비동기 함수를 호출만 하고 반환하지 않으면 그 작업은 현재 체인의 완료·오류와 분리될 수 있다. 이를 흔히 떠 있는 Promise 문제라고 부른다. 다음 단계가 너무 일찍 진행하거나 외부 실패를 체인의 catch가 놓칠 수 있다.

끝나지 않는 Promise에 then을 계속 추가하면 등록된 후속 함수도 계속 유지될 수 있다. Promise가 언젠가 끝날 것이라는 기대만으로 무한 등록하지 않는다. 결과를 관찰하는 객체 역시 수명과 자원 비용을 갖는다.

## 9. 실무 적용과 디버깅 관점

체인의 각 단계에서 반환한 값의 종류를 적고, 시작만 하고 반환하지 않은 비동기 작업을 찾는다. 실패를 기록만 하고 catch가 정상 종료하면 복구한 것으로 처리된다는 점을 확인한다. 호스트의 미처리 거부 경고는 마지막 안전망이지 정상적인 오류 처리 설계가 아니다.

**[미리보기: async / await · Error Handling]** async/await는 Promise 기반 흐름을 문장 형태로 표현하며 실패는 await 경계에서 다룬다. 문법은 다음 async / await에서, 복구·재전파 정책은 Error Handling에서 다룬다.

### 체인 지도를 그려 검증한다

각 then·catch가 반환한 새 Promise에 번호를 붙이고 화살표를 그린다. 원본에 두 번 연결한 가지인지, 이전 then 결과에 이어 붙인 단계인지 확인한다. 각 함수의 반환을 일반 값·undefined·Promise·예외로 표시하면 다음 단계의 상태를 예측할 수 있다.

실행 순서 문제에서는 동기 구간, 이미 준비된 반응, 후속 결과가 확정된 뒤 준비되는 반응을 나눈다. 이 절차는 “then은 마이크로태스크”라는 분류만 외우는 것보다 실제 순서 분석에 유용하다.

## 10. 핵심 정리

executor는 지금 실행되고 후속 처리는 나중에 실행된다. then은 새 Promise를 만들며 반환과 예외로 결과를 연결한다. 상태·결과 전달·실제 작업의 실행과 취소를 각각 구분한다.

## 출처

- [ECMAScript — Promise Objects](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-promise-objects): 상태·반응·Job·결과 해석.
- [MDN — Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise): resolved와 settled.
- [MDN — Promise constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise): 동기 executor·완료 함수·반환값.
- [MDN — then](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then): 새 Promise·반환에 따른 결과.
- [MDN — Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises): 체인·반환 누락·실패 처리.

## 11. 이해도 확인 질문

1. 생성자 안의 출력·resolve·then·생성자 뒤 출력의 순서를 예측하고 실행 단계로 설명하라.
2. resolve에 pending Promise를 전달했을 때 resolved와 fulfilled를 왜 구분해야 하는가?
3. then 안에서 새 비동기 작업을 시작하고 반환하지 않으면 다음 단계가 무엇을 놓치는가?
4. catch가 출력만 한 뒤 종료하면 다음 then은 성공·실패 중 어느 경로로 가며 어떤 값을 받는가?
5. `p.then(success, failure)`가 success 내부의 예외를 형제 failure에서 처리하지 못하는 이유는 무엇인가?
