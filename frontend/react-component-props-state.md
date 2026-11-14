---
track: B
notion: https://app.notion.com/p/415178c4afac81819917deb646d1a549
---

# Component / Props / State


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · React Basics 복습 · Component / Props / State

컴포넌트의 구조·props의 계약·state의 소유와 갱신을 각각 배우고 상태에서 DOM까지 이어지는 흐름을 이해한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 기본 DOM 요소·사용자 컴포넌트·JSX의 역할을 구별할 수 있다.
- 객체·함수·children과 기본값을 props로 전달하는 계약을 설명할 수 있다.
- 저장할 state와 현재 값에서 계산할 데이터를 구분하고 소유 위치를 정할 수 있다.
- 스냅샷·함수형 갱신·객체의 불변 갱신을 예제로 설명할 수 있다.
- props·state·렌더링·커밋·브라우저 표현의 문제 지점을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

DOM API만으로 UI를 만들면 사용자의 동작마다 어떤 요소와 문구를 바꿀지 직접 추적해야 한다. React에서는 현재 데이터에 대한 화면을 선언하고, 상태가 바뀌면 다시 화면을 계산하도록 한다. 이 방식의 기초가 화면 단위인 Component, 외부 입력인 Props, 기억해야 할 값인 State다.

---

## 3. 관련 계층과 역할 구분

JavaScript는 컴포넌트 함수와 데이터 연산을 실행한다. React는 UI 트리와 상태 갱신을 관리하고 React DOM은 브라우저 DOM 반영을 담당한다. CSS와 브라우저는 최종 크기·배치·화면을 처리하며 실제 DOM 요소가 의미와 기본 조작을 제공한다.

컴포넌트가 호출되었다는 것, DOM이 변경되었다는 것, 사용자가 새 픽셀을 보았다는 것을 같은 단계로 표현하지 않는다. 함수 컴포넌트와 Hook을 사용하는 브라우저 React 예제를 기준으로 설명한다.

---

## 4. 컴포넌트는 현재 입력에 대한 화면을 설명한다

React 함수 컴포넌트는 props를 받아 렌더링할 UI를 반환하는 함수다. 이름은 대문자로 시작하여 기본 HTML 요소와 구별한다. JSX는 JavaScript 안에서 UI 구조를 표현하는 문법이며 HTML 문자열 자체가 아니다. 프로젝트의 빌드 설정 등이 JSX를 처리하고 React DOM이 브라우저 DOM 갱신을 맡는다. React는 JavaScript 언어·DOM·브라우저를 대체하는 실행 환경이 아니다.

| 개념 | 값의 소유·변경 | 사용 목적 |
| --- | --- | --- |
| props | 부모가 전달, 받는 쪽은 읽기 전용으로 취급 | 제목·설정·콜백 등 외부 입력 |
| state | 해당 상태를 소유한 컴포넌트가 setter로 갱신 요청 | 상호작용 뒤에도 기억할 값 |
| 렌더링 중의 지역 변수 | 함수 호출마다 계산 | 현재 props·state에서 구할 수 있는 값 |

props가 읽기 전용이라는 말은 평생 같은 값이라는 뜻이 아니다. 부모가 새로운 값을 전달하면 다음 렌더링에서 다른 props를 받는다. 다만 자식이 전달받은 객체를 직접 수정해서는 안 된다. [React Props 안내](https://react.dev/learn/passing-props-to-a-component)가 이 데이터 흐름을 설명한다.

---

## 5. Component: 구조와 책임을 가진 UI 단위

### 기본 요소와 사용자 컴포넌트

`<button>`은 React DOM이 아는 기본 DOM 요소를 표현하고 `<StudyCounter>`는 개발자가 정의한 컴포넌트 타입을 표현한다. 사용자 컴포넌트 이름은 대문자로 시작한다. 컴포넌트를 정의했다고 문서에 자동으로 보이는 것이 아니라 상위 UI 트리에 포함하여 렌더링해야 한다.

```jsx
function StudyTitle({ text }) {
  return <h2>{text}</h2>;
}

function StudyPanel() {
  return (
    <section className="study-panel">
      <StudyTitle text="문서 구조" />
      <p>문서의 의미와 관계를 살펴봅니다.</p>
    </section>
  );
}
```

기존 React 프로젝트에서 StudyPanel을 App에 포함하여 실행할 수 있다. 브라우저 DOM에는 StudyPanel이나 StudyTitle이라는 태그가 자동으로 생기지 않고 section·h2·p가 생긴다. 컴포넌트의 분리는 코드의 책임 단위이며 불필요한 래퍼 요소를 반드시 추가해야 하는 것은 아니다.

### JSX가 HTML과 다른 부분

JSX는 중괄호 안에 JavaScript 식을 넣고 className·htmlFor 같은 React DOM prop 이름을 사용한다. 여러 형제를 반환할 때는 적절한 부모나 Fragment로 묶을 수 있다. JSX의 자기 닫기 문법과 실제 HTML 파서의 void element 규칙은 서로 다른 문법 층이다.

텍스트에 넣은 일반 문자열은 기본적으로 마크업으로 파싱하는 innerHTML과 다르게 처리된다. 다만 URL·직접 HTML 삽입 등 별도 경로의 보안 요구까지 모두 자동으로 사라지는 것은 아니다. 렌더링되는 최종 HTML 구조와 컨트롤의 의미를 확인한다.

### 언제 컴포넌트로 나눌까

반복되는 UI, 독립적인 의미, 별도로 다룰 상태·책임이 있는 영역을 기준으로 나눈다. 모든 div를 별도 컴포넌트로 만들거나 파일 길이만으로 기계적으로 분리할 필요는 없다. 다른 화면에서도 의미가 통하는 API인지, 데이터와 동작의 경계를 이해하기 쉬운지 확인한다.

---

## 6. Props: 데이터와 동작을 입력으로 받기

### 값과 함수와 children을 전달한다

```jsx
function StudyCard({ topic, onStart, children }) {
  return (
    <article>
      <h2>{topic.title}</h2>
      {children}
      <button type="button" onClick={() => onStart(topic.id)}>학습 시작</button>
    </article>
  );
}
```

topic은 객체, onStart는 함수, children은 태그 사이의 UI를 받는다. 부모는 현재 데이터와 처리 함수를 제공하고 자식은 props의 계약에 맞게 UI를 표현한다. 자식이 topic.title을 직접 바꾸거나 onStart를 렌더링 중 실행하는 방식으로 외부 상태를 변경하지 않는다.

### 기본값과 null은 다르다

함수 매개변수의 기본값은 prop이 없거나 undefined일 때 적용된다. 명시적으로 null·0·빈 문자열을 준 경우까지 모두 기본값으로 바뀌지는 않는다. 값이 없음을 어떤 값으로 표현할지 부모와 자식의 계약을 정한다.

### 받은 값을 복사해 state로 두면 자동 동기화되지 않는다

```jsx
function DraftTitle({ initialTitle }) {
  const [title, setTitle] = useState(initialTitle);
  return <p>{title}</p>;
}
```

이 조각은 useState를 import한 React 환경을 전제로 한다. initialTitle은 첫 상태를 정하는 데 사용되며 이후 prop이 바뀐다고 state가 자동으로 따라 바뀌는 계약은 아니다. 현재 prop을 그대로 보여주면 되는지, 독립 편집 초안을 만들려는지 먼저 결정한다. 편집과 서버 값의 충돌 정책을 무의식적인 복사로 숨기지 않는다.

---

## 7. 예제: 두 개의 독립된 학습 카운터

기존 React 웹 프로젝트의 `App.jsx`를 아래 내용으로 바꾸어 실행한다. JSX 파일은 HTML에 일반 script로 그대로 연결하는 코드가 아니다.

```jsx
import { useState } from 'react';

function StudyCounter({ topic, target = 3 }) {
  const [count, setCount] = useState(0);
  const remaining = Math.max(target - count, 0);

  function addPractice() {
    setCount((previous) => previous + 1);
  }

  return (
    <section>
      <h2>{topic}</h2>
      <p>실습 {count}회 / 목표 {target}회</p>
      <p>남은 횟수: {remaining}</p>
      <button type="button" onClick={addPractice}>실습 추가</button>
    </section>
  );
}

export default function App() {
  return (
    <main>
      <h1>오늘의 학습</h1>
      <StudyCounter topic="HTML" target={2} />
      <StudyCounter topic="DOM" />
    </main>
  );
}
```

HTML의 버튼을 누르면 HTML의 count만 증가한다. 같은 컴포넌트 함수를 사용해도 각 렌더링 위치의 상태는 독립적이다. DOM의 target은 생략했으므로 기본값 3을 받는다. `remaining`은 count와 target으로 계산할 수 있으므로 별도 state로 저장하지 않았다. 중복 상태를 줄이면 두 값이 어긋나는 문제도 줄어든다.

useState는 현재 상태와 갱신을 요청하는 setter를 제공하는 Hook이다. 이 예제에서는 컴포넌트 최상위에서 호출한다. 조건문·반복문·일반 이벤트 함수 안에서 임의로 호출하지 않는다. Hook의 상세한 동작 원리는 후속 React Mental Model에서 다룬다.

---

## 8. 클릭에서 화면까지의 변화

처음 렌더링할 때 React는 함수를 호출해 UI를 계산하고 React DOM은 필요한 요소를 문서에 반영한다. 클릭 핸들러가 setter를 호출하면 React에 상태 갱신을 요청한다. 다음 렌더링에서 새 count로 UI를 계산하고, 커밋 단계에서 필요한 DOM 변경을 반영한다. 브라우저는 이 DOM과 스타일을 바탕으로 화면을 갱신한다.

따라서 렌더링은 항상 DOM 전체를 삭제하고 새로 만드는 작업을 뜻하지 않는다. 또한 setter 호출 직후 현재 함수의 지역 변수 count가 바뀌는 것도 아니다. 현재 렌더링에서 얻은 count는 그 렌더링의 스냅샷이다. [State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)이 이 관점을 설명한다.

```jsx
function addTwo() {
  setCount((previous) => previous + 1);
  setCount((previous) => previous + 1);
}
```

이전 값에 의존하는 갱신에는 함수형 갱신을 쓰면 큐의 이전 결과를 바탕으로 다음 값을 계산한다. 반면 같은 핸들러에서 `setCount(count + 1)`을 두 번 쓰면 같은 스냅샷으로 같은 다음 값을 요청할 수 있다. 갱신 함수는 부수 효과 없이 값을 계산해야 한다.

---

## 9. State: 저장할 것·계산할 것·공유할 것

### 값이 바뀔 수 있다고 모두 state는 아니다

props와 state에서 바로 계산할 수 있는 합계·필터 결과·남은 횟수는 렌더링 중 계산할 수 있다. 같은 사실을 여러 state로 중복 저장하면 하나만 갱신하여 불일치가 생길 수 있다. 반면 사용자의 선택처럼 다음 렌더링까지 기억해야 하고 화면에 영향을 주는 정보는 state 후보가 된다.

state 소유 위치는 여러 사용자의 동기화 요구로 정한다. 형제 두 곳에서 같은 선택을 보여야 한다면 공통 부모의 상태 하나를 내려주는 것이 각자 복사한 상태를 맞추는 것보다 명확할 수 있다. 서로 독립적이어야 할 값까지 하나의 전역 상태로 합치지는 않는다.

### 스냅샷을 캡처한 핸들러

한 렌더링에서 만든 핸들러는 그 렌더링의 props·state를 기준으로 동작한다. setter는 그 지역 변수에 값을 대입하는 API가 아니다. 같은 핸들러 안에서 이전 state에 의존하는 갱신을 여러 번 하면 함수형 갱신으로 큐의 이전 결과를 사용한다.

이것을 “setter는 무조건 특정 밀리초 뒤 실행된다” 같은 타이머 규칙으로 외우지 않는다. 중요한 기준은 현재 렌더링의 값과 다음 렌더링을 요청하는 작업의 구분이다. 내부 스케줄링·배칭 상세는 4–6주 React Mental Model에서 확장한다.

### 객체의 변경은 바뀐 경로를 새로 만든다

```jsx
const [profile, setProfile] = useState({
  name: 'Learner',
  settings: { theme: 'light', fontSize: 16 },
});

function applyDarkTheme() {
  setProfile((previous) => ({
    ...previous,
    settings: { ...previous.settings, theme: 'dark' },
  }));
}
```

컴포넌트 최상위에서 상태를 선언한 조각이다. 바깥 객체뿐 아니라 변경한 settings 경로도 새 객체로 만든다. 바깥만 복사하고 공유 중인 settings를 직접 변경하면 이전 스냅샷과 다른 참조에서도 변화가 보일 수 있다. 이것은 JavaScript의 참조 공유 위에서 React 상태를 다루는 문제다.

새 객체를 만들 때마다 무조건 깊은 복사를 수행하는 것이 목적은 아니다. 바뀌지 않은 하위 값은 유지하고 바뀐 경로를 명확히 만든다. React는 같은 값 여부 판단에 `Object.is`를 사용하는 경우가 있으므로 기존 객체를 바꾸어 같은 참조를 넘기는 방식에 의존하지 않는다.

---

## 10. 상태를 직접 바꾸지 않는다

일반 변수를 `let count = 0`으로 두고 click에서 증가시키면 React에 렌더링을 요청하지 못하며 다음 호출에서도 보존되지 않는다. 객체·배열 state도 기존 값을 직접 변경하기보다 변경된 부분을 새 객체·배열로 만들어 setter에 전달한다. 같은 객체를 수정한 뒤 그대로 넘기면 React의 변경 판단과 이전 렌더링의 값 보존을 깨뜨릴 수 있다.

여러 자식이 같은 값을 공유해야 하면 공통 부모가 상태를 소유하고 props와 콜백으로 전달하는 구성을 검토한다. 모든 값을 최상위에 모을 필요는 없다. 값을 사용하는 범위와 동기화 요구를 기준으로 소유 위치를 정한다.

---

## 11. 렌더링이 안전하게 반복될 수 있어야 한다

같은 입력에서 화면 설명을 계산하는 렌더링 단계에 네트워크 요청·외부 데이터 변경·DOM 직접 변경을 무조건 섞으면 재호출 시 효과가 중복될 수 있다. 사용자 동작은 이벤트 핸들러에서, 외부 시스템과의 동기화는 해당 후속 학습의 수명 규칙으로 구별한다.

컴포넌트 함수를 다른 컴포넌트 함수 안에 새로 정의하는 방식도 상태 보존과 타입 정체성에 영향을 줄 수 있다. 재사용할 컴포넌트 정의는 보통 모듈 수준에 둔다. 구체적인 보존 조건은 뒤의 List / Key에서 연결한다.

### 데이터 흐름을 따라 디버깅한다

부모의 입력이 맞는지, 자식이 받은 props가 맞는지, state 소유자가 맞는지, setter에 새 값을 전달하는지, 그 값으로 어떤 JSX를 계산하는지 순서대로 확인한다. 화면이 안 바뀐다고 DOM을 직접 수정하여 상태와 화면을 따로 관리하지 않는다.

---

## 12. 컴포넌트가 반환하는 것은 현재 입력의 UI 설명이다

### 함수 정의·JSX·DOM의 단계

함수를 정의한 것, JSX에서 그 타입을 사용한 것, 브라우저 DOM에 요소가 반영된 것은 다른 단계다. React는 컴포넌트를 호출하여 UI를 계산하고 React DOM은 필요한 문서 변경을 적용한다. 사용자 컴포넌트 이름이 자동으로 실제 HTML 태그가 되는 것은 아니다. 의미와 접근성은 최종 button·heading·list 등의 요소와 속성으로 확인한다.

### 합성은 래퍼 div를 무조건 추가하는 작업이 아니다

코드를 컴포넌트로 나누어도 의미상 필요한 DOM 관계를 유지해야 한다. li가 ul 아래에 있어야 하는 구조를 컴포넌트 경계가 면제하지 않는다. 여러 요소를 묶어 반환할 때 Fragment 같은 방법을 검토하고, 화면 배치만 필요한 래퍼와 의미 있는 컨테이너를 구별한다.

### React 예제의 실행 범위

이 절의 독립 예제는 기존 React 웹 프로젝트의 App.jsx를 교체하여 실행한다. 필요한 import를 포함한 각 예제를 따로 사용하며 여러 App을 한 파일에 동시에 선언하지 않는다. 빌드 도구가 JSX를 처리하는 환경이 필요하고, 아래 파일을 일반 HTML script로 그대로 읽는 예제는 아니다.

---

## 13. Props의 값·기본값·children을 명확한 계약으로 사용하기

### 실험 1. 기본값과 null·0의 차이

```jsx
function Amount({ value = 5, label }) {
  return <p>{label}: {String(value)}</p>;
}

export default function App() {
  return (
    <main>
      <h1>기본값의 범위</h1>
      <Amount label="생략" />
      <Amount label="undefined" value={undefined} />
      <Amount label="null" value={null} />
      <Amount label="0" value={0} />
    </main>
  );
}
```

표시 값은 순서대로 5, 5, null, 0이다. 이 예제는 null 자체를 JSX로 넣어 숨기는 대신 String으로 표시하여 받은 값을 관찰한다. 부모가 값 없음으로 무엇을 전달하는지와 자식의 기본값 적용 조건을 맞춰야 한다. [React props 문서](https://react.dev/learn/passing-props-to-a-component)는 기본 매개변수와 읽기 전용 입력의 의미를 설명한다.

### 실험 2. children과 함수 prop의 책임

```jsx
import { useState } from 'react';

function Panel({ title, onOpen, children }) {
  return (
    <section>
      <h2>{title}</h2>
      {children}
      <button type="button" onClick={onOpen}>이 항목 선택</button>
    </section>
  );
}
export default function App() {
  const [selected, setSelected] = useState('없음');
  return (
    <main>
      <h1>현재 선택: {selected}</h1>
      <Panel title="HTML" onOpen={() => setSelected('HTML')}>
        <p>문서의 구조와 의미를 살펴봅니다.</p>
      </Panel>
      <Panel title="DOM" onOpen={() => setSelected('DOM')}>
        <p>문서 객체의 변경을 살펴봅니다.</p>
      </Panel>
    </main>
  );
}
```

Panel은 전달받은 제목·내용을 표현하고 사용자의 선택을 부모 콜백에 연결한다. selected의 소유자는 App 하나다. 자식이 부모의 객체를 직접 수정하거나 선택값을 각자 복제하여 맞추는 구조가 아니다. 하나의 컴포넌트가 외부 입력과 내부 상태 중 무엇을 소유하는지 명확히 하는 것이 데이터 흐름의 출발점이다.

### 읽기 전용은 변하지 않는다는 뜻이 아니다

부모가 새 제목을 전달하면 자식은 다음 렌더링에서 다른 props를 받는다. 읽기 전용은 자식이 받은 값을 직접 변경하여 외부에 영향을 주지 말라는 계약이다. const로 받았다는 문법 하나가 깊은 객체 변경을 막는다고 생각하지 않는다. 참조 공유의 성질 위에 상태·props 사용 규칙을 적용해야 한다.

---

## 14. 지역 변수와 React state의 차이를 직접 확인하기

### 실험 3. 변수는 증가하지만 UI가 그대로인 이유

```jsx
import { useState } from 'react';

export default function App() {
  let localCount = 0;
  const [renders, setRenders] = useState(0);
  return (
    <main>
      <h1>지역 변수와 state</h1>
      <p>현재 렌더링의 지역 값: {localCount}</p>
      <p>재계산 요청: {renders}</p>
      <button type="button" onClick={() => {
        localCount += 1;
        console.log('핸들러가 가진 지역 값:', localCount);
      }}>지역 변수 증가</button>
      <button type="button" onClick={() => setRenders((value) => value + 1)}>
        state로 재계산 요청
      </button>
    </main>
  );
}
```

지역 증가 버튼을 반복하면 같은 렌더링에서 만들어진 핸들러가 붙잡은 변수가 1·2처럼 증가할 수 있다. 그러나 React에 새 UI 계산을 요청한 것이 아니므로 표시 문구는 그대로다. state 버튼으로 다시 렌더링하면 함수의 새 지역 변수가 0에서 시작한다. “일반 변수는 호출 즉시 무조건 사라진다”가 아니라 보존·갱신을 React와 연결하는 계약이 없다는 뜻이다.

### state는 컴포넌트 함수 파일 하나에 붙은 전역 값이 아니다

같은 컴포넌트 타입을 두 위치에서 사용해도 각 인스턴스의 상태는 독립적이다. 여러 인스턴스가 정말 같은 값을 보여야 하면 공통 소유자가 값을 관리해 전달한다. 파일을 재사용한다는 것과 상태를 공유한다는 것은 다른 요구다.

---

## 15. 상태 스냅샷과 갱신 큐를 순서대로 읽기

### 현재 렌더링의 변수와 다음 렌더링 요청

setter를 호출해도 현재 함수가 받은 count가 다른 값으로 재할당되지는 않는다. 현재 핸들러는 해당 렌더링의 스냅샷을 사용한다. [State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)의 핵심은 특정 밀리초만큼 지연된다는 타이머 규칙이 아니라 현재 값과 다음 계산 요청의 구분이다.

### 실험 4. 같은 값 요청과 함수형 갱신

```jsx
import { useState } from 'react';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <main>
      <h1>현재 값: {count}</h1>
      <button type="button" onClick={() => {
        setCount(count + 1);
        setCount(count + 1);
        console.log('현재 핸들러:', count);
      }}>값으로 두 번 요청</button>
      <button type="button" onClick={() => {
        setCount((value) => value + 1);
        setCount((value) => value + 1);
      }}>함수로 두 번 요청</button>
      <button type="button" onClick={() => setCount(0)}>0으로 초기화</button>
    </main>
  );
}
```

0에서 각각 따로 실험하면 값 요청은 1, 함수 요청은 2로 바뀐다. 첫 방식은 같은 스냅샷으로 계산한 1을 두 번 요청한다. 둘째는 큐의 이전 결과에 다시 1을 더한다. 콘솔은 현재 핸들러의 스냅샷을 출력한다. 개발 모드의 순수성 검사 때문에 갱신 함수가 추가 호출되더라도 순수한 계산의 최종 의미가 두 배가 되어서는 안 된다.

### 값 대체와 갱신 함수가 섞이면 순서가 중요하다

count가 0인 상태에서 setCount(5) 다음에 증가 함수를 요청하면 큐의 대체 결과 5를 받아 6을 만들 수 있다. 반대로 증가 함수 뒤에 setCount(5)를 요청하면 마지막 대체가 5를 정한다. [React의 갱신 큐 설명](https://react.dev/learn/queueing-a-series-of-state-updates)을 순서가 있는 연산으로 읽고, setter 횟수를 결과 증가 횟수로 세지 않는다.

### setter는 완료 Promise가 아니다

await setCount(...)를 사용해 다음 커밋 완료를 기다리는 계약으로 삼지 않는다. setter의 반환은 새로운 UI가 픽셀로 표시되었다는 완료 신호가 아니다. 후속 외부 시스템 동기화와 수명 처리는 해당 후속 React 학습에서 다룬다. 지금은 상태 갱신 요청과 DOM 반영 단계를 구별한다.

---

## 16. 초기 상태와 이후 props 변경의 관계

### 실험 5. props에서 만든 초기 state는 자동 복사되지 않는다

```jsx
import { useState } from 'react';

function Draft({ initialTitle }) {
  const [title] = useState(initialTitle);
  return <p>받은 prop: {initialTitle} / 보관한 초안: {title}</p>;
}
export default function App() {
  const [topic, setTopic] = useState('HTML');
  return (
    <main>
      <button type="button" onClick={() => setTopic('DOM')}>부모 값 변경</button>
      <Draft initialTitle={topic} />
    </main>
  );
}
```

버튼 후 prop은 DOM이지만 초안은 HTML로 남을 수 있다. 첫 상태를 정한 것과 props에 계속 동기화하는 계약은 다르다. 최신 prop만 보여주면 되는 값이라면 중복 state를 만들지 않는다. 독립 편집 초안이라면 언제 새 대상으로 초기화할지와 사용자 편집을 어떻게 보존할지 정책을 정한다.

### 초기화 함수를 전달하는 이유

useState(createInitial())은 함수 호출 결과를 인수로 전달하므로 컴포넌트가 호출될 때 그 표현식도 평가된다. useState(createInitial)은 React가 초기화 함수로 사용할 수 있게 함수를 전달한다. 초기화 함수는 순수해야 하고 개발 모드에서는 추가 호출로 검사될 수 있다. [useState 공식 레퍼런스](https://react.dev/reference/react/useState)의 초기화·반환·동일 값 조건을 확인한다.

---

## 17. 객체·배열 상태를 변경 경로에 따라 갱신하기

### 실험 6. 중첩된 값의 새 경로 만들기

```jsx
import { useState } from 'react';

export default function App() {
  const [profile, setProfile] = useState({
    name: 'Learner', settings: { theme: 'light', fontSize: 16 },
  });
  return (
    <main>
      <p>{profile.name}: {profile.settings.theme}</p>
      <button type="button" onClick={() => setProfile((previous) => ({
        ...previous,
        settings: { ...previous.settings, theme: 'dark' },
      }))}>어두운 테마</button>
    </main>
  );
}
```

바깥 객체와 변경된 settings 경로를 새로 만들고 바뀌지 않은 name·fontSize는 보존한다. 바깥만 복사한 뒤 이전 settings를 직접 수정하면 이전 스냅샷을 공유한 곳에서도 변화가 보일 수 있다. [객체 state 갱신](https://react.dev/learn/updating-objects-in-state)은 이 참조 관계를 근거로 설명한다.

### 배열도 같은 소유 규칙을 따른다

추가는 새 배열, 삭제는 filter, 특정 항목 변경은 map으로 새 배열과 변경 객체를 만들 수 있다. 배열만 복사한 뒤 그 안의 공유 객체를 직접 바꾸는 것도 같은 문제를 남긴다. 불변 갱신은 모든 값을 무조건 깊은 복사하는 관습이 아니라 바뀐 경로와 이전 상태의 의미를 보존하는 방법이다.

### 파생 값과 중복 상태

합계·남은 횟수·선택된 id로 찾을 수 있는 객체를 각각 별도 state로 중복 저장하면 하나만 갱신하여 불일치할 수 있다. 현재 원본에서 계산할 값과 사용자 사건 이후 기억해야 할 값을 구별한다. [상태 구조 선택](https://react.dev/learn/choosing-the-state-structure)의 중복·모순 상태 회피를 제품의 데이터 관계에 적용한다.

---

## 18. 순수한 렌더링과 실제 사용 가능성을 함께 확인하기

### 실행 횟수에 의존하지 않는 화면 계산

렌더링 중 전달받은 배열을 push하거나 외부 카운터를 증가시키면 재호출 시 다른 결과가 될 수 있다. 같은 입력에서 UI를 계산하고, 사용자의 실제 사건은 핸들러로 연결한다. 외부 시스템과의 동기화는 별도 수명 규칙을 가진다. 개발 Strict Mode에서 추가 호출이 관찰되면 그것을 숨기기보다 불순한 변경을 찾는 단서로 사용한다.

### DOM·접근성·성능의 확인 지점

React DevTools에서 props와 state의 소유·변화를 확인하고, Elements에서 실제 노드를 본다. Button이라는 이름만으로 키보드 동작이 생기지 않으며 실제 button을 반환해야 한다. 리렌더링이 있다는 이유만으로 최적화를 추가하지 않고 사용자의 입력 지연과 계산·DOM 반영 비용을 확인한다.

---

## 19. 흔한 오해와 실무 검증

렌더링 중에는 props·state·DOM 같은 외부 상태를 임의로 변경하거나 네트워크 요청을 시작하지 않는다. 개발 모드의 Strict Mode는 불순한 코드를 찾기 위해 일부 함수를 추가 호출할 수 있으므로 함수가 한 번만 호출될 것이라는 가정으로 코드를 작성하지 않는다.

React DevTools에서 어떤 컴포넌트가 어떤 props·state를 갖는지 확인하고, 브라우저 Elements에서 실제 생성된 DOM을 확인한다. 컴포넌트 이름이 Button이라고 접근 가능한 버튼이 되는 것은 아니다. 예제처럼 실제 button을 반환하고 이름을 제공해야 한다.

불필요한 상태를 줄이고 소유 위치를 명확히 하는 것이 먼저다. 리렌더링 자체를 모두 성능 문제로 간주하여 메모이제이션을 추가하기보다, 후속 성능 학습에서 실제 비용을 측정하여 판단한다.

---

## 20. 핵심 정리

<callout icon="✅" color="green_bg">
컴포넌트는 현재 입력의 UI를 설명하고 props는 외부 입력, state는 소유한 기억이다. setter는 갱신을 요청하며 다음 렌더링과 DOM 반영으로 이어진다. 상태를 직접 바꾸지 않고 계산 가능한 값은 렌더링 중 구한다.
</callout>

---

## 출처

- [React — Your First Component](https://react.dev/learn/your-first-component)
- [React — Passing Props](https://react.dev/learn/passing-props-to-a-component)
- [React — State: A Component's Memory](https://react.dev/learn/state-a-components-memory)
- [React — State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)
- [React — Queueing State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)
- [React — Render and Commit](https://react.dev/learn/render-and-commit)
- [React — Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)
- [React — Updating Objects in State](https://react.dev/learn/updating-objects-in-state)
- [React — Keeping Components Pure](https://react.dev/learn/keeping-components-pure)

### 이번 설명의 근거와 적용 범위

- [React — useState](https://react.dev/reference/react/useState): 초기화·setter 반환·함수형 갱신·동일 값·개발 검사.
- [React — Passing Props](https://react.dev/learn/passing-props-to-a-component): 기본값·children·읽기 전용 입력.
- [React — State as a Snapshot](https://react.dev/learn/state-as-a-snapshot): 현재 렌더링과 핸들러의 값.
- [React — Queueing Updates](https://react.dev/learn/queueing-a-series-of-state-updates): 값 대체와 갱신 함수의 순서.
- [React — Updating Objects](https://react.dev/learn/updating-objects-in-state): 중첩 참조와 불변 갱신.
- [React — Choosing State Structure](https://react.dev/learn/choosing-the-state-structure): 중복·모순 상태를 줄이는 기준.

---

## 21. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 같은 StudyCounter를 두 번 렌더링했는데 한쪽만 증가한다. 함수 정의와 상태의 관계로 설명하라.
2. props가 읽기 전용인데 부모의 값 변경이 자식 화면에 반영될 수 있는 이유는 무엇인가?
3. setCount를 호출한 직후 현재 핸들러의 count를 읽으면 왜 이전 값일 수 있는가?
4. count와 target이 있는데 remaining까지 state로 저장하면 어떤 동기화 문제가 생길 수 있는가?
