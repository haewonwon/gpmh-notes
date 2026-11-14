---
track: B
notion: https://app.notion.com/p/415178c4afac8129889bec16ee7c5f4f
---

# Event Handling / Conditional Rendering


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · React Basics 복습 · Event Handling / Conditional Rendering

사용자 사건을 상태 갱신으로, 현재 상태를 조건에 맞는 UI로 연결하고 기본 조작·포커스까지 검증한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 핸들러 전달·호출·인수 전달과 커스텀 콜백 prop을 구별할 수 있다.
- React의 이벤트 대상·전파·기본 동작 제어와 예외를 설명할 수 있다.
- if·삼항·논리 AND의 반환값과 흔한 숫자 0 오류를 분석할 수 있다.
- 조건부 제거·숨김·비활성화가 상태와 탐색에 주는 차이를 판단할 수 있다.
- 사건·state·JSX·DOM·접근성 순서로 상호작용을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

React에서 “버튼을 누르면 문장을 보이게 한다”는 요구는 DOM을 직접 찾아 숨김을 바꾸는 코드보다, 클릭 → 상태 갱신 → 해당 상태의 UI 계산으로 표현할 수 있다. 이벤트 처리와 조건부 렌더링을 하나의 흐름으로 연결하면 화면과 데이터가 따로 움직이는 일을 줄일 수 있다.

---

## 3. 관련 계층과 역할 구분

브라우저와 HTML 컨트롤은 사용자 입력과 기본 동작을 제공한다. React DOM의 이벤트 처리 체계는 등록한 React 핸들러를 호출하고, 핸들러는 상태 소유자에게 갱신을 요청한다. 렌더링 코드는 그 상태의 JSX를 계산한다.

React 이벤트의 이름과 예외는 공식 문서의 계약을 따른다. CSS는 표시 방법을 바꿀 수 있지만 조건부 제거와 같은 수명·포커스 결과를 자동으로 재현하는 것은 아니다.

---

## 4. 핸들러는 호출 결과가 아니라 함수로 전달한다

`onClick={handleClick}`은 나중에 호출할 함수를 전달한다. `onClick={handleClick()}`은 렌더링 중 함수를 호출하므로 전혀 다르다. 인수가 필요하면 `onClick={() => handleClick(id)}`처럼 함수를 전달한다. React의 이벤트 prop 이름은 onClick, onSubmit처럼 대소문자를 구별한다.

React 이벤트 객체는 target·currentTarget·preventDefault·stopPropagation 같은 인터페이스를 제공한다. 웹의 React 이벤트는 React DOM의 이벤트 처리 체계를 통하므로 모든 이벤트가 네이티브 등록 방식과 완전히 같다고 가정하지 않는다. 예를 들어 React의 onScroll은 전파되지 않는다. 기본 동작 취소와 전파 중단의 개념적 구분은 그대로 필요하다. [React 이벤트 안내](https://react.dev/learn/responding-to-events)를 기준으로 예외를 확인한다.

---

## 5. 이벤트를 props와 상태의 흐름으로 연결하기

### 인수가 있을 때도 함수로 전달한다

```jsx
<button type="button" onClick={() => selectTopic('html')}>HTML 선택</button>
```

위 조각은 selectTopic을 호출 가능한 범위에 정의한 JSX다. 렌더링 중 선택을 실행하는 대신 나중에 클릭할 함수를 만든다. 핸들러 안에서 호출할 로직과 JSX가 지금 계산할 표현을 구별한다.

### 커스텀 이벤트 prop은 업무 동작을 표현할 수 있다

```jsx
function StartButton({ topicId, onStart }) {
  return (
    <button type="button" onClick={() => onStart(topicId)}>
      학습 시작
    </button>
  );
}
```

onStart는 개발자가 정한 컴포넌트 API 이름이며 브라우저가 그 이름의 DOM 이벤트를 자동으로 발생시키는 것이 아니다. StartButton은 실제 button의 onClick에서 전달받은 콜백을 호출한다. 부모는 상태를 소유하고 자식은 사용자의 의도를 전달할 수 있다.

### target과 currentTarget을 React에서도 구별한다

버튼 안 아이콘을 누르면 target은 내부 요소일 수 있고 currentTarget은 리스너가 붙은 버튼이다. 버튼 자체의 데이터 속성이나 폼을 읽으려면 필요한 등록 대상을 명시적으로 선택한다. 비동기 처리에 필요한 값은 핸들러 실행 중 확보한다.

---

## 6. 전파·기본 동작과 컴포넌트 계층

컴포넌트의 소스 중첩과 최종 DOM의 관계를 확인한다. 일반적인 중첩 DOM의 클릭은 부모 React 핸들러에도 전달될 수 있지만 모든 이벤트가 같은 전파 규칙을 가진 것은 아니다. 예를 들어 React의 onFocus·onBlur와 네이티브 focus·blur의 버블링 차이, onScroll의 예외를 문서에서 확인한다.

폼의 onSubmit에서 preventDefault를 호출해 기본 제출을 취소해도 다른 핸들러의 관찰을 모두 없애는 것은 아니다. 자식 stopPropagation으로 모든 충돌을 덮기보다 잘못된 인터랙티브 중첩과 상태 책임부터 점검한다.

---

## 7. 예제: 상세 정보의 표시 상태를 UI로 표현

기존 React 프로젝트의 `App.jsx`에서 실행한다.

```jsx
import { useState } from 'react';

function Details({ title, children }) {
  const [open, setOpen] = useState(false);

  function toggle() {
    setOpen((previous) => !previous);
  }

  return (
    <section>
      <h2>{title}</h2>
      <button
        type="button"
        aria-expanded={open}
        aria-controls="study-details"
        onClick={toggle}
      >
        {open ? '상세 접기' : '상세 보기'}
      </button>
      <div id="study-details" hidden={!open}>
        {children}
      </div>
      {open ? <p>상세 정보를 펼쳤습니다.</p> : <p>요약을 보고 있습니다.</p>}
    </section>
  );
}

export default function App() {
  return (
    <main>
      <h1>React 기초</h1>
      <Details title="이벤트와 화면">
        <p>상태를 바꾸면 해당 상태의 UI를 다시 계산합니다.</p>
        <a href="https://react.dev/learn/responding-to-events">공식 문서</a>
      </Details>
    </main>
  );
}
```

children은 부모가 태그 사이에 전달한 UI를 받는 prop이다. 클릭하면 핸들러가 다음 open을 요청하고 React가 버튼 문구, aria-expanded, hidden, 조건 문단을 새 상태에 맞게 반영한다. 버튼은 키보드로 실행할 수 있고 펼침 상태도 노출한다. 이 예제는 Details 한 개를 전제로 한다. 여러 개를 렌더링하려면 id를 고유하게 전달·생성하여 aria-controls 관계가 충돌하지 않게 해야 한다.

`hidden` 영역은 DOM에 남지만 기본적으로 화면과 접근성 트리에서 숨겨진다. 반면 `{open && <div>...</div>}`처럼 조건으로 제외하면 해당 부분을 렌더 트리에서 제거한다. 둘은 포커스·내부 상태·수명에 대한 결과가 다를 수 있다. 단지 모양이 같은가만으로 선택하지 않는다.

---

## 8. 조건을 표현하는 방법

컴포넌트 앞에서 `if`로 다른 UI를 반환하거나, 일부 위치에서 삼항 연산자·논리 AND를 사용할 수 있다. 렌더링할 것이 없으면 null을 반환할 수 있다. Hook 호출 순서는 조건 분기와 관계없이 지켜야 하므로 useState 같은 Hook을 조건문 안에 넣지 않는다.

```jsx
function ResultSummary({ count }) {
  return (
    <section>
      <h2>검색 결과</h2>
      {count > 0 ? <p>{count}개를 찾았습니다.</p> : <p>결과가 없습니다.</p>}
    </section>
  );
}
```

`count && <p>...</p>`는 count가 0일 때 숫자 0을 결과로 반환하여 화면에 표시할 수 있다. 조건을 boolean으로 만들거나 위 예제처럼 분기를 명시한다. false·null·undefined는 보통 화면의 노드를 만들지 않지만 숫자 0도 같은 방식으로 사라진다고 생각하면 안 된다. [React 조건부 렌더링 안내](https://react.dev/learn/conditional-rendering)가 이 차이를 설명한다.

---

## 9. 조건부 렌더링의 선택과 함정

### if·삼항·논리 AND의 반환값

조건이 복잡하면 컴포넌트 앞에서 if로 UI 분기를 정하면 읽기 쉬울 수 있다. 짧은 두 갈래는 삼항, 참일 때만 추가할 명확한 boolean 조건은 논리 AND를 사용할 수 있다. 스타일 취향보다 실제 반환되는 값과 읽기 난도를 기준으로 고른다.

```jsx
function Message({ status }) {
  if (status === 'loading') return <p role="status">불러오는 중</p>;
  if (status === 'error') return <p role="alert">결과를 불러오지 못했습니다.</p>;
  return <p>결과가 준비되었습니다.</p>;
}
```

이 컴포넌트는 부모가 정한 status를 입력받는다. 실제 비동기 요청은 여기서 시작하지 않는다. 로딩·오류·성공 상태가 동시에 참인 여러 불리언보다 하나의 상태값이 더 명확한 경우도 있다. 상태를 바꾸는 사건과 각 상태의 UI를 함께 설계한다.

### null은 UI를 생략하지만 부수 효과의 정리 명령은 아니다

컴포넌트가 null을 반환하면 그 반환 UI가 비어 있게 된다. 부모가 해당 컴포넌트를 렌더 트리에서 제거하는 것과 모든 수명 결과가 동일하다고 생각하지 않는다. 이 페이지에서는 화면의 포함·제외를 구별하고 외부 작업 정리는 후속 수명 학습으로 연결한다.

### 조건문과 Hook 호출 순서

useState 같은 Hook은 조건에 따라 건너뛰는 위치에 두지 않는다. 필요한 Hook을 최상위에서 호출한 다음 반환 UI를 분기하거나, 상태를 가진 하위 컴포넌트를 명확히 분리한다. 이 규칙의 내부 원리는 후속 React Mental Model에서 다룬다.

---

## 10. 숨김·제거·비활성화의 차이

| 방법 | UI·DOM 의미 | 사용자 흐름의 질문 |
| --- | --- | --- |
| 조건에서 제외 | 해당 하위 UI를 트리에서 제거 | 지역 상태·포커스가 사라져도 되는가? |
| hidden 등 숨김 | DOM을 유지하되 일반적으로 표시·탐색에서 숨김 | 다시 열 때 어떤 상태를 유지하는가? |
| disabled | 컨트롤을 남기고 조작을 제한 | 왜 사용할 수 없는지 알 수 있는가? |
| CSS만으로 투명하게 표시 | 구현에 따라 여전히 포커스·탐색 가능 | 보이지 않는 조작 대상이 남는가? |

시각적으로 사라진 것처럼 보여도 키보드가 들어갈 수 있으면 혼란이 생긴다. 모달·팝오버처럼 포커스가 이동하는 UI는 열기·닫기 후 위치까지 설계해야 하며 해당 패턴의 세부 구현은 접근성 심화에서 다룬다.

---

## 11. 부모와 자식의 동작을 연결한다

자식이 부모의 상태를 직접 수정하는 대신 부모가 콜백 prop을 전달할 수 있다. 예를 들어 부모는 `onOpen`을 제공하고 자식 버튼은 이를 이벤트 시점에 호출한다. 사용자에게 의미 있는 동작 이름을 prop으로 두면 UI 구현과 상위 처리 흐름의 관계가 분명해진다.

폼은 onSubmit에서 기본 제출을 취소할지 정하고, 부모의 클릭과 자식 클릭이 함께 실행되는 경우에는 의도한 전파인지 먼저 확인한다. `return false`를 이벤트 제어의 일반 해법으로 사용하지 않는다. 라이브러리의 이벤트 처리 위에 브라우저의 기본 동작이 여전히 존재한다.

---

## 12. 사건 → 상태 → UI로 버그를 추적하기

클릭 자체가 안 잡히면 실제 컨트롤과 disabled·겹침·리스너 연결을 확인한다. 핸들러는 실행되는데 상태가 그대로면 setter와 현재 스냅샷을 확인한다. 상태는 맞는데 UI가 틀리면 조건의 타입과 JSX 반환값을 확인한다. DOM은 맞는데 사용이 어렵다면 숨김·포커스·이름을 확인한다.

조건에 따라 다른 컴포넌트 타입이나 key를 렌더링하면 상태가 초기화될 수 있다. 같은 문자열이 보인다는 이유만으로 같은 인스턴스라고 판단하지 않는다. 상태와 항목의 정체성은 바로 다음 List / Key에서 자세히 연결한다.

---

## 13. 이벤트 처리와 렌더링 계산의 실행 시점

### 전달할 함수와 지금 실행할 함수를 구별한다

React는 렌더링에서 JSX를 계산한다. onClick에 함수 호출 결과를 넣으면 클릭하기 전에 그 함수를 실행한다. 이벤트가 발생했을 때 실행할 동작은 함수로 전달해야 한다. 인수가 필요한 경우 화살표 함수로 감싸는 것은 그 시점을 표현하는 일반적인 코드이며, 모양만 보고 모두 잘못된 패턴이나 성능 문제로 취급하지 않는다.

### 사용자 사건이 외부 값을 바꾸는 지점

렌더링은 현재 props·state에 대한 UI를 계산하고, 핸들러는 사용자의 사건을 상태 갱신이나 외부 작업으로 연결한다. “함수 컴포넌트 안에 있으므로 어느 줄에서든 같은 일을 해도 된다”는 모델은 순수한 화면 계산과 실제 사건을 섞는다. [React 이벤트 안내](https://react.dev/learn/responding-to-events)를 실행 시점의 계약으로 읽는다.

### 실험 1. 인수가 있는 콜백 prop

```jsx
import { useState } from 'react';

function TopicButton({ topic, onChoose }) {
  return (
    <button type="button" onClick={() => onChoose(topic.id)}>
      {topic.title} 선택
    </button>
  );
}
export default function App() {
  const [selected, setSelected] = useState('없음');
  return (
    <main>
      <h1>선택: {selected}</h1>
      <TopicButton topic={{ id: 'html', title: 'HTML' }} onChoose={setSelected} />
      <TopicButton topic={{ id: 'dom', title: 'DOM' }} onChoose={setSelected} />
    </main>
  );
}
```

onChoose는 개발자가 정한 컴포넌트 계약이고 브라우저가 그 이름의 DOM 이벤트를 자동 발생시키는 것은 아니다. 자식은 실제 button의 onClick에서 필요한 id를 부모에 전달하고 부모가 선택 상태를 소유한다. 클릭 전에 선택이 바뀐다면 함수를 전달할 자리에서 호출하지 않았는지 먼저 확인한다.

---

## 14. React 이벤트 객체를 관찰하는 범위

### target·currentTarget과 값의 스냅샷

아이콘이나 span을 누르면 target은 내부 요소일 수 있고 currentTarget은 핸들러가 등록된 컨트롤이다. 객체 참조를 저장하는 것과 그 순간의 입력 문자열을 저장하는 것도 다르다. 비동기 작업 동안 DOM 값이 바뀔 수 있다면 업무에 사용할 값을 핸들러 안에서 복사해 둔다.

### 실험 2. 버튼 안 요소의 클릭 위치

```jsx
export default function App() {
  return (
    <main>
      <button type="button" data-topic="html" onClick={(event) => {
        console.log('출발:', event.target.tagName);
        console.log('등록 대상:', event.currentTarget.tagName);
        console.log('버튼의 데이터:', event.currentTarget.dataset.topic);
      }}>
        <span>HTML</span> 선택
      </button>
    </main>
  );
}
```

span 글자를 누르면 출발은 SPAN, 등록 대상은 BUTTON이다. 버튼의 나머지 부분이나 키보드 활성화에서는 출발 관찰이 달라질 수 있지만 버튼 자체의 데이터는 currentTarget에서 읽는 계약이 명확하다. event 객체의 모든 프로퍼티가 나중에도 같은 상태로 남는다고 가정하지 않는다.

### React의 이벤트 표면과 네이티브 예외

React DOM은 네이티브 이벤트 위에 핸들러 인터페이스를 제공한다. onFocus·onBlur의 전파와 onScroll의 예외 등은 네이티브 이벤트 이름을 그대로 대입한 모델과 다를 수 있다. 이 문서는 브라우저의 현대 React DOM을 기준으로 하며 React Native나 다른 라이브러리의 이벤트 규칙을 일반화하지 않는다. [React 이벤트 객체 레퍼런스](https://react.dev/reference/react-dom/components/common#react-event-object)를 확인한다.

---

## 15. 기본 동작 취소와 부모 핸들러 실행을 나누기

### 실험 3. 폼 제출은 취소해도 부모 관찰은 남는다

```jsx
import { useState } from 'react';

export default function App() {
  const [message, setMessage] = useState('대기');
  return (
    <main onClick={() => console.log('main에서 click 관찰')}>
      <form onSubmit={(event) => {
        event.preventDefault();
        const data = new FormData(event.currentTarget);
        setMessage(`제출 데이터: ${data.get('topic')}`);
      }}>
        <label htmlFor="topic">학습 주제</label>
        <input id="topic" name="topic" defaultValue="HTML" required />
        <button type="submit">데이터 확인</button>
      </form>
      <p role="status">{message}</p>
    </main>
  );
}
```

기본 제출을 취소하여 문서 이동은 하지 않지만 부모의 click 관찰을 지우지 않는다. Enter 제출도 버튼 클릭 로그와 완전히 같은 입력 경로라고 가정하지 않는다. form의 onSubmit에서 제출을 처리하고, 일반 버튼이면 type button으로 의도를 명시한다. 입력은 여기서 FormData로 현재 값을 읽으며 제어 방식의 상세는 마지막 입력 페이지에서 연결한다.

### 전파를 막는 대신 처리 책임을 분리할 수 있다

부모 카드 클릭과 자식 버튼 클릭이 충돌하면 DOM 구조와 동작 목적부터 점검한다. 링크 안에 버튼을 잘못 중첩한 문제를 stopPropagation만으로 덮지 않는다. 부모가 자신이 처리할 대상인지 확인하는 방법과 자식 전파 중단의 영향을 비교하여, 상위 관찰·키보드 기능이 필요한지 함께 판단한다.

---

## 16. 조건부 렌더링에서 JavaScript의 반환값을 읽기

### 실험 4. 0·false·null의 차이

```jsx
export default function App() {
  const count = 0;
  return (
    <main>
      <h1>조건식의 결과</h1>
      <p>A: {count && <strong>결과 있음</strong>}</p>
      <p>B: {count > 0 && <strong>결과 있음</strong>}</p>
      <p>C: {count > 0 ? '결과 있음' : '결과 없음'}</p>
    </main>
  );
}
```

A는 AND의 반환값인 숫자 0을 표시한다. B는 boolean false라 해당 추가 UI가 없고 C는 빈 결과를 명시적으로 설명한다. 조건식이 자동으로 boolean으로만 반환된다는 가정이 오류의 원인이다. [React 조건부 렌더링 문서](https://react.dev/learn/conditional-rendering)의 빈 위치와 숫자 표시 규칙을 적용한 예다.

### 분기 방식은 읽기와 반환 결과로 선택한다

두 갈래 문구는 삼항이 간단하고, 여러 상태의 큰 화면은 먼저 if로 반환하면 명확할 수 있다. 중첩 삼항을 계속 늘리기보다 상태 이름·조건의 배타성·반환 UI를 정리한다. JSX 밖에서 계산할 중간 값과 UI 자체를 분리하면 디버깅하기도 쉬워진다.

### Hook 호출 순서는 UI 조건과 분리한다

조건에 따라 useState를 호출하거나 건너뛰지 않는다. 필요한 Hook은 최상위에서 호출하고, 그 뒤 어떤 UI를 반환할지 결정한다. UI를 숨기거나 null을 반환하는 것과 Hook 호출을 건너뛰는 것은 다른 일이다. 내부 원리는 후속 React Mental Model에서 다룬다.

---

## 17. 배타적인 화면 상태를 이름으로 표현하기

### 여러 불리언이 만드는 모순 상태

loading·error·success를 독립 boolean으로 두면 둘 이상이 동시에 참이 되는 조합을 만들 수 있다. 항상 하나의 상태만 의미 있다면 status 한 값이 모순 조합을 줄일 수 있다. 반대로 서로 독립적인 플래그까지 억지로 하나로 합칠 필요는 없다. 실제 데이터 관계로 판단한다.

### 실험 5. 상태별 UI를 검증하는 작은 도구

```jsx
import { useState } from 'react';

function Result({ status }) {
  if (status === 'loading') return <p>목록을 불러오고 있습니다.</p>;
  if (status === 'error') return <p>목록을 불러오지 못했습니다. 다시 시도하세요.</p>;
  if (status === 'empty') return <p>조건에 맞는 결과가 없습니다.</p>;
  if (status === 'success') return <p>학습 항목 세 개를 찾았습니다.</p>;
  return <p>검색을 시작하세요.</p>;
}
export default function App() {
  const [status, setStatus] = useState('idle');
  return (
    <main>
      <h1>검색 상태 미리보기</h1>
      <button type="button" onClick={() => setStatus('loading')}>로딩</button>
      <button type="button" onClick={() => setStatus('empty')}>빈 결과</button>
      <button type="button" onClick={() => setStatus('error')}>실패</button>
      <button type="button" onClick={() => setStatus('success')}>성공</button>
      <div><Result status={status} /></div>
      <p role="status">현재 상태: {status}</p>
    </main>
  );
}
```

이 예제는 네트워크를 수행하지 않고 상태별 화면을 직접 시험하는 도구다. 실제 요청의 시작·실패·빈 배열·성공이 어느 상태로 이동해야 하는지 설계한 뒤 연결한다. 로딩 문구만 바꾸고 이전 결과가 남아 잘못된 상태를 암시하는지, 다시 시도할 경로가 있는지 등을 확인한다.

---

## 18. 조건부 제거·숨김·비활성화를 목적에 맞게 고르기

### 같은 시각 결과라도 사용자 경로가 다르다

조건에서 제외하면 해당 하위 UI가 렌더 트리에서 빠질 수 있고, hidden은 DOM을 유지하면서 일반적으로 표시·접근성 탐색에서 숨긴다. disabled는 컨트롤을 남기되 조작과 폼 데이터 등의 조건을 바꾼다. opacity 0만 적용하면 보이지 않아도 포커스할 수 있는 대상이 남을 수 있다. 모양만으로 세 방법을 치환하지 않는다.

### 실험 6. 관계와 상태를 가진 접기 UI

```jsx
import { useState } from 'react';

function Disclosure({ id, title, children }) {
  const [open, setOpen] = useState(false);
  return (
    <section>
      <h2>{title}</h2>
      <button type="button" aria-expanded={open} aria-controls={id}
        onClick={() => setOpen((value) => !value)}>
        {open ? '설명 접기' : '설명 보기'}
      </button>
      <div id={id} hidden={!open}>{children}</div>
    </section>
  );
}
export default function App() {
  return (
    <main>
      <h1>학습 안내</h1>
      <Disclosure id="html-description" title="HTML">
        <p>문서의 구조와 의미를 학습합니다.</p>
        <a href="https://html.spec.whatwg.org/">HTML 표준</a>
      </Disclosure>
      <Disclosure id="dom-description" title="DOM">
        <p>노드의 관계와 변경을 학습합니다.</p>
      </Disclosure>
    </main>
  );
}
```

각 컨트롤의 aria-controls가 고유한 영역을 가리키고 aria-expanded는 open과 같은 기준을 사용한다. 버튼 문구·영역 표시·의미 상태가 같은 렌더링에서 맞춰진다. [WAI Disclosure 패턴](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)은 역할·관계·키보드 조작을 확인하는 공식 안내다.

### 포커스가 있는 영역을 숨길 때

내부 링크를 조작 중인데 다른 코드가 영역을 닫으면 현재 작업 위치가 사라질 수 있다. 어떤 사건이 닫기를 실행하며 포커스를 어디로 되돌릴지 정한다. 버튼·DOM 상태만 테스트하지 말고 Tab·Shift+Tab·Enter·Space와 실제 보조 기술에서도 같은 흐름을 확인한다.

---

## 19. 사건에서 최종 UI까지의 검증 절차

### 핸들러가 실행되는가

실제 렌더링된 컨트롤, disabled, 다른 요소의 가림, 함수 전달 여부, 이벤트 종류를 확인한다. 부모·자식 로그가 함께 나오는 것은 우선 전파 모델로 설명하고 중복 등록으로 단정하지 않는다.

### 상태와 조건이 같은 사실을 표현하는가

setter가 요청한 값과 다음 렌더링의 조건식을 확인한다. 문자열 loading과 boolean 조건을 섞거나 0을 빈 위치로 기대하고 있지 않은지 살핀다. 상태가 맞는데 화면이 틀리면 반환 JSX와 숨김 속성·CSS를 확인한다.

### 사용자가 변화를 인지하고 계속 작업할 수 있는가

오류·빈 결과·완료 안내, 포커스, 컨트롤 이름과 상태를 검사한다. 상태 영역·ARIA 속성이 있다는 사실만으로 모든 브라우저·보조 기술의 안내를 보장하지 않으므로 지원 환경에서 확인한다. 이벤트 처리의 정확성과 접근 가능한 기능 완료는 함께 검증해야 한다.

---

## 20. 흔한 문제와 실무 검증

렌더링 중 setter를 무조건 호출하면 계속 렌더링을 요청할 수 있다. 핸들러를 전달해야 할 자리에 호출하고 있지 않은지 확인한다. DevTools에서 state가 바뀌는지, 그 값으로 JSX의 조건이 결정되는지, 최종 DOM 속성이 의도대로 변하는지 순서대로 추적한다.

조건부로 없어진 영역에 포커스가 있었다면 다음 포커스 위치를 설계한다. 표시 여부만 바꾸면서 키보드·보조 기술에 계속 노출되는 CSS를 쓴 경우도 점검한다. 이벤트 핸들러의 긴 계산은 사용자 입력을 늦추므로 필요하면 측정하고 작업 분리를 검토한다.

---

## 21. 핵심 정리

<callout icon="✅" color="green_bg">
핸들러는 사용자 동작을 상태 갱신으로 연결하고 조건부 렌더링은 현재 상태를 화면으로 연결한다. 함수 전달·조건의 타입·제거와 숨김·포커스를 구별해야 전체 흐름이 일관된다.
</callout>

---

## 출처

- [React — Responding to Events](https://react.dev/learn/responding-to-events)
- [React — Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [React — Common DOM Component Props](https://react.dev/reference/react-dom/components/common)
- [WAI ARIA APG — Disclosure Pattern](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/)

### 이번 설명의 근거와 적용 범위

- [React — Responding to Events](https://react.dev/learn/responding-to-events): 핸들러 전달·콜백 prop·전파와 기본 동작 제어.
- [React — Conditional Rendering](https://react.dev/learn/conditional-rendering): if·삼항·AND·빈 위치·숫자 0의 결과.
- [React — Event Object](https://react.dev/reference/react-dom/components/common#react-event-object): target·currentTarget·React DOM 이벤트 표면과 예외.
- [W3C WAI — Disclosure](https://www.w3.org/WAI/ARIA/apg/patterns/disclosure/): 접기 UI의 의미 상태·관계·키보드 요구.

---

## 22. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. onClick에 함수를 전달하는 것과 호출하는 것의 실행 시점 차이를 설명하라.
2. `count && <p>결과</p>`에서 count가 0일 때 예상하지 않은 숫자가 보이는 이유는 무엇인가?
3. hidden 속성으로 숨기는 방법과 조건부로 제거하는 방법을 DOM·포커스 관점에서 비교하라.
4. 접기 UI를 여러 개 만들었더니 한 버튼이 잘못된 영역을 가리킨다. 어떤 접근성 연결을 확인하겠는가?
