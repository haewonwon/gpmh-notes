---
track: B
notion: https://app.notion.com/p/415178c4afac81a89e9ffa2cbc86a2be
---

# preventDefault / stopPropagation


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · preventDefault / stopPropagation

기본 동작의 취소와 이벤트 전달의 중단을 각각 이해하고 필요한 흐름만 제어한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 기본 동작·리스너 실행·전파를 서로 구별할 수 있다.
- preventDefault·stopPropagation·stopImmediatePropagation의 결과를 예측할 수 있다.
- cancelable·passive·호출 위치·비동기 시점의 제약을 설명할 수 있다.
- 폼·링크에서 사용자 기본 조작과 대체 흐름을 함께 설계할 수 있다.
- 취소 플래그와 단계별 로그로 문제를 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

링크를 눌렀을 때 페이지가 이동하는 일과 부모의 click 리스너가 실행되는 일은 같은 클릭에서 일어나지만 서로 다른 현상이다. 전파를 막아도 이동할 수 있고, 이동을 취소해도 부모 리스너는 실행될 수 있다. 이 구분이 없으면 의도치 않은 동작을 다른 방식으로 막다가 더 큰 문제를 만든다.

---

## 3. 관련 계층과 역할 구분

HTML은 링크 이동·폼 제출 같은 기본 동작을 제공한다. DOM 이벤트 모델은 리스너의 경로와 취소·중단 플래그를 관리한다. JavaScript 리스너는 어떤 흐름을 제어할지 선택한다.

기본 동작을 취소하면 해당 작업의 로딩·오류·접근성 안내를 애플리케이션이 책임질 수 있다. 프레임워크의 콜백 반환 관례가 DOM의 기본 규칙을 자동으로 바꾸는 것은 아니다.

---

## 4. 두 개의 흐름을 구분한다

기본 동작은 링크 이동, 폼 제출, 체크박스 선택 전환처럼 이벤트와 연결된 브라우저의 동작이다. 전파는 이벤트를 타깃과 조상의 리스너로 전달하는 과정이다. 정확한 기본 동작 시점은 이벤트와 컨트롤에 따라 달라지므로 모든 기본 동작이 리스너 실행 뒤에만 시작된다고 단정하지 않는다.

| API | 제어하는 것 | 그대로 남는 것 |
| --- | --- | --- |
| `preventDefault()` | 취소 가능한 이벤트의 기본 동작 | 다른 리스너로의 전파 |
| `stopPropagation()` | 이후 경로로의 전파 | 기본 동작과 같은 대상의 다른 리스너 |
| `stopImmediatePropagation()` | 전파와 같은 대상의 이후 리스너 실행 | 기본 동작 |

`preventDefault()`는 이벤트의 `cancelable`이 true이고 passive 리스너가 아닌 상황에서 취소 플래그를 세운다. `defaultPrevented`로 취소 상태를 확인할 수 있다. [DOM 표준](https://dom.spec.whatwg.org/#interface-event)에 이 동작이 정의되어 있다.

---

## 5. 기본 동작·리스너·전파를 각각 예상하기

### 기본 동작은 브라우저가 제공하는 의미와 연결된다

링크 클릭의 이동, submit 버튼의 폼 제출, 체크박스의 선택 전환은 HTML 요소가 제공하는 동작이다. 리스너는 그 동작과 관련된 이벤트를 관찰·처리하는 JavaScript 코드다. click 리스너를 달았다고 브라우저가 기존 동작을 자동으로 없애 주지 않는다.

반대로 일반 div에 click 리스너를 달았다고 링크 이동이나 버튼의 모든 키보드 동작이 생기는 것도 아니다. 어떤 HTML 요소를 썼는지와 어떤 이벤트를 제어하는지를 함께 본다.

### 취소 상태와 전파 상태는 별개의 상태다

preventDefault는 기본 동작 취소 플래그에, stopPropagation은 전파 중단 플래그에 영향을 준다. 한 메서드를 호출했다고 다른 플래그가 함께 정해지는 것이 아니다. 기본 동작이 전혀 없는 이벤트라도 cancelable로 구성하여 애플리케이션이 취소 결과를 이용할 수 있지만, 그것만으로 새로운 브라우저 기본 동작이 생기지는 않는다.

### 어느 위치에서 중단하는지가 중요하다

조상의 캡처 리스너에서 전파를 중단하면 이벤트가 타깃 리스너까지 도달하지 못할 수 있다. 타깃의 비캡처 리스너에서 중단하면 이미 실행된 조상 캡처 리스너는 되돌아가지 않는다. 같은 메서드 이름이어도 호출 위치에 따라 남는 실행이 다르다.

| 호출 위치와 제어 | 일반적인 이후 관찰 |
| --- | --- |
| 조상 캡처에서 stopPropagation | 같은 대상의 다른 리스너는 실행될 수 있으나 이후 경로는 중단 |
| 타깃에서 stopPropagation | 같은 대상의 다른 리스너는 계속될 수 있고 조상 버블링은 중단 |
| 타깃에서 preventDefault | 전파는 계속되며 취소 가능한 기본 동작은 취소 |
| 타깃에서 stopImmediatePropagation | 같은 대상의 이후 리스너와 이후 경로 중단 |

이 비교는 호출 전에 이미 실행된 리스너까지 취소하는 설명이 아니다. 등록 순서와 현재 단계를 함께 추적한다.

---

## 6. 예제: 링크의 이동과 부모 클릭을 각각 제어하기

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>이벤트 제어</title></head>
<body>
  <h1>기본 동작과 전파</h1>
  <div id="panel"><a id="link" href="#destination">목적지로 이동</a></div>
  <p id="destination">목적지</p>
  <script>
    const panel = document.querySelector('#panel');
    const link = document.querySelector('#link');

    panel.addEventListener('click', () => console.log('부모 실행'));
    link.addEventListener('click', (event) => {
      event.preventDefault();
      console.log('링크 첫 리스너', event.defaultPrevented);
    });
    link.addEventListener('click', () => console.log('링크 둘째 리스너'));
  </script>
</body>
</html>
```

현재 코드는 두 링크 리스너와 부모 리스너가 실행되지만 해시 이동은 취소된다. `preventDefault()`를 `stopPropagation()`으로 바꾸면 링크 둘째 리스너는 실행되고 부모의 버블링 리스너는 실행되지 않는다. 이동은 취소하지 않았으므로 주소의 해시가 바뀔 수 있다. `stopImmediatePropagation()`으로 바꾸면 링크 둘째 리스너도 실행되지 않지만 이동 취소는 별개다.

이미 실행된 조상의 캡처 리스너를 뒤늦게 되돌릴 수는 없다. stopPropagation을 “부모의 모든 코드를 무효화하는 기능”으로 이해하면 안 된다.

---

## 7. 취소 가능성을 직접 확인하는 작은 실험

```javascript
const target = new EventTarget();
target.addEventListener('before-study', (event) => {
  event.preventDefault();
  console.log(event.cancelable, event.defaultPrevented);
});

const first = target.dispatchEvent(new Event('before-study'));
const second = target.dispatchEvent(new Event('before-study', { cancelable: true }));
console.log(first, second);
```

기본 Event는 cancelable이 false이므로 첫 호출에서 defaultPrevented가 true가 되지 않는다. 둘째는 취소 가능하게 만들었으므로 취소 플래그가 설정된다. dispatchEvent의 boolean 결과는 취소 여부를 반영한다. 이 코드는 취소 프로토콜을 관찰하는 예제이며 실제 폼 제출을 발생시키지 않는다.

### 취소 후에 이어질 일을 명시한다

브라우저의 폼 제출을 취소하고 사용자 코드로 처리했다면 로딩·실패·중복 제출·성공 안내도 그 처리 흐름에서 설계해야 한다. 링크 이동을 취소했다면 사용자가 링크의 주소를 복사하거나 새 탭으로 여는 기대와 충돌하지 않는지도 확인한다. 기본 동작을 없앤다는 사실만으로 대체 UX가 완성되지는 않는다.

---

## 8. 폼에서는 제출 이벤트를 기준으로 처리한다

폼의 기본 제출을 JavaScript 처리로 바꿀 때는 보통 form의 `submit` 이벤트에서 동기적으로 `preventDefault()`를 호출한다. 클릭 리스너만 다루면 Enter 제출 같은 경로를 놓칠 수 있다. 비동기 작업을 `await`한 뒤 취소하려고 하면 이벤트 전달과 기본 동작의 시점을 놓칠 수 있으므로 취소 여부를 먼저 결정한다.

브라우저 검증에 실패하면 일반적인 submit 이벤트에 도달하지 않을 수 있다. 또한 `form.submit()`은 submit 이벤트와 제약 검증을 우회하는 동작이므로 사용자 제출과 같은 경로가 필요할 때는 `requestSubmit()`의 동작을 확인한다.

---

## 9. passive와 취소 가능 여부

passive 리스너는 기본 동작을 취소하지 않겠다는 약속이다. 해당 리스너에서 `preventDefault()`를 호출해도 취소되지 않으며 경고가 나올 수 있다. 특히 스크롤과 관련된 이벤트는 대상·브라우저에 따라 passive 기본 정책을 확인해야 한다. `{ passive: false }`를 지정하는 것은 정말 취소가 필요한 경우에 한정한다. 불필요한 취소 가능 리스너는 스크롤 반응성에 영향을 줄 수 있다.

`return false`를 일반적인 `addEventListener` 콜백에서 반환해도 이 두 API를 대신하지 않는다. 특정 라이브러리나 HTML 이벤트 핸들러의 관례를 DOM API 전체의 규칙으로 일반화하지 않는다.

---

## 10. passive·비동기·라이브러리 관례에서 생기는 오류

### passive는 취소하지 않겠다는 약속이다

스크롤 관련 입력에서 브라우저는 취소 여부를 기다려야 할 수 있다. passive 리스너는 취소하지 않는다고 알리므로 이 대기를 줄일 수 있다. 브라우저별 기본 정책을 확인하고 실제 취소가 필요할 때만 non-passive로 등록한다. 취소가 필요하지 않은 관찰 코드에 무조건 passive false를 붙이지 않는다.

### await 뒤의 preventDefault는 너무 늦을 수 있다

```javascript
async function handleSubmit(event) {
  event.preventDefault(); // 현재 제출의 기본 동작 취소는 먼저 결정
  const form = event.currentTarget;
  const data = new FormData(form);
  // 이후 비동기 검증·요청을 수행하고 성공·실패를 처리한다.
  console.log([...data.entries()]);
}
```

이 함수는 form의 submit 리스너로 등록하는 조각이다. 비동기 작업 후에 event.currentTarget을 다시 읽는 대신 현재 form을 먼저 확보한다. 취소를 먼저 했다는 이유로 서버 검증과 실제 전송 처리가 생기는 것은 아니다.

### return false의 의미는 호출 환경마다 다르다

DOM addEventListener는 콜백의 false 반환값을 기본 취소나 전파 중단으로 처리하지 않는다. HTML 인라인 핸들러나 특정 라이브러리는 다른 관례를 가질 수 있다. 자신이 사용하는 등록 API의 규칙을 확인하고 필요한 제어 메서드를 명시한다.

### 부모가 이벤트를 관찰해야 하는 요구도 있다

자식의 모든 이벤트를 막으면 위임 처리, 팝오버 바깥 클릭 감지, 상위 키보드 처리 등이 실패할 수 있다. 실제로 충돌하는 기능의 책임 범위를 먼저 정한다. DOM 구조·이벤트 종류·상태 판정으로 해결할 수 있는 문제인지 확인한 뒤 전파 중단을 사용한다.

---

## 11. 중단할 흐름을 세 개의 질문으로 구별하기

### 기본 동작은 무엇인가

링크의 주소 이동, 폼 제출, 체크박스 선택은 HTML 컨트롤이 제공하는 동작이다. 현재 코드에 연결된 동작이 실제로 무엇인지 먼저 적는다. 같은 click이라도 button type과 form 문맥에 따라 제출이 일어날 수도 있고 별도 기본 작업이 없을 수도 있다. 이벤트 이름 하나만 보고 취소할 작업을 추정하지 않는다.

### 어느 리스너가 이후에 실행될 수 있는가

캡처·타깃·버블링은 이벤트를 관찰할 경로다. 부모의 분석 리스너가 실행되는 것과 링크 이동은 별개다. 둘 중 하나만 막을 수도 있고 둘 다 막아야 할 수도 있다. 이 선택을 하지 않은 채 preventDefault와 stopPropagation을 습관적으로 함께 쓰면 불필요한 기능까지 막을 수 있다.

### 취소·중단이 가능한 시점과 등록 조건인가

cancelable은 이벤트가 취소 플래그를 가질 수 있는지, passive는 현재 리스너가 취소하지 않겠다고 약속했는지와 관계한다. stopPropagation은 이미 호출을 끝낸 리스너를 되돌리는 시간 역행 기능이 아니다. [DOM Event의 플래그와 메서드](https://dom.spec.whatwg.org/#interface-event)를 이 세 질문의 근거로 읽는다.

---

## 12. 같은 사건에 세 메서드를 바꾸어 적용하기

### 실험 1. 기본 동작과 전파를 분리해서 관찰하기

아래 파일을 브라우저에서 열고 select에서 방식을 바꾸어 링크를 누른다. 목적지는 같은 문서의 fragment여서 실제 외부 서버로 이동하지 않는다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>취소와 전파 실험</title></head>
<body>
  <label for="mode">처리 방식</label>
  <select id="mode">
    <option value="none">제어하지 않음</option>
    <option value="default">기본 동작 취소</option>
    <option value="propagation">전파 중단</option>
    <option value="immediate">같은 대상의 이후 리스너도 중단</option>
  </select>
  <div id="parent"><a id="link" href="#destination">목적지</a></div>
  <p id="destination">도착 영역</p>
  <pre id="output"></pre>
  <script>
    const mode = document.querySelector('#mode');
    const parent = document.querySelector('#parent');
    const link = document.querySelector('#link');
    const output = document.querySelector('#output');
    const log = (value) => { output.textContent += `${value}\n`; };
    parent.addEventListener('click', () => {
      output.textContent = '';
      log('부모 capture');
    }, true);
    link.addEventListener('click', (event) => {
      log('링크 첫 리스너');
      if (mode.value === 'default') event.preventDefault();
      if (mode.value === 'propagation') event.stopPropagation();
      if (mode.value === 'immediate') event.stopImmediatePropagation();
      log(`취소 상태: ${event.defaultPrevented}`);
    });
    link.addEventListener('click', () => log('링크 둘째 리스너'));
    parent.addEventListener('click', () => log('부모 bubble'));
  </script>
</body>
</html>
```

| 선택 | 부모 capture | 링크 둘째 | 부모 bubble | fragment 이동 |
| --- | --- | --- | --- | --- |
| 제어 없음 | 실행 | 실행 | 실행 | 가능 |
| preventDefault | 실행 | 실행 | 실행 | 취소 |
| stopPropagation | 실행 | 실행 | 중단 | 가능 |
| stopImmediatePropagation | 실행 | 중단 | 중단 | 가능 |

부모 capture는 링크 핸들러보다 먼저 실행되어 어떤 선택에서도 이미 관찰했다. 링크의 첫 리스너 이후만 제어할 수 있다. 주소가 이미 같은 fragment라 새로 바뀌는 모습이 잘 안 보일 수 있으므로 실행 전 해시를 지우거나 다른 상태에서 비교한다. 핵심은 로그와 주소를 별도의 결과로 읽는 것이다.

### 같은 대상의 모든 작업을 지우는 것이 아니다

stopImmediatePropagation은 현재 리스너 뒤에 남은 같은 이벤트의 호출을 중단한다. 앞서 실행된 핸들러가 서버 요청을 시작하거나 state를 바꿨다면 그 효과를 되돌리지 않는다. 취소된 기본 작업과 이미 실행한 애플리케이션 코드는 서로 다른 복구 정책을 가진다.

---

## 13. 캡처 단계의 중단과 취소 플래그를 검증하기

### 실험 2. 캡처에서 중단하면 타깃까지 못 갈 수 있다

```html
<div id="capture-root"><button id="capture-button" type="button">실행</button></div>
<script>
  const root = document.querySelector('#capture-root');
  const button = document.querySelector('#capture-button');
  root.addEventListener('click', (event) => {
    console.log('root capture 1');
    event.stopPropagation();
  }, true);
  root.addEventListener('click', () => console.log('root capture 2'), true);
  button.addEventListener('click', () => console.log('button'));
  root.addEventListener('click', () => console.log('root bubble'));
</script>
```

두 root capture 로그가 나오고 button·root bubble은 나오지 않는다. 첫 호출에서 같은 대상의 나머지 capture 리스너를 중단한 것은 아니지만 이후 경로로 전달하지 않는다. stopPropagation은 “부모만 막는 메서드”가 아니라 어디에서 호출했는지에 따라 다음 경로를 막는 메서드다.

### 실험 3. cancelable false는 취소되지 않는다

```javascript
{
  const target = new EventTarget();
  target.addEventListener('before-action', (event) => event.preventDefault());
  const first = new Event('before-action');
  const second = new Event('before-action', { cancelable: true });
  console.log(target.dispatchEvent(first), first.defaultPrevented);
  console.log(target.dispatchEvent(second), second.defaultPrevented);
}
```

첫 줄은 true·false, 둘째는 false·true다. dispatchEvent의 반환은 이벤트가 취소되었는지를 반영한다. preventDefault 자체의 반환값을 조건으로 사용해 취소 성공을 판단하지 않는다. 해당 메서드는 취소 상태를 boolean으로 돌려주는 검사 함수가 아니다.

### 앱이 취소 프로토콜을 사용하는 경우

직접 만든 cancelable 사건을 알린 뒤 dispatchEvent가 true일 때만 앱의 후속 작업을 수행하도록 정할 수 있다. 이는 애플리케이션의 계약이다. 새 Event를 만들었다고 브라우저에 임의의 파일 저장·페이지 이동 같은 기본 동작을 추가한 것은 아니다. [dispatchEvent의 반환 계약](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)과 브라우저의 HTML 활성화 동작을 구별한다.

---

## 14. 컨트롤의 활성화 시점 때문에 생기는 반례

### 체크박스의 상태가 리스너 안에서 이미 바뀌어 보이는 이유

모든 기본 동작이 “모든 리스너 실행 후 처음 시작한다”는 단일 모델을 따르지는 않는다. HTML 체크박스에는 click 전달 전에 상태를 잠시 바꾸는 활성화 준비와, 취소된 경우 이전 상태로 되돌리는 규칙이 있다. 따라서 클릭 핸들러 안에서 본 checked와 호출이 끝난 뒤의 checked가 다를 수 있다.

### 실험 4. 취소하면 이전 선택 상태로 복구된다

```html
<label><input id="consent" type="checkbox">동의</label>
<script>
  const consent = document.querySelector('#consent');
  consent.addEventListener('click', (event) => {
    console.log('리스너 안:', consent.checked);
    event.preventDefault();
  });
  consent.click();
  console.log('호출 뒤:', consent.checked);
</script>
```

초기 선택이 false인 이 예제에서 리스너 안에서는 true, 호출 뒤에는 false를 관찰한다. 이는 비동기 state가 늦게 도착해서가 아니라 [HTML input의 활성화·취소 복구 규칙](https://html.spec.whatwg.org/multipage/input.html#the-input-element) 때문이다. 사용자 선택을 막으려는 목적이라면 컨트롤의 disabled·read-only 요구와 대체 설명이 맞는지 먼저 검토한다.

### label 클릭과 input 클릭을 한 번의 문자열 로그로 단정하지 않는다

레이블은 연결된 컨트롤의 활성화를 돕는다. 넓은 조상에 click 로그를 붙이고 레이블을 눌렀을 때 여러 관련 관찰이 보이면 사용자 클릭을 두 번 했다고 즉시 결론 내리지 않는다. target·currentTarget·이벤트 종류를 기록하고 레이블과 연결된 컨트롤의 기본 동작을 나누어 확인한다.

---

## 15. 폼 제출의 세 경로를 구별하기

### 검증·submit 이벤트·전송은 별개의 단계다

일반적인 사용자 제출은 적용되는 제약 검증을 거치고, 제출 가능한 경우 form의 submit 이벤트를 알린 뒤 취소 여부에 따라 전송으로 이어진다. invalid 이벤트는 기본적으로 버블링하지 않으므로 부모에서 검사 흐름을 관찰하려면 캡처 등을 구별한다. 버튼 click 하나의 로그만으로 폼의 모든 제출 경로를 설명하지 않는다.

### 실험 5. requestSubmit과 제약 검증

```html
<form id="apply-form">
  <label for="apply-email">이메일</label>
  <input id="apply-email" name="email" type="email" required>
  <button type="submit">신청</button>
</form>
<script>
  const form = document.querySelector('#apply-form');
  const input = document.querySelector('#apply-email');
  form.addEventListener('invalid', (event) => {
    console.log('검증 실패:', event.target.name);
  }, true);
  form.addEventListener('submit', (event) => {
    event.preventDefault();
    console.log('제출 이벤트');
  });
  form.requestSubmit();
  input.value = 'learner@example.com';
  form.requestSubmit();
</script>
```

첫 호출은 빈 required 입력의 검증 실패를 관찰하고 일반적인 submit 핸들러까지 가지 않는다. 둘째는 유효한 값으로 submit에 도달하지만 핸들러가 기본 제출을 취소하므로 이 예제는 서버로 전송하지 않는다. 브라우저의 검증 메시지 표시 방식은 환경에 따라 달라질 수 있다.

### submit 메서드와 이벤트 이름을 혼동하지 않는다

form.submit 메서드는 일반적인 submit 이벤트·제약 검증을 우회한다. 반면 requestSubmit은 제출 버튼을 통한 경로와 연결된다. new Event로 submit이라는 이름을 직접 dispatch하는 작업도 브라우저의 전체 폼 제출 절차를 자동 실행하는 일반 API가 아니다. 원하는 경로가 검증·리스너·실제 전송 중 무엇을 포함하는지 먼저 정한다.

---

## 16. passive와 비동기 처리의 계약

### 실험 6. passive 리스너는 취소하지 않는다

```javascript
{
  const target = new EventTarget();
  target.addEventListener('before-action', (event) => {
    event.preventDefault();
  }, { passive: true });
  const event = new Event('before-action', { cancelable: true });
  console.log(target.dispatchEvent(event), event.defaultPrevented);
}
```

브라우저에서 결과는 true·false이며 경고가 나올 수 있다. cancelable true만으로 현재 리스너의 취소가 보장되지 않는다. 실제 scroll·touch·wheel의 passive 기본 정책은 대상과 브라우저에 따라 확인해야 한다. 예제처럼 옵션을 명시하면 해당 실험의 조건은 분명해진다.

### Promise를 반환해도 전달 과정이 그 Promise를 기다리지는 않는다

비동기 핸들러의 나중 결과로 현재 기본 동작을 결정하려고 하면 호출 시점을 놓칠 수 있다. 현재 사건을 애플리케이션이 맡아 처리할 것이라면 기본 취소 여부를 동기적으로 정하고 필요한 form·값을 확보한 뒤 비동기 작업을 시작한다. 특정 빠른 Promise에서 우연히 취소된 관찰을 모든 입력 경로의 보장으로 사용하지 않는다.

### 비동기 승인 뒤 재제출에는 별도의 상태 설계가 필요하다

검사를 통과한 뒤 다시 requestSubmit을 호출하면 같은 핸들러에 재진입할 수 있다. 중복 요청·재진입·사용자의 값 변경·오래된 승인 결과를 구별해야 한다. 이 페이지에서는 취소 시점과 책임을 구별하고, 실제 네트워크 처리는 뒤의 fetch 학습에서 연결한다.

---

## 17. 접근성·성능 관점에서 제어를 최소화하기

### 기본 조작을 취소하면 대체 경로가 필요하다

Space·Enter·화살표 동작을 전부 취소하면 사용자가 네이티브 컨트롤을 조작하지 못할 수 있다. 단축키를 구현할 때도 텍스트 편집 중인지, 브라우저나 보조 기술의 조작을 침범하지 않는지 확인한다. 먼저 적절한 네이티브 요소를 고르고 필요한 범위의 동작만 추가한다.

### 전파 중단 대신 책임 범위를 검사할 수 있다

부모가 위임한 동작과 자식의 동작이 충돌하면 올바른 소속·data-action·상태 조건으로 부모가 처리하지 않게 할 수 있다. 무조건 전파를 없애면 상위의 키보드 처리나 상태 관찰이 뜻밖에 끊길 수 있다. 흐름 제어는 누가 그 사건을 알아야 하는지까지 포함해 판단한다.

### 측정과 오류 기록의 기준

느린 스크롤은 passive 정책뿐 아니라 리스너의 긴 계산과 레이아웃 비용도 조사한다. 디버깅 로그에는 type·target·currentTarget·phase·cancelable·defaultPrevented와 등록 옵션을 남긴다. 어떤 사건을 취소했는지 불분명한 “안 됨” 로그보다 조건을 재현할 수 있는 기록이 필요하다.

---

## 18. 실무 적용과 검증

중첩 UI에서 부모와 자식이 함께 실행되는 문제는 먼저 구조와 역할을 점검한다. 링크 안에 버튼을 중첩하는 식의 잘못된 HTML을 stopPropagation으로 덮어두지 않는다. 자식 전파를 무조건 막으면 상위 위임·분석·키보드 처리 등 다른 기능이 이벤트를 관찰하지 못할 수 있다.

기본 키보드 동작을 취소할 때는 사용자가 어떤 대체 조작을 얻는지도 확인한다. 키 입력이나 체크박스 기본 동작을 전부 막으면 컨트롤을 사용할 수 없게 만들 수 있다. 디버깅할 때 `cancelable`, `defaultPrevented`, 리스너 옵션, 캡처·버블링 위치를 같이 살핀다.

---

## 19. 핵심 정리

<callout icon="✅" color="green_bg">
preventDefault는 기본 동작 취소, stopPropagation은 전파 중단이다. 같은 대상의 이후 리스너까지 멈추려면 stopImmediatePropagation을 구별한다. 어떤 API든 먼저 제어할 흐름과 호출 시점을 결정해야 한다.
</callout>

---

## 출처

- [WHATWG — Event 인터페이스](https://dom.spec.whatwg.org/#interface-event)
- [MDN — preventDefault](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault)
- [MDN — stopPropagation](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation)
- [MDN — HTMLFormElement.requestSubmit](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/requestSubmit)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Event 인터페이스](https://dom.spec.whatwg.org/#interface-event): 기본 취소·전파 중단·즉시 중단의 플래그.
- [WHATWG — Event Dispatch](https://dom.spec.whatwg.org/#concept-event-dispatch): 호출 단계와 활성화·취소 복구의 연결.
- [WHATWG — input 활성화 규칙](https://html.spec.whatwg.org/multipage/input.html#the-input-element): 체크박스의 사전 상태 전환과 취소 후 복구.
- [MDN — dispatchEvent](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent): boolean 반환값과 동기 호출의 의미.
- [MDN — requestSubmit](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/requestSubmit): 검증과 submit 이벤트를 포함하는 제출 경로.
- [MDN — submit](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit): 직접 submit 호출이 우회하는 단계.
- [MDN — addEventListener 옵션](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener): passive 등 리스너 등록 계약.

---

## 20. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 링크 리스너에서 preventDefault를 호출했는데 부모 로그가 나온다. 왜 정상적인 결과인가?
2. 같은 버튼에 리스너 두 개가 등록되어 있다. stopPropagation과 stopImmediatePropagation의 결과를 비교하라.
3. 비동기 검사를 기다린 뒤 폼 제출을 취소하려고 한다. 호출 시점을 어떻게 설계해야 하는가?
4. passive 리스너의 preventDefault가 동작하지 않을 때 어떤 정보와 UX 요구를 확인하겠는가?
