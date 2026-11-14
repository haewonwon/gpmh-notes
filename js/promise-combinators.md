---
track: A
notion: https://app.notion.com/p/415178c4afac8121b51df726931d821f
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · Async JavaScript · `Promise.all` / allSettled / race

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- all·allSettled·race의 완료 조건과 실패 계약을 비교한다.
- 입력 순서·완료 순서·후속 실행 시점을 구분한다.
- 빈 입력·일반 값·함수 값·영구 pending의 예외를 판단한다.
- 조기 실패와 취소, 타임아웃과 실제 작업 중단의 차이를 설명한다.
- 부분 실패·독립 작업·동시 실행량 요구에 맞는 조합을 선택한다.

## 2. 이 주제를 왜 배우는가

화면 하나에 사용자·권한·추천 목록이 필요하다고 하자. 모두 성공해야 하는지, 추천만 실패해도 화면을 보여 줄지, 가장 먼저 온 응답만 사용할지에 따라 완료 조건이 다르다. 세 API는 여러 결과를 하나의 Promise로 묶되 “어떤 결과를 언제 돌려줄지”가 다르다.

**Promise 조합(composition)** 은 여러 Promise·값을 입력받아 하나의 결과 계약으로 연결하는 것이다. 이 API들은 보통 이미 시작된 작업의 결과를 관찰한다. 함수 목록을 주었다고 자동으로 실행하거나 실행 스레드를 늘려 주지 않는다.

### 결과 정책을 먼저 글로 적는다

동시에 요청한다는 것은 결과 정책이 아니다. 필요한 성공 조건이 “세 개 전부”, “각 결과를 개별 표시”, “첫 확정 결과 하나” 중 무엇인지 먼저 정한다. 같은 작업 목록이라도 이 조건에 따라 다음 단계에 전달할 데이터 형식이 달라진다.

독립된 작업이라는 말도 완료 수집 방식과 구분한다. 두 작업을 함께 시작할 수 있어도 화면을 만들기 위해 둘 다 필요하다면 all이 적합할 수 있다. 반대로 별도 카드라면 각 실패를 보존하는 allSettled가 적합할 수 있다. 실행 의존과 업무상 성공 조건은 서로 다른 축이다.

## 3. 세 API의 계약 비교

| 기준 | `Promise.all` | `Promise.allSettled` | `Promise.race` |
| --- | --- | --- | --- |
| 완료 기준 | 전부 성공 또는 한 입력 실패 | 모든 입력의 성공·실패가 확정 | 처음 전달된 성공 또는 실패 |
| 성공 결과 | 성공 값의 배열 | 각 입력의 상태 기록 배열 | 먼저 확정된 한 값 |
| 입력 작업 실패 | 바깥 Promise 거부 | 상태 기록에 저장 | 먼저 확정된 입력이 실패면 거부 |
| 배열의 순서 | 입력 순서 | 입력 순서 | 배열을 반환하지 않음 |
| 다른 작업 취소 | 하지 않음 | 하지 않음 | 하지 않음 |
| 빈 입력 | 이행된 `[]` | 이행된 `[]` | 계속 pending |

**settled**는 fulfilled 또는 rejected로 상태가 확정됐다는 뜻이다. race는 “먼저 성공한 것”이 아니라 “먼저 성공 또는 실패가 전달된 것”을 따른다. 세 API의 바깥 결과와 각 작업의 상태를 분리해서 읽어야 한다.

### 결과 Promise와 개별 작업 상태를 분리한다

조합이 반환한 Promise는 개별 작업들의 위에 놓이는 관찰 계약이다. 바깥 결과가 실패했다고 모든 입력이 실패한 것도 아니고, 바깥 결과가 성공했다고 입력이 전부 성공한 것도 아니다. allSettled의 성공 값에는 실패 기록이 들어갈 수 있다.

| 상황 | 개별 작업 상태 | 조합의 결과 판단 |
| --- | --- | --- |
| all에서 하나 거부 | 나머지는 pending·fulfilled 등일 수 있음 | 전체 결과는 거부 |
| allSettled가 이행 | 입력별 fulfilled 또는 rejected | 기록 배열을 성공 값으로 전달 |
| race가 이행 | 다른 입력은 아직 진행 중일 수 있음 | 처음 관찰한 성공 값 전달 |

명세의 [Promise 생성자 정적 메서드](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-properties-of-the-promise-constructor)는 입력을 순회하고 각 결과를 연결하는 절차를 정한다. 다른 작업에 중단 명령을 보내는 알고리즘은 여기에 자동 포함되지 않는다.

## 4. all: 전부 필요한 결과 모으기

```javascript
function wait(value, ms) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}
const slow = wait("user", 20);
const fast = wait("permissions", 0);
Promise.all([slow, fast, 3]).then(values => {
  console.log(values.join(",")); // user,permissions,3
});
```

작업은 `wait`를 호출할 때 시작됐다. fast가 먼저 완료되어도 결과 배열의 인덱스는 입력 순서에 대응한다. 일반 값 `3`도 성공 값으로 처리된다. 전체 시간은 순차 대기 합이 아니라 실제 작업들이 겹쳐 진행한 상태와 마지막 필수 완료 시점의 영향을 받는다.

all은 각 입력 결과를 저장하고 남은 완료를 추적한다. 모두 성공해야 결과 배열을 제공한다. 어떤 입력의 거부가 먼저 전달되면 바깥 Promise를 거부하며 다른 성공 결과를 최종 배열로 제공하지 않는다.

### 입력 위치와 완료 시점의 별도 기록

all은 결과마다 대응할 위치를 유지하여 성공 값을 채운다. 따라서 두 번째 작업이 먼저 끝나도 두 번째 칸으로 들어간다. 이 계약 덕분에 `[user, permissions]`처럼 입력 의미를 위치와 연결할 수 있지만, 입력 순서를 바꾸고 소비 코드를 그대로 두면 다른 값을 잘못 해석할 수 있다.

업무에서는 결과 위치 외에 작업 ID도 유지하면 좋다. 동적으로 만든 작업 목록이 필터링되거나 순서가 바뀌는 경우에는 위치만으로 어떤 요청의 결과인지 판단하기 어렵다. 결과 수집 방식과 데이터 식별 방식은 함께 설계한다.

## 5. all의 조기 실패는 작업 취소가 아니다

```javascript
const slow = new Promise(resolve => {
  setTimeout(() => {
    console.log("slow still finished");
    resolve("slow");
  }, 20);
});
const failed = Promise.reject(new Error("unavailable"));
Promise.all([slow, failed]).catch(error => console.log(error.message));
```

출력은 `unavailable` 다음 `slow still finished`다. 바깥 결과는 이미 실패했지만 slow의 타이머는 중단되지 않았다. 다른 작업이 저장·전송 같은 효과를 수행했다면 그 효과도 자동으로 롤백되지 않는다.

all의 조기 실패는 **관찰하는 결과를 빨리 확정하는 정책**이다. 전체 요청 취소나 트랜잭션의 원자성과 같지 않다. 모든 실패 기록이 필요하면 all의 catch 하나만으로는 입력별 결과를 다 얻지 못한다.

### 먼저 관찰된 실패만으로 전체 원인을 다 알 수 없다

all의 거부 이유 하나는 해당 조합이 먼저 전달받은 실패다. 다른 입력이 나중에 실패할 수도 있으므로 그 이유 하나가 시스템의 유일한 문제라고 단정하지 않는다. 동시에 발생한 실패를 모두 기록해야 한다면 개별 작업 기록 또는 allSettled 같은 수집 정책이 필요하다.

또한 이미 외부에 반영한 효과는 Promise 결과만 되돌려서 취소되지 않는다. 요청 취소가 가능한 경우에도 서버가 처리했는지 모르는 구간이 있을 수 있다. 조합의 완료와 업무 효과의 원자성을 별도 문제로 구분해야 재시도·중복 실행 정책을 세울 수 있다.

## 6. allSettled: 부분 실패를 데이터로 수집하기

```javascript
Promise.allSettled([
  Promise.resolve("profile"),
  Promise.reject(new Error("recommendations failed"))
]).then(results => {
  for (const result of results) {
    if (result.status === "fulfilled") {
      console.log("ok", result.value);
    } else {
      console.log("failed", result.reason.message);
    }
  }
});
```

출력은 `ok profile`, `failed recommendations failed`다. 성공 기록에는 `status: "fulfilled"`와 `value`, 실패 기록에는 `status: "rejected"`와 `reason`이 있다. 실패 기록의 value를 무조건 읽으면 `undefined`를 정상 데이터로 오해할 수 있다.

일반적인 입력 Promise의 거부 때문에 바깥 allSettled가 거부되는 것은 아니다. 하지만 잘못된 iterable이나 순회 도중 예외 등 조합 자체의 실패까지 없다는 뜻은 아니다. 또한 하나가 영원히 pending이면 전체도 끝나지 않는다.

API가 완료했다고 업무상 성공한 것도 아니다. HTTP 응답 코드나 응답 본문의 성공 규칙은 작업을 만드는 쪽에서 판단해 성공·실패를 표현해야 한다. 호스트 fetch의 세부 규칙은 Track B의 fetch 문서와 구분한다.

### 상태 기록을 업무 결과로 해석한다

allSettled 결과를 받은 뒤에는 status로 분기하고 각 기록을 원래 작업과 연결한다. 실패한 기록을 단순히 버리고 성공 값만 모으면 어떤 입력이 빠졌는지 정보가 사라진다. 일부 성공 화면이라면 성공 데이터와 실패 설명을 함께 반환하는 형태를 고려한다.

입력 하나가 끝나지 않으면 allSettled도 끝나지 않는다. “모든 실패를 허용한다”와 “언제나 빠르게 끝난다”는 다른 말이다. 개별 작업의 완료·타임아웃 계약이 없으면 바깥 조합만 바꾸어 무한 대기를 해결할 수 없다.

## 7. race: 처음 확정된 결과 선택하기

```javascript
const slowSuccess = new Promise(resolve => {
  setTimeout(() => resolve("ok"), 20);
});
const fastFailure = Promise.reject(new Error("fast failure"));
Promise.race([slowSuccess, fastFailure])
  .then(value => console.log(value))
  .catch(error => console.log(error.message)); // fast failure
```

빠른 실패도 race의 승자가 된다. “성공하는 응답 하나면 된다”는 요구에 race를 그대로 고르면 실패 정책이 어긋날 수 있다. 이 페이지 범위는 all·allSettled·race의 차이를 정확히 선택하는 데 둔다.

이미 확정된 Promise나 일반 값이 여럿이면 각 입력의 후속 처리가 연결되는 순서도 중요하다. “과거 실제 시간상 가장 먼저 완료된 값”을 알아내는 기록 장치가 아니다. 조합하기 전에 끝난 입력들이라면 입력 순서대로 관찰 반응이 등록될 수 있다.

### race는 과거 완료 시각을 비교하지 않는다

```javascript
const firstFinished = Promise.resolve("finished earlier");
const secondFinished = Promise.resolve("listed first");
Promise.race([secondFinished, firstFinished])
  .then(value => console.log(value)); // listed first
```

두 입력 모두 조합 이전에 이행되어 있다. race는 내부에 저장된 실제 완료 시각을 비교하는 대신 각 입력의 반응을 연결하여 먼저 전달되는 결과를 따른다. 이 예제에서는 입력에 나열한 순서가 관찰 순서에 영향을 준다.

따라서 과거 작업의 실제 소요 시간 순위를 알고 싶다면 별도 시각·측정값을 기록해야 한다. race는 순위 분석 API가 아니라 첫 확정 결과를 따르는 조합 API다.

## 8. 빈 입력과 비동기 후속 처리

```javascript
Promise.all([]).then(values => console.log("all", values.length));
Promise.allSettled([]).then(values => console.log("settled", values.length));
Promise.race([]).then(() => console.log("never"));
console.log("sync");
```

출력은 `sync`, `all 0`, `settled 0`다. race의 후속 출력은 생기지 않는다. all·allSettled의 빈 입력 결과는 이미 fulfilled여도 then은 동기 본문 중간에 실행되지 않는다.

빈 배열을 “할 일이 없으니 곧 성공”으로 읽는 것은 race에는 맞지 않는다. 동적으로 만든 작업 목록을 race에 넣기 전에 빈 입력 정책을 정한다. pending Promise만 존재한다고 Node.js 프로세스가 계속 살아 있어야 하는 것도 아니다. 실제 활성 작업 여부는 호스트의 종료 규칙을 따른다.

### 빈 작업 목록의 업무 의미

동적으로 생성한 작업이 0개일 때 all·allSettled의 성공 빈 배열은 자연스러울 수 있다. 반면 필수 요청을 만드는 코드가 잘못되어 빈 목록이 됐다면 그 성공은 오류를 숨길 수 있다. 빈 입력을 허용하는지 호출 경계에서 확인한다.

race의 빈 입력은 결과를 확정해 줄 입력이 없으므로 계속 pending이다. 이 상태를 “작업 없음”으로 표현할지 오류로 처리할지 명시해야 한다. 이미 이행된 빈 결과에 then을 붙여도 후속 함수가 동기적으로 실행되지 않는다는 규칙은 그대로 적용된다.

## 9. 함수 목록과 작업 시작 시점

```javascript
function loadA() { console.log("start A"); return Promise.resolve("A"); }
function loadB() { console.log("start B"); return Promise.resolve("B"); }
Promise.all([loadA, loadB]).then(values => {
  console.log(typeof values[0]); // function
});
```

함수들은 일반 값으로 들어갔으므로 호출되지 않는다. 실행하려면 `[loadA(), loadB()]`처럼 호출한 반환값을 넘기거나, 명시적으로 각 함수를 호출하는 변환을 해야 한다. 인자 표현식을 평가하는 중 작업은 이미 시작될 수 있다.

호출 자체가 동기 예외를 던지면 all을 호출하기 위한 배열을 만드는 단계에서 실패할 수도 있다. 모든 오류가 조합 API 내부로 들어간다고 가정하지 않는다. 시작 함수를 `Promise.resolve().then(start)` 같은 후속 단계에서 호출하면 그 단계의 예외는 Promise 거부로 연결할 수 있다.

### 시작 함수의 예외를 포함할 경계

```javascript
function start() { throw new Error("start failed"); }
Promise.all([Promise.resolve().then(start)])
  .catch(error => console.log(error.message)); // start failed
console.log("registered");
```

출력은 `registered`, `start failed`다. start는 후속 반응에서 실행되어 예외가 해당 Promise의 거부로 연결된다. 반대로 `Promise.all([start()])`에서는 all 호출에 들어가기 전 배열 요소를 계산하다 예외가 발생한다.

이 패턴은 호출 시점도 미루므로 무조건적인 치환 규칙은 아니다. 시작 함수가 동기적으로 해야 할 효과가 있는지 확인하고, 실패를 어느 완료 계약으로 연결하려는지에 따라 경계를 선택한다.

## 10. 타임아웃·취소·동시 실행량

타임아웃 Promise와 실제 작업을 race하면 기다리는 결과에 시간 제한을 둘 수 있다. 그러나 패한 작업은 계속 진행한다. 아래는 외부 통신 없이 동작하는 실험이다.

```javascript
const work = new Promise(resolve => {
  setTimeout(() => { console.log("work completed"); resolve("done"); }, 30);
});
let timeoutId;
const timeout = new Promise((resolve, reject) => {
  timeoutId = setTimeout(() => reject(new Error("timeout")), 5);
});
Promise.race([work, timeout])
  .then(value => console.log(value))
  .catch(error => console.log(error.message))
  .finally(() => clearTimeout(timeoutId));
```

`timeout` 다음 `work completed`가 출력된다. 타이머 정리는 기다리던 타임아웃만 해제하며 work까지 중단시키지 않는다. 실제 작업을 취소하려면 그 API가 제공하는 취소 계약이 필요하다. `AbortController`는 지원 API에 취소 신호를 전달하는 호스트 기능이며 모든 Promise에 자동 적용되는 언어 기능은 아니다.

대량 작업을 `items.map(start)`로 만들고 all에 넘기면 이미 모두 시작됐을 수 있다. all은 동시 실행량을 제한하는 작업 스케줄러가 아니다. 요청 수 제한이 있다면 작업을 꺼내 시작하는 계층에서 별도 제한을 둔다.

### 종료 조건과 정리 책임

타임아웃 Promise의 타이머는 실제 작업이 먼저 끝났을 때 정리해야 불필요한 자원이 남지 않는다. 그러나 타이머를 지운다고 작업 API의 취소가 된 것은 아니다. 반대로 취소를 요청해도 그 실패 이유를 어떻게 소비할지 별도 처리해야 할 수 있다.

실행량 제한도 결과 조합 앞의 시작 계층에서 적용한다. 이미 수천 개의 작업을 시작한 뒤 그 Promise들을 묶어도 동시 요청 수는 줄어들지 않는다. 순차 처리·묶음 시작·별도 작업 제한 중 무엇이 필요한지는 외부 용량과 완료 요구를 기준으로 선택한다.

## 11. 실무 선택과 디버깅 관점

- 모두 성공해야 결과를 쓸 수 있으면 all을 고려한다.
- 일부 실패를 기록하면서 나머지 결과도 써야 하면 allSettled를 고려한다.
- 성공·실패를 포함해 먼저 확정된 결과 하나가 정책이면 race를 고려한다.

각 입력의 ID·시작·완료·상태와 바깥 조합의 완료를 따로 기록한다. 빈 목록, 한 입력 실패, 끝나지 않는 입력, 입력과 완료 순서의 역전을 재현한다. 조합의 거부 후에도 부수 효과가 계속되는지 확인해야 중복 저장과 자원 낭비를 조사할 수 있다.

### 최소한의 검증 시나리오

정상 입력만 검사하지 말고 완료 순서가 뒤집힌 입력, 하나의 거부, 여러 거부, 영구 pending, 빈 목록을 각각 확인한다. 값 배열뿐 아니라 개별 작업이 계속 수행하는 효과와 바깥 결과의 완료 시점을 따로 기록한다.

검증 기대값은 선택한 조합에 따라 달라진다. all에서는 첫 거부 전파와 성공 배열의 입력 순서, allSettled에서는 상태별 필드와 모든 완료, race에서는 빠른 실패와 남은 작업의 계속 진행을 확인한다. 세 API가 모두 같은 Promise를 반환한다는 사실만으로 테스트를 하나로 합치지 않는다.

## 12. 핵심 정리

세 API는 결과를 묶는 완료 계약이다. all과 allSettled의 배열은 입력 순서를 유지하며 race는 첫 성공 또는 실패를 따른다. 조기 확정은 취소·롤백·실행량 제한과 다르다. 빈 입력과 영구 pending도 설계에 포함한다.

## 출처

- [MDN — Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all): 입력 순서·조기 실패·빈 입력.
- [MDN — Promise.allSettled](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled): 결과 기록과 모든 입력의 완료.
- [MDN — Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race): 첫 확정 결과·빈 입력·비동기 전달.
- [ECMAScript — Promise constructor properties](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-properties-of-the-promise-constructor): 조합 알고리즘·iterable 처리.
- [MDN — AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController): 호스트 작업의 취소 신호.

## 13. 이해도 확인 질문

1. 세 요청이 서로 다른 순서로 끝날 때 all과 allSettled의 결과 배열 순서는 무엇을 따르는가?
2. all이 실패한 뒤 다른 요청의 저장 작업이 실행됐다. 왜 가능한지와 필요한 별도 정책을 설명하라.
3. 필수 프로필과 선택 추천을 함께 읽을 때 부분 실패 결과가 필요하다면 어떤 조합과 후속 분기를 선택할 것인가?
4. race에 빈 배열, 함수 두 개, 이미 확정된 Promise 두 개를 각각 넘길 때 무엇을 확인해야 하는가?
5. race 타임아웃을 추가했는데 네트워크 사용과 부수 효과가 남는다. 결과 확정·취소·동시 실행량 제한의 책임을 나누어 설명하라.
