---
track: A
notion: https://app.notion.com/p/415178c4afac81c39856cfade6c4ebfd
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · Async JavaScript · Callback

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 콜백을 함수 전달과 호출 책임의 관점으로 설명한다.
- 동기 콜백과 비동기 콜백의 실행 순서를 구분한다.
- 콜백의 인자·반환값·호출 횟수·실패 계약을 읽는다.
- 즉시 호출, 반환값 혼동, this 소실, 중복 완료 버그를 찾는다.

## 2. 이 주제를 왜 배우는가

목록의 각 값을 어떻게 변환할지, 타이머가 끝난 뒤 무엇을 할지는 API를 사용하는 코드가 정해야 한다. API는 작업을 수행하고 사용자는 그 안에서 실행할 일부 동작을 함수로 전달한다. 이때 전달한 함수가 콜백이다.

**콜백(callback)** 은 다른 함수나 API에 인자로 넘겨 그쪽 코드가 호출하도록 하는 함수다. 문법의 종류가 아니라 맡은 역할이다. 선언문·함수 표현식·화살표 함수 모두 콜백으로 쓰일 수 있다.

### 제어를 넘긴다는 것의 계약

일반 함수를 직접 호출하면 호출자가 언제 어떤 인자로 실행할지 정한다. 콜백은 그 결정의 일부를 다른 API에 맡긴다. 따라서 코드를 사용하는 사람은 함수 본문뿐 아니라 API의 호출 횟수와 시점도 알아야 한다.

예를 들어 변환 콜백은 반환값이 결과 목록의 요소가 될 수 있지만 알림 콜백은 반환값이 무시될 수 있다. 같은 return 문도 어느 API가 받는지에 따라 의미가 달라진다. 함수 타입의 인자를 받는다는 사실만으로 구체적인 동작을 추측하지 않는다.

## 3. 함수 전달과 즉시 호출

```javascript
function run(action) {
  console.log("before");
  const result = action(3);
  console.log("after", result);
}
function double(n) { return n * 2; }
run(double);
```

출력은 `before`, `after 6`이다. `double` 함수 값을 전달했고 `run`이 원하는 시점에 호출했다. `run(double(3))`였다면 먼저 `double`을 호출한 숫자 `6`을 전달하여 `action(3)`에서 `TypeError`가 났을 것이다.

**호출 책임의 위임**이 핵심이다. 콜백을 받는 API가 어떤 인자로 언제 호출하는지, 반환값을 사용하는지 문서에서 확인해야 한다.

### 호출하는 쪽이 반환값을 사용할지 정한다

```javascript
function transform(value, callback) { return callback(value); }
function notify(value, callback) { callback(value); }
function double(n) { return n * 2; }
console.log(transform(3, double), notify(3, double)); // 6 undefined
```

double은 두 경우 모두 6을 반환했다. 그러나 notify는 그 값을 자신의 호출자에게 전달하지 않았으므로 최종 결과가 undefined다. 콜백의 반환과 바깥 API의 반환은 별도의 연결이다.

API 설계에서는 콜백이 계산값을 제공하는지, 흐름 중단 여부를 알리는지, 단순 알림인지 명시한다. 사용 시에도 반환한 값이 소비되는지 문서와 실제 호출 코드를 함께 읽어야 한다.

## 4. 동기 콜백과 비동기 콜백

```javascript
console.log("A");
const doubled = [1, 2].map(n => {
  console.log("map", n);
  return n * 2;
});
setTimeout(() => console.log("timer"), 0);
console.log("B", doubled.join(","));
```

```text
A
map 1
map 2
B 2,4
timer
```

`map`은 호출 중에 콜백을 실행하고 새 배열을 반환한다. 타이머는 후속 콜백을 예약하고 지금 반환한다. 따라서 콜백이 있다는 사실만으로 비동기라고 판단할 수 없다. 이미 배운 작업·큐 규칙과 API의 호출 계약을 함께 적용한다.

일부 사용자 정의 API가 캐시 유무에 따라 한 번은 동기, 한 번은 비동기로 완료 콜백을 부르면 호출자가 상태 준비 순서를 예측하기 어렵다. API를 설계할 때는 일정한 실행 시점을 정해 문서화하는 편이 좋다.

### 동기·비동기가 입력에 따라 섞이는 경우

캐시가 있으면 콜백을 즉시 부르고 없으면 나중에 부르는 API는 호출자에게 두 가지 순서를 요구한다. 콜백이 실행되기 전에 준비하려던 지역 상태가 어떤 경우에는 준비되지 않았을 수 있다. 이는 콜백이라는 방식 자체의 필수 속성이 아니라 일관되지 않은 호출 시점 계약의 문제다.

```javascript
let ready = false;
function immediate(done) { done(); }
immediate(() => console.log(ready)); // false
ready = true;
```

비동기라고 잘못 믿으면 true를 기대하기 쉽다. 반대로 API가 명확히 동기 콜백으로 동작한다면 위 결과는 정상이다. 콜백의 실행 시점은 함수 이름이나 데이터의 빠르기로 추측하지 않고 계약으로 확인한다.

## 5. 콜백 return이 돌아가는 곳

```javascript
function load(callback) {
  setTimeout(() => {
    const value = callback("data");
    console.log("callback returned", value);
  }, 0);
  return "scheduled";
}
const result = load(text => text.toUpperCase());
console.log("load returned", result);
```

출력은 `load returned scheduled` 다음 `callback returned DATA`다. 콜백이 반환한 `DATA`는 콜백을 호출한 타이머 본문으로 돌아간다. 이미 종료된 `load`의 반환값을 바꾸지 않는다.

같은 `return`이라도 어느 함수 본문에 있는지 표시해야 한다. 비동기 함수의 결과를 사용하려면 완료 시점의 흐름 안에서 이어 처리하거나 결과를 나타내는 별도 계약을 사용해야 한다.

### 바깥 함수 종료와 콜백 종료

타이머 본문의 return은 그 타이머 호출을 끝내며 이미 반환한 바깥 함수를 다시 종료시키지 않는다. 반복 API의 콜백에서 return해도 바깥 반복 자체가 종료된다고 일반화할 수 없다. 예를 들어 forEach의 return은 현재 콜백을 끝낼 뿐 일반적인 break 문을 대신하지 않는다.

중첩 코드에서 return을 분석할 때는 각 함수 몸체에 경계를 그린다. 그 return이 어느 호출 표현식에 값을 돌려주는지 연결하면 “여기서 return했는데 왜 아래 코드가 실행됐지”라는 질문의 대부분을 설명할 수 있다.

## 6. Error-first 계약

**error-first callback**은 첫 인자로 실패를, 두 번째 등으로 성공 결과를 전달하는 관례다. `Node.js에서` 흔하지만 JavaScript 문법이 강제하는 규칙은 아니다. 다음은 환경의 파일 API 대신 타이머로 흉내 낸 독립 예제다.

```javascript
function readValue(key, done) {
  setTimeout(() => {
    if (key !== "answer") {
      done(new Error("missing key"));
      return;
    }
    done(null, 42);
  }, 0);
}
readValue("answer", (error, value) => {
  if (error) {
    console.log("failed", error.message);
    return;
  }
  console.log("value", value); // value 42
});
```

실패 후 `return`하지 않고 아래 성공 호출도 실행하면 같은 작업이 두 번 완료된 것처럼 보인다. 일회성 완료 API라면 성공 또는 실패 중 하나를 한 번 전달하는 계약을 지켜야 한다. 이벤트 알림처럼 여러 번 호출하는 API는 또 다른 계약이다.

**[미리보기: Error Handling]** `Error` 객체는 실패의 종류·메시지·원인을 전달하는 데 쓴다. 던지기와 비동기 오류 경계의 세부 규칙은 Async JavaScript 마지막 Error Handling에서 다룬다.

### 일회 완료와 다회 알림은 다른 API다

error-first는 오류 위치의 관례이며, 콜백이 한 번 호출되는지까지 문법이 강제하지 않는다. 일회성 읽기 완료라면 성공 또는 실패 중 하나로 끝나야 하지만 진행률·이벤트 알림은 여러 번 호출하는 것이 정상일 수 있다. API 문서에서 호출 횟수와 종료 신호를 별도로 확인한다.

성공·실패 경로가 둘 다 실행되면 중복 저장이나 중복 후속 처리가 생길 수 있다. 이를 막는 일회 완료 보호는 단지 두 번째 결과를 무시하는 정책일 수 있으며, 원래 작업의 부수 효과를 되돌리는 것은 아니다. 완료 통지의 중복과 실제 작업의 중복을 나누어 조사한다.

[Node.js의 오류 전달 문서](https://nodejs.org/api/errors.html#error-propagation-and-interception)는 동기 throw와 콜백 기반 실패를 구분한다. 언어 예외와 error 인자로 전달하는 실패 값이 항상 같은 경로를 따른다고 생각하지 않는다.

## 7. this·추가 인자·중첩 흐름

메서드 `obj.handle`을 콜백으로 전달하면 API가 `obj.handle()`처럼 호출해 준다는 보장이 없다. 호출 계약에 따라 `this`가 사라질 수 있다. 필요한 경우 앞서 배운 `bind`나 래퍼 함수를 사용한다.

API는 예상보다 많은 인자를 줄 수도 있다. 예를 들어 배열 메서드는 값·인덱스·배열을 제공할 수 있다. 콜백에 다른 표준 함수를 그대로 넣기 전에 그 함수의 두 번째 인자가 무엇인지 확인해야 한다. `["10", "10"].map(parseInt)`에서 인덱스가 진법 인자로 전달되는 것은 대표적인 함정이다.

작업 A의 결과로 B를 시작하고 B의 결과로 C를 시작하면 중첩이 깊어진다. 각 단계를 이름 있는 함수로 추출하면 읽기는 나아지지만, 완료·실패 전달 계약을 일관되게 유지해야 한다.

### 콜백의 추가 인자가 만드는 버그

```javascript
const values = ["10", "10", "10"];
const wrong = values.map(parseInt);
const right = values.map(value => parseInt(value, 10));
console.log(wrong[0], Number.isNaN(wrong[1]), wrong[2]); // 10 true 2
console.log(right.join(",")); // 10,10,10
```

map은 값뿐 아니라 인덱스도 전달한다. parseInt의 두 번째 인자는 진법이므로 두 번째 요소에서는 1, 세 번째에서는 2가 전달되어 의도와 다른 해석이 된다. 래퍼로 필요한 인자만 받고 진법을 명시하면 계약을 맞출 수 있다.

함수 이름이 하는 일과 콜백으로서 받는 인자 목록을 함께 확인해야 한다. 메서드 전달의 this 소실과 마찬가지로, 직접 호출할 때의 계약을 다른 API가 그대로 보존해 주리라고 기대해서는 안 된다.

## 8. 흔한 오해·예외·주의점

- 콜백은 화살표 함수만 뜻하지 않고, 무조건 비동기도 아니다.
- 호출 결과를 넘기는 것과 함수 자체를 넘기는 것은 다르다.
- 콜백의 `return`이 바깥 API의 반환값이나 호출자의 실행을 종료하지 않는다.
- 전달한 함수가 한 번만 호출되는지 여러 번 호출되는지는 API마다 다르다.

### 콜백을 무시하거나 중복 호출하는 경로

일회성 완료 API를 구현할 때 모든 분기를 적어 본다. 어떤 분기에서도 호출하지 않으면 호출자는 완료를 영원히 기다릴 수 있고, 실패 통지 후 성공 통지까지 진행하면 두 번 완료된다. 콜백 타입만 맞는지 검사하는 것으로는 이 제어 흐름 계약을 검증하지 못한다.

오류를 받았다고 무조건 문자열 메시지를 읽는 것도 피한다. API의 오류 인자가 어떤 값인지와 null이 성공을 뜻하는지 확인한다. 오류 객체를 다루는 세부 처리·재전파는 뒤의 Error Handling에서 연결한다.

## 9. 실무 적용과 디버깅 관점

콜백을 받는 API마다 호출 시점, 인자 순서, 반환값 사용 여부, 횟수, 실패 경로, 취소·해제 방법을 확인한다. 중복 저장·중복 화면 갱신이 발생하면 완료 경로가 두 번 실행되는지 작업 ID와 횟수로 기록한다.

**[미리보기: Promise]** 일회성 비동기 완료를 값으로 표현하면 후속 동작과 실패를 체인으로 연결할 수 있다. 콜백 기반 API를 Promise로 감싸는 방식은 다음 Promise에서 다룬다.

### 콜백 기반 함수를 검증하는 질문

호출 횟수 0·1·여러 번을 각각 관찰하고, 성공·실패 입력 모두에서 기대한 경로가 실행되는지 확인한다. 동기·비동기 여부에 따라 바깥 준비 코드와 어느 순서로 실행되는지도 기록한다. 반환값을 사용하는 API라면 실제 소비한 결과까지 검사한다.

중첩이 깊으면 각 작업 단계를 이름 있는 함수로 나누고, 다음 단계를 시작하는 위치를 한 군데로 모은다. 깊이를 줄이는 것만으로 오류 계약이 해결되지는 않지만 성공·실패 뒤에 실행이 계속되는지 추적하기 쉬워진다.

### 통합 사례: 변환 콜백의 입력 범위를 맞춘다

문자열 숫자 목록을 정수로 바꾸는 기능에서 parseInt를 바로 전달하는 경우와 래퍼를 사용하는 경우를 비교한다. 단순히 함수가 숫자를 읽는다는 기능 이름만으로는 콜백 계약이 맞는지 판단할 수 없다.

```text
map의 첫 호출:  parseInt("10", 0, 원본배열)
map의 둘째 호출: parseInt("10", 1, 원본배열)
map의 셋째 호출: parseInt("10", 2, 원본배열)
```

두 번째 인자는 map에서는 인덱스지만 parseInt에서는 진법이다. 두 계약의 인자 의미가 충돌한 것이다. 래퍼 `value => parseInt(value, 10)`는 map이 주는 추가 인자를 받지 않고 원하는 진법을 명시하여 경계를 맞춘다.

this를 필요로 하는 메서드를 전달할 때도 같은 원리다. API는 그 함수가 원래 어느 객체의 속성이었는지를 복원해 주지 않는다. 수신자를 유지할 필요가 있으면 bind 또는 래퍼로 호출 계약을 구성한다.

검증 표에는 콜백이 받는 각 인자, 반환값 사용처, 호출 시점과 횟수를 적는다. 이 네 가지가 맞는지 확인하면 함수를 넘겼다는 문법적 사실만 보는 것보다 실패를 정확히 찾을 수 있다. 래퍼는 불필요한 코드가 아니라 서로 다른 두 함수 계약을 연결하는 작은 어댑터가 될 수 있다.

## 10. 핵심 정리

콜백은 실행할 함수를 전달해 호출 책임을 맡기는 역할이다. 동기·비동기 여부와 결과 전달 방식은 API 계약이다. 함수의 전달, 실행, 반환의 위치를 구분하면 대부분의 콜백 버그를 설명할 수 있다.

## 출처

- [MDN — Callback function](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function): 콜백 역할과 동기·비동기 비교.
- [Node.js — Errors](https://nodejs.org/api/errors.html#error-propagation-and-interception): error-first 콜백과 오류 전달.
- [MDN — Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions): 함수 값과 인자 전달.

## 11. 이해도 확인 질문

1. `schedule(run)`과 `schedule(run())`은 호출 순서와 전달값이 어떻게 다른가?
2. `map`의 콜백과 타이머의 콜백이 실행되는 시점을 비교하라.
3. 타이머 안에서 반환한 값이 바깥 함수의 반환값이 되지 않는 이유는 무엇인가?
4. error-first 콜백의 실패 경로 뒤에 `return`이 없으면 어떤 중복 완료 버그가 생길 수 있는가?
5. 메서드를 콜백으로 넘기기 전에 this와 인자 계약에서 무엇을 확인해야 하는가?
