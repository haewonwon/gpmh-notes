---
track: B
notion: https://app.notion.com/p/415178c4afac81cbbb1dd1204d83ec55
---

# Event Capturing / Target / Bubbling / Delegation


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · Event Capturing / Target / Bubbling / Delegation

이벤트 객체와 등록 옵션, 경로의 단계, 부모에서 처리하는 위임을 차례로 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- type·target·currentTarget·eventPhase·bubbles의 의미를 구별할 수 있다.
- 캡처·타깃·버블링에서 리스너의 실행 순서를 예측할 수 있다.
- 동적 자식·아이콘 클릭·중첩 위젯의 소속을 고려해 위임할 수 있다.
- 이벤트 종류·합성 이벤트·리스너 수명에 따른 예외를 설명할 수 있다.
- 키보드 조작과 로그·DevTools로 이벤트 경로를 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

버튼을 눌렀는데 부모의 핸들러도 실행되는 것은 보통 중복 등록 오류가 아니라 전파의 결과다. 이 전파를 이해하면 실행 순서를 디버깅할 수 있고, 목록의 모든 자식에 리스너를 붙이지 않는 이벤트 위임도 구현할 수 있다.

---

## 3. 관련 계층과 역할 구분

사용자 입력 장치와 브라우저는 이벤트 발생의 출발점이다. DOM 이벤트 모델은 대상과 경로를 결정하고 등록된 JavaScript 리스너를 호출한다. HTML 요소가 제공하는 기본 동작은 리스너 전파와 구별하며 다음 페이지에서 취소 여부를 배운다.

CSS는 hit testing과 보이는 위치에 영향을 줄 수 있지만 캡처·버블링을 화면 좌표의 이동으로 바꾸지는 않는다. React의 이벤트 추상화는 뒤의 React Basics에서 별도로 연결한다.

---

## 4. 이벤트와 리스너의 역할

이벤트는 클릭·키 입력 같은 발생 사실과 관련 정보를 전달하는 객체다. 리스너는 특정 대상으로 들어온 이벤트를 처리하는 함수다. `addEventListener(type, listener, options)`로 등록한다. `capture: true`는 캡처 단계의 처리를, 기본값 false는 타깃 및 버블링 쪽 처리를 지정한다.

`event.target`은 일반적인 DOM 예제에서 이벤트가 시작된 대상이고, `event.currentTarget`은 현재 실행 중인 리스너가 등록된 대상이다. 부모에서 자식 클릭을 처리할 때 두 값이 달라진다. [DOM 이벤트 모델](https://dom.spec.whatwg.org/#events)이 이벤트 경로와 각 단계를 정의한다.

---

## 5. 이벤트 객체와 등록 옵션을 먼저 읽는다

### 이벤트 종류와 대상은 별개의 정보다

이벤트의 type은 click·input·keydown 같은 종류이고 target은 출발 대상을 나타낸다. 같은 button에 여러 종류의 리스너를 붙일 수 있고, 같은 click 리스너를 부모에 붙여 여러 자식의 동작을 처리할 수도 있다. EventTarget은 DOM Element에만 있는 개념이 아니며 window·document 등도 이벤트를 받을 수 있다.

| 정보 | 뜻 | 확인 목적 |
| --- | --- | --- |
| `type` | 이벤트 종류 | 의도한 입력 경로인가 |
| `target` | 관찰자에게 보이는 출발 대상 | 어떤 자식에서 시작했는가 |
| `currentTarget` | 현재 리스너 등록 대상 | 어느 단계의 코드를 실행 중인가 |
| `eventPhase` | NONE·CAPTURING·AT_TARGET·BUBBLING | 경로의 어느 지점인가 |
| `bubbles` | 버블링 여부 | 조상의 비캡처 위임이 가능한가 |
| `cancelable` | 기본 동작 취소 가능 여부 | 다음 페이지의 취소 제어와 연결 |

### 리스너 옵션은 처리 요구를 표현한다

capture는 캡처 단계 사용, once는 첫 실행 뒤 자동 제거, passive는 기본 동작을 취소하지 않겠다는 약속이다. signal을 지정하면 관련 AbortSignal로 등록 수명을 묶을 수 있다. 각 옵션은 서로 다른 목적이므로 “빠른 이벤트 옵션” 하나로 뭉뚱그리지 않는다.

```javascript
const button = document.querySelector('#start');
function handleFirstClick() {
  console.log('첫 실행');
}
if (button) {
  button.addEventListener('click', handleFirstClick, { once: true });
}
```

동일한 함수 내용이 아니라 동일한 함수 객체 참조가 등록·제거에서 중요하다. 아래처럼 익명 함수를 새로 작성하면 원래 리스너를 찾지 못한다.

```javascript
const button = document.querySelector('#start');
if (button) {
  button.addEventListener('click', () => console.log('실행'));
  button.removeEventListener('click', () => console.log('실행')); // 다른 함수
}
```

---

## 6. 캡처 → 타깃 → 버블링

단순한 문서 안에서 클릭은 조상 경로의 캡처 리스너를 거쳐 실제 타깃에 도달하고, 버블링하는 이벤트라면 조상 쪽의 리스너로 이어진다. 같은 타깃에서는 캡처 리스너가 먼저, 비캡처 리스너가 뒤에 실행된다. 타깃에서의 `eventPhase`는 두 경우 모두 `AT_TARGET`이다.

모든 이벤트가 버블링하지는 않는다. `focus`와 `blur`는 버블링하지 않으므로 위임에 캡처를 쓰거나, 목적에 맞으면 버블링하는 `focusin`·`focusout`을 검토한다. 합성 이벤트는 생성 옵션에 따라 `bubbles`가 다르다.

---

## 7. 전파 경로를 단계별로 추적하기

### 캡처 리스너와 비캡처 리스너

단순한 `div > button > span`에서 span을 누르면 div와 button의 캡처 리스너가 타깃을 향해 실행되고, span의 타깃 리스너를 거친 뒤 button과 div의 버블링 리스너로 돌아온다. 이벤트 경로는 화면 좌표의 상자 크기보다 DOM 조상 관계를 기준으로 이해한다.

타깃에도 capture와 비capture 리스너가 있을 수 있다. 타깃에서 capture 리스너가 먼저 실행되고 뒤에 비capture 리스너가 실행되며, 둘의 eventPhase는 모두 AT_TARGET이다. “capture 옵션이면 무조건 phase가 CAPTURING”이라고 읽으면 틀린다.

### 모든 관찰자가 같은 출발 노드를 보는 것은 아니다

일반 DOM 예제에서는 target이 그대로 유지되지만 Shadow DOM 경계에서는 관찰자 위치에 따라 target이 재지정될 수 있다. 이 페이지의 실험은 경계가 없는 문서로 한정하고, 경계를 가진 웹 컴포넌트는 이후 해당 기능을 배울 때 composed·composedPath와 연결한다.

### currentTarget은 나중에도 그대로 남는 전역 정보가 아니다

currentTarget은 현재 리스너 실행과 연결된 정보다. 나중에 실행되는 비동기 코드에서 그 값을 다시 읽을 때 같은 등록 대상을 기대하지 말고, 필요한 요소 참조를 리스너 안에서 지역 변수로 확보한다. 이벤트 객체 전체를 로그로 펼칠 때도 출력 시점과 펼쳐 보는 시점의 객체 상태를 구별한다.

---

## 8. 예제: 실행 순서를 보고 위임으로 연결하기

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>이벤트 경로</title></head>
<body>
  <h1>이벤트 경로</h1>
  <div id="panel">
    <ul id="notes">
      <li><button type="button" data-action="open"><span>HTML</span></button></li>
    </ul>
  </div>
  <script>
    const panel = document.querySelector('#panel');
    const list = document.querySelector('#notes');
    const button = list.querySelector('button');
    const log = (name) => (event) => {
      console.log(name, event.eventPhase, event.target.tagName,
                  event.currentTarget.tagName);
    };

    panel.addEventListener('click', log('panel capture'), { capture: true });
    list.addEventListener('click', log('list capture'), { capture: true });
    button.addEventListener('click', log('button'));
    list.addEventListener('click', log('list bubble'));
    panel.addEventListener('click', log('panel bubble'));

    list.addEventListener('click', (event) => {
      if (!(event.target instanceof Element)) return;
      const control = event.target.closest('button[data-action="open"]');
      if (!control || !list.contains(control)) return;
      console.log('선택:', control.textContent.trim());
    });
  </script>
</body>
</html>
```

버튼 안의 `span`을 클릭하면 로그는 panel capture → list capture → button → list bubble → 위임 처리 → panel bubble 순서로 나온다. 예제에 등록하지 않은 조상의 이벤트 경로도 존재하지만 로그를 출력하지 않을 뿐이다. 실제 target은 `span`이므로 button의 리스너도 타깃 단계가 아니라 버블링 단계에서 실행된다. 버튼의 빈 부분을 눌러 버튼 자체가 타깃이면 button 리스너의 단계는 `AT_TARGET`이 된다.

위임 리스너는 target이 버튼 안의 요소일 가능성을 고려하여 `closest`로 적절한 조상을 찾는다. currentTarget인 `list`만 읽어서는 어떤 버튼이 눌렸는지 구별할 수 없다. `contains` 검사는 선택한 컨트롤이 위임 범위에 있는지 확인한다. 중첩된 독립 위젯이 있다면 추가로 소속 경계를 검사해야 한다.

동적으로 같은 형태의 버튼을 목록 안에 추가해도 부모의 위임 리스너가 처리할 수 있다. 자식을 새로 만들 때마다 리스너를 다시 붙일 필요가 없다는 것이 위임의 이점이다.

---

## 9. 위임을 안전하게 적용하는 조건

### target의 태그를 바로 검사하지 않는다

버튼 안에 span·아이콘·강조 텍스트가 있으면 실제 출발 대상이 버튼이 아닐 수 있다. 가장 가까운 의도한 컨트롤을 closest로 찾고, 위임 범위 안에 있는지 확인한다. 데이터 속성으로 동작을 구별할 때는 허용한 값만 처리하여 임의 문자열이 코드 실행으로 이어지지 않게 한다.

### 중첩 위젯의 소속 경계

바깥 목록 안에 독립된 안쪽 목록이 있으면 바깥 위임 코드가 안쪽 버튼까지 처리할 수 있다. DOM의 contains는 바깥 아래에 있다는 사실만 알려주므로 독립 영역 구분이 필요하면 closest로 소속 목록까지 확인하는 등의 규칙을 추가한다. 모든 자손을 같은 기능으로 처리할지부터 결정해야 한다.

### 위임하기 어려운 이벤트도 있다

focus·blur처럼 버블링하지 않는 이벤트는 기본 버블링 위임으로 잡히지 않는다. 목적에 맞으면 focusin·focusout 또는 캡처를 선택한다. mouseenter와 mouseleave도 일반적인 mouseover·mouseout과 전파·발생 조건이 다르므로 이름만 바꾸어 같은 동작을 기대하지 않는다.

### 직접 등록이 더 분명한 경우

대상 수가 작고 각 컨트롤의 수명이 명확하면 직접 등록이 간단하다. 동적으로 추가되는 많은 반복 항목을 같은 방식으로 처리하면 위임이 편리할 수 있다. 성능은 이벤트 빈도·대상 수·위임 처리 비용을 측정하며, document에 모든 기능을 몰아넣으면 범위와 의존성 관리 비용이 커질 수 있다.

---

## 10. 사용자 입력과 프로그램이 발생시킨 이벤트

```javascript
const target = new EventTarget();
target.addEventListener('study', (event) => {
  console.log(event.type, event.bubbles, event.isTrusted);
});
target.dispatchEvent(new Event('study', { bubbles: true }));
```

이 예제는 DOM 요소 없이도 이벤트 등록과 전달을 실험한다. 직접 dispatch한 이벤트는 브라우저가 신뢰된 사용자 입력으로 만든 이벤트와 구별된다. dispatchEvent는 이 호출의 리스너를 동기적으로 실행하지만, 모든 실제 사용자 입력이 같은 호출 경로로 발생한다고 일반화하지 않는다.

합성 이벤트를 발생시켰다고 실제 포인터·키보드 조작의 모든 기본 동작이나 사용자 활성화 조건이 재현되는 것은 아니다. 기능을 검증할 때 이벤트 함수 호출 시험과 실제 사용자 조작 시험을 구별한다.

---

## 11. 위임이 해결하는 것과 남는 조건

위임은 이벤트 전파를 이용하는 처리 방식이지 별도의 이벤트 단계가 아니다. 이벤트 종류가 버블링하는지, 하위 코드가 전파를 중단하는지, 위임 범위가 지나치게 넓지 않은지 확인해야 한다. 버튼마다 전용 상태와 정리 작업이 필요한 경우에는 직접 등록하는 편이 더 명확할 수도 있다.

Shadow DOM처럼 경계를 가진 구조에서는 target이 바깥 관찰자에게 재지정될 수 있으며 모든 내부 노드가 그대로 보이는 것은 아니다. 이 페이지의 순서 예측은 일반 DOM 트리 예제를 기준으로 한다. 경계와 `composedPath()`의 상세한 동작은 해당 구조를 배울 때 확장한다.

---

## 12. 이벤트 전파를 경로와 리스너 목록으로 읽기

### 먼저 경로를 만들고 그 경로에서 리스너를 호출한다

단순 DOM에서 출발 요소를 정하면 브라우저는 관련 조상을 포함한 이벤트 경로를 구성한다. 이후 캡처 방향과 타깃·버블링 방향의 호출을 진행한다. 이 때문에 리스너 안에서 DOM을 바꿨다고 이미 진행 중인 이벤트가 항상 새 부모 관계로 처음부터 다시 계산되는 것은 아니다. [DOM 이벤트 전달 알고리즘](https://dom.spec.whatwg.org/#concept-event-dispatch)을 경로 구성과 호출 단계로 나누어 읽는다.

이 설명은 Shadow DOM 경계가 없는 문서를 기준으로 한다. 경계가 있는 문서의 target 재지정·composed 여부는 추가 규칙이므로 여기의 간단한 가족 관계만으로 모든 경계를 설명하지 않는다.

### 하나의 이벤트 객체를 여러 위치에서 관찰한다

일반적인 경로에서는 여러 리스너가 같은 발생 사실을 서로 다른 위치에서 관찰한다. target은 출발 대상, currentTarget은 현재 리스너가 붙은 대상이다. phase는 이 관찰이 어느 전달 단계인지 알려준다. 같은 버튼에서 생긴 이벤트가 부모에서 보인다고 새로운 클릭을 부모가 다시 만들었다는 뜻은 아니다.

### 실행 순서를 예측할 세 가지 질문

이 이벤트는 어떤 경로를 가지는가, 각 리스너는 capture인지 비capture인지, 이벤트가 bubbles를 허용하는가를 먼저 확인한다. 그 뒤 같은 대상·같은 처리 단계의 등록 순서와 중단 여부를 본다. 화면의 사각형이 겹친 정도나 함수 파일의 위치로 전파 순서를 예측하지 않는다.

---

## 13. 캡처·타깃·버블링을 재현하는 실험실

### 공통 HTML과 출력 도우미

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>이벤트 경로 실험</title></head>
<body>
  <h1>이벤트 경로 실험</h1>
  <div id="lab"></div>
  <pre id="output"></pre>
  <script>
    const lab = document.querySelector('#lab');
    const output = document.querySelector('#output');
    function fixture() {
      lab.innerHTML = '<div class="outer"><button type="button"><span>실행</span></button></div>';
      output.textContent = '';
      return {
        outer: lab.querySelector('.outer'),
        button: lab.querySelector('button'),
        span: lab.querySelector('span'),
      };
    }
    function log(value) { output.textContent += `${value}\n`; }
  </script>
</body>
</html>
```

파일을 연 뒤 다음 실험을 콘솔에서 각각 실행한다. fixture는 해당 실험 영역만 교체하므로 이전 실험에서 분리한 요소의 리스너가 새 트리에 자동으로 이어지지 않는다. 프로그램이 만든 이벤트로 순서를 확인한 뒤 실제 버튼 클릭·키보드 조작도 비교한다.

### 실험 1. 타깃의 두 리스너는 모두 타깃 단계다

```javascript
{
  const { outer, button, span } = fixture();
  for (const [name, element] of Object.entries({ outer, button, span })) {
    element.addEventListener('click', (event) => {
      log(`${name} capture: ${event.eventPhase}`);
    }, true);
    element.addEventListener('click', (event) => {
      log(`${name} noncapture: ${event.eventPhase}`);
    });
  }
  span.dispatchEvent(new MouseEvent('click', { bubbles: true }));
}
```

```text
outer capture: 1
button capture: 1
span capture: 2
span noncapture: 2
button noncapture: 3
outer noncapture: 3
```

숫자 1은 캡처, 2는 타깃, 3은 버블링 단계다. span에는 capture로 등록한 리스너와 비capture 리스너가 있지만 둘 다 타깃에서 호출된다. 다음에는 span 대신 button에 이벤트를 전달하여 버튼의 단계가 어떻게 달라지는지 확인한다. 같은 옵션의 리스너라도 대상의 경로 위치에 따라 phase가 달라질 수 있다.

### 실험 2. bubbles false가 캡처까지 없애지는 않는다

위 실험에서 새 fixture로 같은 리스너를 다시 등록한 뒤 bubbles를 false로 바꾼다. 조상 캡처와 타깃의 두 호출은 볼 수 있지만 조상 비capture 버블링 호출은 생략된다. “버블링하지 않는 이벤트는 부모가 어떤 방식으로도 관찰할 수 없다”는 결론은 틀리다.

focus·blur에 부모의 기본 비capture 리스너를 붙여 기대한 결과가 없으면 캡처 또는 목적에 맞는 focusin·focusout을 검토한다. 서로 다른 이벤트는 발생 조건도 다를 수 있어 단순한 이름 치환으로 취급하지 않는다.

### 실험 3. currentTarget은 전달 뒤 정리된다

```javascript
{
  const { outer, span } = fixture();
  let capturedTarget;
  let capturedCurrent;
  const event = new Event('study', { bubbles: true });
  outer.addEventListener('study', (received) => {
    capturedTarget = received.target;
    capturedCurrent = received.currentTarget;
    log(`전달 중: ${received.eventPhase}`);
  });
  span.dispatchEvent(event);
  log(capturedTarget === span);
  log(capturedCurrent === outer);
  log(event.currentTarget === null);
  log(event.eventPhase);
}
```

마지막 네 결과는 true, true, true, 0이다. 리스너 안에서 필요한 요소를 저장하면 나중에도 그 참조를 사용할 수 있다. 그러나 이벤트 객체의 currentTarget을 나중에 다시 읽어 같은 값이 남아 있다고 기대하지 않는다. 전달 알고리즘은 종료 시 currentTarget과 단계 등의 상태를 정리한다.

---

## 14. 리스너의 중복·제거·수명을 정확히 관리하기

### 실험 4. 같은 함수의 같은 등록은 단순 누적되지 않는다

```javascript
{
  const target = new EventTarget();
  let calls = 0;
  function listener() { calls += 1; }
  target.addEventListener('study', listener);
  target.addEventListener('study', listener, { once: true });
  target.dispatchEvent(new Event('study'));
  target.dispatchEvent(new Event('study'));
  log(calls);
}
```

calls는 2다. 같은 type·callback·capture인 두 번째 등록이 기존 리스너를 once로 갱신한 것이 아니다. 중복 등록 방지의 기준과 옵션을 바꾸는 작업은 다르다. 옵션을 바꿔야 한다면 기존 등록을 적절하게 제거한 뒤 다시 등록하는 방식으로 의도를 분명히 한다. [DOM의 리스너 등록 규칙](https://dom.spec.whatwg.org/#concept-event-listener)과 [MDN addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)를 근거로 한다.

### 실험 5. 제거할 때 capture가 다르면 남는다

```javascript
{
  const target = new EventTarget();
  let calls = 0;
  function listener() { calls += 1; }
  target.addEventListener('study', listener, true);
  target.removeEventListener('study', listener, false);
  target.dispatchEvent(new Event('study'));
  target.removeEventListener('study', listener, true);
  target.dispatchEvent(new Event('study'));
  log(calls);
}
```

결과는 1이다. 첫 제거는 capture 설정이 달라 해당 등록을 제거하지 못한다. 두 번째 제거 뒤에는 더 호출되지 않는다. 함수 내용이 같은 새 화살표 함수를 만들어 넘기는 것도 같은 callback 참조가 아니므로 제거를 보장하지 않는다.

### once와 AbortSignal로 수명을 표현한다

한 번만 의미 있는 동작에는 once를 사용할 수 있다. 화면 단위의 여러 등록을 함께 정리해야 한다면 하나의 AbortController의 signal로 묶는 방법을 검토한다. 정리 기능은 중복 초기화를 무조건 허용하는 해법이 아니라 소유한 리스너가 언제 끝나야 하는지 표현하는 도구다.

```javascript
{
  const { outer, button } = fixture();
  const controller = new AbortController();
  outer.addEventListener('click', () => log('outer'), { signal: controller.signal });
  button.addEventListener('click', () => log('button'), { signal: controller.signal });
  controller.abort();
  button.click();
  log('두 등록을 정리했습니다.');
}
```

abort 뒤 button·outer 로그는 나오지 않는다. 다른 코드가 별도로 등록한 리스너까지 모두 삭제한 것은 아니며, 해당 signal과 연결된 등록만 정리한다.

---

## 15. 이벤트 위임에서 대상·동작·소속을 나누기

### 가장 가까운 버튼과 올바른 소유자는 다를 수 있다

버튼 안의 아이콘을 클릭하면 closest로 버튼을 찾을 수 있지만, 그 버튼이 중첩된 독립 목록의 버튼일 수도 있다. contains는 바깥 아래에 있다는 사실만 확인하므로 위젯의 소유 경계를 대체하지 못한다. 위임은 “가까운 컨트롤을 찾기”와 “이 컨트롤을 내가 처리할 책임이 있는가”의 두 검사를 가진다.

### 실험 6. 안쪽 목록의 클릭을 바깥에서 처리하지 않기

```html
<ul id="outer-list">
  <li><button type="button" data-action="open"><span>바깥 항목</span></button></li>
  <li>
    <ul>
      <li><button type="button" data-action="open">안쪽 항목</button></li>
    </ul>
  </li>
</ul>
<script>
  const outer = document.querySelector('#outer-list');
  outer.addEventListener('click', (event) => {
    if (!(event.target instanceof Element)) return;
    const button = event.target.closest('button[data-action]');
    if (!button || button.closest('ul') !== outer) return;
    if (button.dataset.action !== 'open') return;
    console.log('바깥 목록의 책임:', button.textContent.trim());
  });
</script>
```

바깥 버튼의 span을 눌러도 버튼을 찾아 처리한다. 안쪽 버튼은 가장 가까운 ul이 바깥 목록과 다르므로 처리하지 않는다. 이벤트 자체를 무조건 중단한 것이 아니라 이 리스너의 업무 처리에서 제외한 것이다. 상위의 다른 관찰자가 계속 이벤트를 볼 필요가 있다면 이런 범위 검사가 도움이 된다.

### 동적으로 추가한 항목의 계약

같은 책임 범위에 같은 data-action 구조로 새 버튼을 추가하면 부모 리스너가 처리할 수 있다. 그러나 data 속성이 사용자 입력으로 바뀔 수 있다면 허용한 동작만 선택하고 문자열을 코드처럼 실행하지 않는다. 위임의 유연함이 실행 권한을 임의의 마크업에 넘긴다는 뜻은 아니다.

---

## 16. 프로그램 이벤트와 실제 조작의 검증 범위

### dispatchEvent의 동기 호출

직접 dispatchEvent를 호출하면 관련 리스너는 해당 호출 안에서 동기적으로 실행된다. 이것이 브라우저가 실제 사용자 입력을 만드는 전체 과정과 같다는 뜻은 아니다. 프로그램 이벤트의 bubbles·cancelable 기본값도 생성 시 확인해야 한다. [MDN dispatchEvent](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)는 이 호출·반환 계약을 설명한다.

### isTrusted를 접근 권한으로 사용하지 않는다

이 값은 이벤트의 생성 경로와 관련된 정보이지 사용자의 계정 권한이나 요청의 정당성을 증명하는 인증 수단이 아니다. 브라우저가 제공하는 이벤트와 직접 발생시킨 이벤트를 구별하는 단서로 읽되 보안 정책을 이 값 하나에 의존하지 않는다. 클라이언트 검증만으로 서버의 인가를 대체하지 못하는 것과 같은 책임 구분이 필요하다.

### 실제 키보드 경로를 따로 확인한다

스크립트로 click 리스너를 실행시켰다고 Tab 이동·Enter·Space의 실제 조작과 포커스 표시를 검증한 것은 아니다. 네이티브 button과 링크를 사용하고 마우스·키보드·보조 기술에서 같은 목적을 달성하는지 확인한다. 캡처·버블링 이해는 기능의 전달 구조를 설명하고, 접근성 검증은 사용자가 그 경로에 진입할 수 있는지 확인한다.

---

## 17. 흔한 오해와 실무 검증

캡처는 화면의 바깥 좌표에서 안쪽으로 움직인다는 뜻이 아니라 DOM 경로의 단계다. 캡처 옵션이 기본 동작을 취소하지도 않는다. `eventPhase`, target, currentTarget을 함께 출력하면 잘못된 가정을 쉽게 찾을 수 있다.

리스너를 제거할 때는 등록할 때 사용한 함수 참조와 capture 설정을 맞춘다. 같은 내용의 새 함수를 전달하면 같은 리스너가 아니다. 화면이나 위젯이 제거될 때 필요한 리스너 정리를 수행한다.

키보드로도 실행 가능한 `button`과 `a`를 사용하고 `click` 처리로 연결하면 기본 조작을 활용할 수 있다. 마우스 전용 이벤트에만 기능을 묶어 접근성을 잃지 않는다. 위임의 성능 이점은 리스너 수와 이벤트 빈도에 따라 달라지므로 모든 이벤트를 document 하나에 모으는 방식으로 일반화하지 않는다.

\[미리보기: preventDefault / stopPropagation\]

전파 중단과 브라우저 기본 동작 취소는 서로 다른 제어다. 바로 다음 페이지에서 같은 클릭에 대해 각각 무엇이 달라지는지 비교한다.

---

## 18. 핵심 정리

<callout icon="✅" color="green_bg">
전파는 이벤트가 DOM 경로를 따라 전달되는 과정이고, 위임은 그 과정을 이용하여 부모에서 자식의 동작을 처리하는 방식이다. target으로 출발점을 확인하고 currentTarget으로 현재 리스너의 위치를 확인한다.
</callout>

---

## 출처

- [WHATWG — DOM Events](https://dom.spec.whatwg.org/#events)
- [MDN — Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)
- [MDN — Event.currentTarget](https://developer.mozilla.org/en-US/docs/Web/API/Event/currentTarget)
- [MDN — addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Dispatching Events](https://dom.spec.whatwg.org/#concept-event-dispatch): 경로 구성·단계별 전달·종료 시 상태 정리.
- [WHATWG — Event Listener](https://dom.spec.whatwg.org/#concept-event-listener): 중복 등록과 capture·once·signal의 계약.
- [MDN — addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener): 등록 옵션과 함수 참조의 사용.
- [MDN — removeEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener): 제거할 때 일치해야 하는 조건.
- [MDN — dispatchEvent](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent): 동기 호출·취소 반환값의 범위.
- [MDN — closest](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest): 자신부터 조상을 찾는 위임 대상 탐색.

---

## 19. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 버튼 자체를 눌렀을 때와 버튼 안의 span을 눌렀을 때 button 리스너의 eventPhase와 target은 어떻게 다른가?
2. 위임 코드가 `event.target.matches('button')`만 사용하면 어떤 클릭을 놓칠 수 있는가?
3. focus를 부모의 기본 리스너로 처리했지만 실행되지 않는다. 이벤트 종류와 단계 관점에서 두 가지 선택지를 제시하라.
4. 새 목록 항목에 직접 리스너가 없는데 부모 코드가 실행된다. 어떤 경로로 동작하는지 설명하라.
