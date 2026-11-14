---
track: B
notion: https://app.notion.com/p/415178c4afac819f9b0ae33555d07cf0
---

# List / Key


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · React Basics 복습 · List / Key

배열에서 UI를 만드는 과정과 다음 렌더링에서 같은 항목을 이어가는 정체성을 분리하여 이해한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- filter·map·return으로 목록 데이터를 올바른 JSX 구조로 변환할 수 있다.
- 형제 범위·타입·key와 항목의 데이터 정체성을 설명할 수 있다.
- 재정렬·삭제·필터·제목 변경에서 상태 보존·초기화를 예측할 수 있다.
- 가장 바깥 요소·Fragment·별도 id prop의 사용 기준을 설명할 수 있다.
- 포커스와 행별 state를 만든 뒤 실제 목록 변경으로 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

목록은 단순히 같은 모양을 여러 번 그리는 작업이 아니다. 다음 렌더링에서 순서가 바뀌어도 React가 이전의 어떤 항목과 이어지는지 알아야 한다. key는 이 대응을 위한 정보다. 잘못 고르면 경고가 없어도 다른 행에 상태가 붙는 문제를 만들 수 있다.

---

## 3. 관련 계층과 역할 구분

JavaScript 배열 연산은 표시할 데이터와 JSX 배열을 만든다. React는 형제 사이의 key·타입 등으로 UI의 대응을 관리한다. React DOM이 실제 노드를 반영하고 HTML 목록·표 규칙은 최종 구조의 유효성과 접근성을 결정한다.

key는 React의 특별한 대응 정보이며 서버 ID·HTML id·일반 prop과 역할이 다르다. 데이터 영속 저장이나 모든 렌더링 성능 최적화를 대신하지 않는다.

---

## 4. 배열에서 UI와 정체성을 함께 만든다

`filter`는 표시할 데이터를 고르고 `map`은 각 데이터를 JSX로 바꾼다. 이 함수들은 JavaScript 배열 기능이며 React 전용 반복 문법이 아니다. React의 key는 형제 목록 안에서 항목을 구별하는 식별자다. 같은 형제 안에서 고유하고 렌더링 사이에 안정적이어야 한다. 문서 전체의 HTML id와 같은 요구는 아니다.

key는 일반 prop으로 전달되지 않는다. 자식이 식별자도 필요하면 `id={item.id}`처럼 따로 전달한다. 배열이 직접 반환하는 가장 바깥 JSX 요소에 key를 둬야 한다. [React 목록 안내](https://react.dev/learn/rendering-lists)가 이 규칙을 설명한다.

---

## 5. 배열의 변환과 목록의 구조

### filter와 map은 서로 다른 단계다

```jsx
const topics = [
  { id: 'html', title: 'HTML', enabled: true },
  { id: 'dom', title: 'DOM', enabled: false },
];

const rows = topics
  .filter((topic) => topic.enabled)
  .map((topic) => <li key={topic.id}>{topic.title}</li>);
```

React 컴포넌트에서 `{rows}`를 ul 안에 넣어 표시할 수 있다. filter는 데이터를 선택하고 map은 그 항목을 UI로 변환한다. map 안에서 중괄호 함수 본문을 쓰면 return을 명시해야 한다. 소괄호로 JSX를 반환한 예제와 중괄호만 작성한 함수를 혼동하지 않는다.

```jsx
const rows = topics.map((topic) => {
  return <li key={topic.id}>{topic.title}</li>;
});
```

return이 없으면 undefined의 배열을 만들 수 있어 UI가 보이지 않는다. 이 문제는 key의 우선순위 문제가 아니라 JavaScript 함수의 반환 문제다. 먼저 실제 map 결과가 무엇인지 확인한다.

### 데이터 구조와 HTML 콘텐츠 모델을 함께 지킨다

컴포넌트로 나누어도 ul 바로 아래에 임의의 div가 와도 된다는 규칙으로 바뀌지 않는다. Row가 실제로 li를 반환하는지, table의 행이 올바른 부모 아래에 있는지 확인한다. key는 React의 대응 정보이고 HTML 유효성을 검증해 주는 속성은 아니다.

---

## 6. 형제 범위의 정체성을 어떻게 이어갈까

### 같은 데이터와 같은 위치는 다르다

항목을 뒤집거나 앞에 삽입하면 위치는 바뀌어도 데이터의 id는 유지될 수 있다. 안정적인 key는 이런 요구를 표현한다. 인덱스 key는 위치를 정체성으로 삼으므로 “0번 위치가 계속 같은 항목”이라는 가정이 들어간다.

### 같은 key만으로 모든 상태가 영원히 유지되지는 않는다

형제 범위, 컴포넌트 타입, 부모의 보존 여부도 관계한다. key는 같아도 다른 컴포넌트 타입으로 바꾸거나 부모 전체를 새로 만들면 하위 state가 초기화될 수 있다. 렌더 트리에서 제거된 컴포넌트의 지역 state를 key가 별도 저장소에 보관해 주는 것은 아니다.

### 다른 목록에서는 같은 key를 사용할 수 있다

key는 형제 사이의 구별을 위해 사용한다. 서로 다른 ul 아래에서 같은 데이터 id를 사용한다고 문서 전체 key 충돌이 되는 것은 아니다. HTML id는 같은 문서 안의 고유성·레이블·주소 연결 등 다른 역할이므로 두 식별자를 같은 목적으로 사용하지 않는다.

---

## 7. 예제: 순서를 바꿔도 행의 상태가 따라가게 한다

기존 React 프로젝트의 `App.jsx`에서 실행한다.

```jsx
import { useState } from 'react';

const initialTopics = [
  { id: 'html', title: 'HTML' },
  { id: 'dom', title: 'DOM' },
  { id: 'events', title: 'Events' },
];

function TopicRow({ topic, onRemove }) {
  const [practiceCount, setPracticeCount] = useState(0);
  return (
    <li>
      <span>{topic.title}: 실습 {practiceCount}회 </span>
      <button type="button" onClick={() => setPracticeCount((count) => count + 1)}>
        {topic.title} 실습 추가
      </button>
      <button type="button" onClick={() => onRemove(topic.id)}>
        {topic.title} 삭제
      </button>
    </li>
  );
}

export default function App() {
  const [topics, setTopics] = useState(initialTopics);

  function removeTopic(id) {
    setTopics((previous) => previous.filter((topic) => topic.id !== id));
  }

  return (
    <main>
      <h1>학습 순서</h1>
      <button type="button" onClick={() => setTopics((previous) => [...previous].reverse())}>
        순서 뒤집기
      </button>
      <ul>
        {topics.map((topic) => (
          <TopicRow key={topic.id} topic={topic} onRemove={removeTopic} />
        ))}
      </ul>
      {topics.length === 0 && <p>학습 항목이 없습니다.</p>}
    </main>
  );
}
```

HTML 실습 횟수를 올린 뒤 순서를 뒤집으면 HTML 행이 이동해도 그 실습 횟수는 HTML에 남는다. key와 컴포넌트 타입으로 같은 형제 항목임을 이어갈 수 있기 때문이다. 원본 state 배열에 reverse를 직접 적용하지 않고 복사본을 뒤집었다.

`map((topic, index) => <TopicRow key={index} ... />)`로 바꾸고 같은 실험을 하면 React는 0번 위치의 컴포넌트를 계속 0번 위치로 대응시킬 수 있다. 그 위치의 props는 다른 주제가 되었는데 지역 state는 남아 실습 횟수가 다른 주제에 붙는 현상을 볼 수 있다. key는 경고를 지우기 위한 장식이 아니다.

---

## 8. key가 보존과 초기화에 관여한다

상태는 객체 데이터 안에 자동으로 저장되는 것이 아니라 React가 관리하는 렌더 트리의 위치·타입·key 관계에 연결된다. 동일 형제 범위에서 안정적인 key는 재정렬 뒤에도 대응을 유지하는 데 쓰이고, key가 바뀌면 다른 인스턴스로 취급되어 상태가 초기화될 수 있다. 목록에서 제거한 항목을 나중에 같은 key로 다시 넣었다고 제거 전 지역 state가 영구 복원되는 것은 아니다. [상태 보존과 초기화](https://react.dev/learn/preserving-and-resetting-state)를 함께 참고한다.

컴포넌트 내부의 li에만 key를 넣으면 부모 배열의 TopicRow 정체성을 제공하지 못한다. 여러 노드를 한 항목으로 반환해야 한다면 key를 지정할 수 있는 명시적 Fragment 등을 사용한다. 짧은 `<>...</>` 문법에는 key를 직접 붙일 수 없다.

---

## 9. 여러 요소를 한 항목으로 반환하기

```jsx
import { Fragment } from 'react';

function TopicDefinitions({ topics }) {
  return (
    <dl>
      {topics.map((topic) => (
        <Fragment key={topic.id}>
          <dt>{topic.title}</dt>
          <dd>{topic.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

한 데이터가 dt·dd 두 요소를 만들지만 Fragment는 추가 DOM 래퍼를 만들지 않는다. 배열이 직접 반환하는 가장 바깥 항목에 key가 있으므로 두 요소를 하나의 대응 단위로 표현한다. 짧은 Fragment 문법에는 key를 지정할 수 없어 명시적 Fragment를 사용했다.

자식 내부에 key를 숨기는 대신 부모의 map에서 대응 단위를 정한다. 자식이 id를 알아야 한다면 별도 prop을 추가한다. React의 특별한 key 정보는 일반 props.key로 읽는 계약이 아니다.

---

## 10. 어떤 값을 key로 고를까

데이터베이스 ID나 데이터 생성 시 정한 안정적인 ID를 사용한다. 렌더링할 때마다 Math.random이나 새로운 UUID를 만들면 매번 다른 항목으로 인식되어 상태·포커스를 잃고 불필요한 재생성이 생길 수 있다. ID가 필요하면 데이터 생성 시 만들고 이후 유지한다.

인덱스는 목록이 고정되고 재정렬·삽입·삭제 등으로 항목과 위치의 대응이 바뀌지 않는 제한된 상황에서 검토할 수 있다. 앞으로 변할 수 있는 목록에 기본 선택으로 쓰지 않는다. 제목도 중복되거나 변경될 수 있으므로 정체성을 보장하는지 확인한다.

---

## 11. 안정적인 ID를 데이터 생성 시점에 정하기

### 생성 시점과 렌더링 시점

새 데이터의 id를 만들려면 생성 사건에서 한 번 정하고 이후 데이터에 보관한다. 렌더링할 때마다 id를 다시 만들면 같은 데이터도 매번 다른 정체성으로 표시된다. 서버 ID가 있다면 보통 그것을 사용하고, 로컬 초안은 생성 시 ID와 서버 저장 후 ID의 전환이 상태 보존에 어떤 영향을 주는지도 고려한다.

중복된 title을 key로 쓰면 서로 다른 항목을 구별하지 못할 수 있다. 제목 수정 자체로 key가 바뀌면 편집 중 상태가 초기화될 수도 있다. 표시값과 정체성이 같은 수명인지 확인한다.

### 상태를 행에 둘지 목록에 둘지

행의 임시 펼침 상태는 행에 둘 수 있지만 필터로 행을 잠시 제거해도 편집값을 보존해야 한다면 목록 또는 별도 데이터 소유자가 id별로 저장해야 할 수 있다. 안정적인 key는 현재 남아 있는 항목을 이어주는 도구이며 영구 데이터 보존 정책을 대신하지 않는다.

### 재정렬 시험은 상태를 먼저 만든 뒤 한다

빈 행만 뒤집으면 잘못된 key가 겉으로 드러나지 않을 수 있다. 행마다 다른 카운터·펼침·입력 상태를 만든 뒤 앞 삽입, 중간 삭제, 정렬, 필터 해제, 제목 변경을 시험한다. 어떤 상태가 어느 데이터에 남아야 하는지 먼저 예상하고 DevTools와 실제 조작으로 확인한다.

큰 목록이 느릴 때는 안정적인 key 여부와 별개로 렌더링 수·각 행의 계산·DOM 양을 측정한다. 목록 가상화·메모이제이션 등 세부 최적화는 후속 React Performance에서 다룬다.

---

## 12. 목록 처리에서 데이터 변환과 UI 대응을 분리하기

### 목록을 그리는 일에는 서로 다른 두 문제가 있다

첫째는 이번 화면에 어떤 데이터를 어떤 순서로 표시할지 계산하는 문제다. `filter`·`map`·정렬 결과가 여기에 해당한다. 둘째는 이전 화면에 있던 행과 이번 화면의 행을 어떻게 이어갈지 결정하는 문제다. 이때 필요한 정보가 항목의 타입과 key다. 배열을 올바르게 만들었어도 두 번째 대응이 틀리면 화면의 제목은 맞는데 입력값이나 펼침 상태는 다른 항목에 남을 수 있다.

다음 배열을 생각한다. A·B·C는 데이터의 ID이고 괄호는 행 컴포넌트가 가진 지역 카운터다.

| 시점 | ID를 key로 사용 | 인덱스를 key로 사용 |
| --- | --- | --- |
| 변경 전 데이터 A, B, C | A(1), B(2), C(3) | 위치 0의 A(1), 위치 1의 B(2), 위치 2의 C(3) |
| C, B, A로 뒤집음 | C(3), B(2), A(1) | 위치 0의 C(1), 위치 1의 B(2), 위치 2의 A(3) |

인덱스 방식에서도 React는 새 props의 제목을 표시한다. 오류는 텍스트가 갱신되지 않는 데 있는 것이 아니라, 위치에 남은 state가 다른 데이터와 결합하는 데 있다. [React의 key 설명](https://react.dev/learn/rendering-lists#why-does-react-need-keys)은 삽입·삭제·정렬에서 안정적인 식별자가 필요한 이유를 이 대응 문제로 설명한다.

### key가 없다는 경고와 잘못된 key는 다른 문제다

중복되지 않는 인덱스 key를 주면 누락 경고는 없어질 수 있다. 그러나 경고가 없다는 사실은 데이터의 수명과 key의 수명이 일치한다는 증거가 아니다. 반대로 모든 행이 state 없이 같은 모양의 텍스트만 표시한다면 잘못된 대응이 눈에 띄지 않을 수도 있다. 시험 전에 행마다 다른 상태를 만들어야 하는 이유다.

### 목록 변환이 원본 state를 바꾸지 않도록 한다

`reverse()`·`sort()`는 호출한 배열을 변경한다. state 배열을 직접 뒤집고 같은 참조를 setter에 넘기는 방식은 피한다. `[...previous].reverse()`처럼 새 배열을 만든다. 복사는 목록 컨테이너를 새로 만드는 것이며 기존 항목의 `id`까지 바꾸는 작업은 아니다. 항목의 속성을 수정할 때는 해당 객체도 새 객체로 교체한다. [배열 state 갱신 안내](https://react.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array)를 참고한다.

---

## 13. 실험: ID와 위치가 서로 다른 결과를 만드는 순간

### 실행 조건과 관찰 목표

기존 React 프로젝트의 `App.jsx`를 아래 코드로 교체한다. 먼저 `keyMode = 'id'`로 실행하고, 실험을 마친 뒤 `'index'`로 바꿔 새로고침한다. 실행 중 모드를 바꾸면 key 변경 자체에 따른 초기화가 섞이므로 두 실행을 분리한다. 로컬의 `http://localhost` 또는 HTTPS에서 `crypto.randomUUID()`를 지원하는 브라우저를 사용한다.

```jsx
import { useState } from 'react';

const keyMode = 'id'; // 대비 실행에서는 'index'로 바꾸고 새로고침한다.
const initialTopics = [
  { id: 'html', title: 'HTML' },
  { id: 'dom', title: 'DOM' },
  { id: 'events', title: 'Events' },
];

function TopicRow({ topic, onRemove }) {
  const [count, setCount] = useState(0);
  return (
    <li>
      <span>{topic.title}: {count}회 </span>
      <button type="button" onClick={() => setCount((n) => n + 1)}>
        {topic.title} 횟수 추가
      </button>
      <button type="button" onClick={() => onRemove(topic.id)}>
        {topic.title} 삭제
      </button>
    </li>
  );
}

export default function App() {
  const [topics, setTopics] = useState(initialTopics);

  function prependTopic() {
    const topic = { id: crypto.randomUUID(), title: '새 주제' };
    setTopics((previous) => [topic, ...previous]);
  }

  function removeTopic(id) {
    setTopics((previous) => previous.filter((topic) => topic.id !== id));
  }

  return (
    <main>
      <h1>목록 대응 실험: {keyMode}</h1>
      <button type="button" onClick={prependTopic}>앞에 추가</button>
      <button type="button" onClick={() => {
        setTopics((previous) => [...previous].reverse());
      }}>순서 뒤집기</button>
      <ul>
        {topics.map((topic, index) => (
          <TopicRow
            key={keyMode === 'id' ? topic.id : index}
            topic={topic}
            onRemove={removeTopic}
          />
        ))}
      </ul>
    </main>
  );
}
```

### 뒤집기: props 변경과 state 보존을 함께 읽는다

HTML은 1회, DOM은 2회, Events는 3회로 만든 뒤 순서를 뒤집는다. ID 방식은 Events 3회 → DOM 2회 → HTML 1회다. 인덱스 방식은 Events 1회 → DOM 2회 → HTML 3회다. 새 배열의 첫 데이터가 Events가 되어 첫 행의 props는 바뀌지만, 인덱스 0에 대응한 컴포넌트의 state는 1이기 때문이다. 가운데 행이 우연히 맞다고 전체 대응이 맞는 것은 아니다.

### 앞 삽입: 새 데이터가 기존 state를 받는 오류

새로고침하여 기존 행의 카운터를 구분한 뒤 ‘앞에 추가’를 누른다. ID 방식은 새 행만 0이고 기존 세 행은 자기 카운터를 유지한다. 인덱스 방식에서는 새 행이 기존 위치 0의 카운터를 받고 기존 데이터들이 뒤쪽 위치의 state와 결합한다. 맨 끝의 새 위치에 배치된 행이 0으로 시작할 수 있다. 삽입한 데이터가 새로 생겼다는 사실과 React가 어느 항목을 새 인스턴스로 판단했는지가 어긋난 것이다.

### 삭제: 없어진 데이터의 상태를 다음 행이 물려받는다

다시 초기 상태에서 서로 다른 횟수를 만든 뒤 DOM을 삭제한다. 인덱스 key에서는 Events가 위치 1로 이동하여 DOM이 쓰던 카운터와 결합한다. ID 방식에서는 DOM의 행과 지역 state가 제거되고 HTML·Events의 대응은 유지된다. DevTools에서 행의 props와 state를 동시에 보면 이 차이를 확인하기 쉽다.

### ID는 생성 사건에서 만들고 갱신 함수는 순수하게 둔다

새 UUID는 클릭 핸들러에서 만들고 setter의 갱신 함수는 받은 배열로 새 배열만 계산한다. 개발 모드의 Strict Mode에서는 순수성을 점검하기 위해 갱신 함수를 추가 호출할 수 있으므로 그 안에서 외부 카운터 증가 같은 부수 효과를 만들지 않는다. 데이터 생성 이후의 렌더링에서는 저장된 ID를 계속 읽는다. [React useState의 갱신 함수 규칙](https://react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice)을 함께 적용한 코드다.

---

## 14. key가 같아도 상태가 사라지는 경계

### 타입이 바뀌면 같은 key만으로 이어지지 않는다

다음 코드는 독립된 `App.jsx` 예제다. 먼저 횟수를 올린 뒤 화면 타입을 바꿔 본다.

```jsx
import { useState } from 'react';

function PracticeCounter() {
  const [count, setCount] = useState(0);
  return <button type="button" onClick={() => setCount((n) => n + 1)}>
    실습 {count}회
  </button>;
}

function Summary() {
  return <p>요약 화면</p>;
}

export default function App() {
  const [summary, setSummary] = useState(false);
  return (
    <main>
      <h1>타입과 상태</h1>
      <button type="button" onClick={() => setSummary((value) => !value)}>
        화면 타입 전환
      </button>
      {summary ? <Summary key="topic" /> : <PracticeCounter key="topic" />}
    </main>
  );
}
```

key는 계속 `topic`이지만 `PracticeCounter`와 `Summary`는 서로 다른 컴포넌트 타입이다. 요약으로 바꾸면 카운터 컴포넌트가 제거되며, 돌아오면 새 카운터가 0에서 시작한다. 이 결과는 ‘같은 ID의 데이터를 표시한다’와 ‘같은 컴포넌트가 계속 존재한다’가 서로 다른 조건임을 보여 준다.

### 제거했다가 복원하는 것과 숨기는 것은 다르다

`shown && <PracticeCounter key="topic" />`에서 shown을 false로 바꾸면 해당 컴포넌트는 트리에서 제거된다. 나중에 true가 되어도 이전 지역 state를 key가 복원해 주지 않는다. 반면 계속 렌더링한 `<section hidden={!shown}>` 내부의 카운터는 같은 트리 위치와 타입을 유지할 수 있다. 이때 `hidden`은 브라우저에 표시하지 않는 조건이며 React 컴포넌트를 제거하는 조건이 아니다.

필터를 잠시 적용해 행을 제거해도 편집 중 데이터가 남아야 한다면, 보존할 초안을 부모의 데이터에 `id`별로 보관하는 설계를 검토한다. 모든 행을 무조건 숨겨 두면 DOM 양도 유지되므로 그 비용과 실제 보존 요구를 함께 판단한다.

### key의 범위는 부모 안의 형제 관계다

오른쪽 목록과 왼쪽 목록에 같은 key를 가진 행이 있어도 두 행은 각 부모 아래의 별도 상태를 가질 수 있다. 한 목록에서 제거하고 다른 부모 아래에 넣는 것을 목록 내부 재정렬과 같은 상태 보존으로 기대하지 않는다. 부모 자체의 key를 바꿔 새로 만들면 그 아래 상태도 다시 시작할 수 있다. 이 경계는 [상태 보존과 초기화의 렌더 트리 설명](https://react.dev/learn/preserving-and-resetting-state#state-is-tied-to-a-position-in-the-render-tree)을 기준으로 판단한다.

---

## 15. key를 두는 위치와 DOM의 식별자를 확인하기

### 부모 배열의 대응 단위에 둔다

부모가 `topics.map(topic => <TopicRow topic={topic} />)`를 반환하면서 TopicRow 내부의 li에만 key를 두면, 부모 배열의 항목에는 여전히 key가 없다. 내부 li는 각 TopicRow의 반환 결과이며 부모 배열에서 구별할 대상은 TopicRow다. 따라서 `<TopicRow key={topic.id} ... />`에 둔다.

일반 prop과의 차이도 확인한다. `<TopicRow key={topic.id} topicId={topic.id} />`에서 자식은 `topicId`를 읽는다. `key`는 React가 소비하는 특별한 정보이므로 `props.key`로 업무 식별자를 받는 API를 설계하지 않는다. 같은 값을 두 자리에 주더라도 각각 React의 대응과 자식의 데이터라는 다른 책임을 맡는다.

### Fragment는 추가 DOM 없이 여러 형제를 묶는다

다음도 독립 실행할 수 있는 `App.jsx`다.

```jsx
import { Fragment } from 'react';

const topics = [
  { id: 'html', title: 'HTML', description: '문서의 의미 구조' },
  { id: 'dom', title: 'DOM', description: '문서를 다루는 객체 모델' },
];

export default function App() {
  return (
    <main>
      <h1>학습 용어</h1>
      <dl>
        {topics.map((topic) => (
          <Fragment key={topic.id}>
            <dt>{topic.title}</dt>
            <dd>{topic.description}</dd>
          </Fragment>
        ))}
      </dl>
    </main>
  );
}
```

Elements 패널에서는 dl 아래에 dt·dd·dt·dd가 있고 Fragment라는 HTML 요소는 없다. 한 데이터가 만든 두 형제의 React 대응 단위를 Fragment로 표현한 것이다. key를 적어야 하므로 짧은 `<>...</>` 대신 import한 Fragment를 사용한다. [React Fragment 문서](https://react.dev/reference/react/Fragment#rendering-a-list-of-fragments)가 이 사용 범위를 설명한다.

### key는 HTML id나 접근 가능한 이름을 만들지 않는다

Fragment와 TopicRow의 key는 DOM 속성이 아니다. 레이블과 입력을 연결할 때는 `htmlFor`와 HTML `id`를 별도로 제공해야 한다. 데이터 ID가 같은 항목을 한 문서에 두 번 표시한다면 key는 각 목록에서 유효하더라도 HTML id는 문서 전체에서 중복되지 않도록 접두어 등을 구분해야 한다. key가 올바르다는 사실은 유효한 HTML 구조와 레이블 연결까지 검사했다는 뜻이 아니다.

---

## 16. 목록 문제를 재현하고 수정하는 검사 흐름

### 증상부터 대응 관계를 기록한다

‘삭제 후 다른 행의 입력값이 바뀐다’면 먼저 항목마다 구별되는 값을 만든다. 변경 전 데이터 ID·화면 순서·각 행의 state를 적고 한 항목만 삭제한다. React DevTools에서 남은 행의 props ID와 state가 어느 이전 행의 것인지 비교한다. key 누락 경고만 찾는 것으로 검사를 끝내지 않는다.

인덱스나 변경 가능한 제목이 key라면, 보존하려는 데이터의 안정적인 ID로 바꾼다. 새로고침하고 동일한 조작을 다시 수행하여 정상 동작을 확인한다. 개발 도중 key 표현식을 바꾸는 순간 생긴 초기화는 수정 후 데이터 변경 동작과 분리해서 해석한다.

### 포커스와 삭제 정책도 별도로 확인한다

키보드로 행의 버튼에 포커스를 둔 뒤 재정렬한다. DOM이 보존·이동하는지와 브라우저의 실제 포커스가 어디에 있는지 확인한다. 안정적인 key는 대응에 필요한 조건이지만 모든 브라우저의 모든 DOM 이동에서 포커스 보존을 보장하는 접근성 API는 아니다. 삭제한 컨트롤은 더 이상 포커스를 유지할 수 없으므로 다음 행·목록 제목 등 적절한 위치로 이어 주는 정책을 제품 UI에 설계한다.

### 올바른 key와 빠른 목록은 같은 판정이 아니다

key는 이전 항목과의 대응을 제공한다. 부모가 다시 계산될 때 행 함수의 실행을 모두 생략한다는 계약은 아니다. 큰 목록의 지연은 항목 수, 행별 계산, DOM 반영을 나눠 관찰한다. 먼저 정확성을 고친 뒤 React DevTools와 브라우저 Performance 기록으로 병목을 찾고 같은 데이터·조작 조건에서 개선 전후를 비교한다. 실측 없이 ‘key를 붙였으니 성능이 해결됐다’고 판단하지 않는다.

---

## 17. 접근성·성능·디버깅

정렬·삽입 후 값만 확인하지 말고 각 행의 상태와 포커스도 확인한다. key는 HTML의 목록 의미나 컨트롤 이름을 대신하지 않으므로 ul·li와 구별 가능한 버튼 이름을 제공한다. 포커스된 삭제 버튼의 행이 사라질 때는 다음 작업 위치를 설계해야 한다. 예제는 key와 상태의 실험이며 삭제 후 포커스 이동까지 완성한 제품 UI는 아니다.

React 경고, DevTools의 행별 state, 재정렬 후 DOM과 포커스를 함께 검사한다. 안정적인 key가 불필요한 재생성을 줄일 수는 있지만 key 자체가 모든 리렌더링을 없애거나 큰 목록의 모든 성능 문제를 해결하지는 않는다.

---

## 18. 핵심 정리

<callout icon="✅" color="green_bg">
map은 데이터로 UI를 만들고 key는 형제들 사이에서 그 UI의 정체성을 연결한다. 안정적인 데이터 식별자를 사용하고, 재정렬·삭제·초기화 때 상태가 어떻게 이어지는지 확인한다.
</callout>

---

## 출처

- [React — Rendering Lists](https://react.dev/learn/rendering-lists)
- [React — Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state)
- [React — Fragment](https://react.dev/reference/react/Fragment)

### 이번 설명의 근거와 적용 범위

- [React — Rendering Lists](https://react.dev/learn/rendering-lists#rules-of-keys): 형제 범위의 고유성, 렌더링 사이의 안정성, 인덱스·무작위 key의 문제, key와 일반 prop의 구분을 확인했다.
- [React — Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state): 트리 위치·타입·key에 따른 상태의 보존과 제거, 부모의 변화가 하위 상태에 미치는 범위를 확인했다.
- [React — Fragment](https://react.dev/reference/react/Fragment#rendering-a-list-of-fragments): 명시적인 Fragment의 key와 추가 DOM 래퍼 없이 반환되는 형제 구조를 확인했다.
- [React — Updating Arrays in State](https://react.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array): reverse·sort 전에 배열을 복사하고 항목 수정 시 필요한 객체를 새로 만드는 규칙을 확인했다.
- [React — useState의 개발 모드 점검](https://react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice): 초기화 함수·갱신 함수의 순수성 요구를 확인했다. 예제의 UUID 생성은 이벤트 처리 단계에 둔다.

---

## 19. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 인덱스 key 목록의 앞에 항목을 삽입하면 기존 행의 state가 다른 데이터에 붙을 수 있는 이유를 설명하라.
2. 자식 컴포넌트 안의 li에만 key를 넣어도 부모 map의 경고가 남는 이유는 무엇인가?
3. 렌더링마다 새 UUID를 key로 만들면 포커스와 지역 state에 어떤 일이 생길 수 있는가?
4. 삭제한 항목을 같은 key로 나중에 다시 넣으면 이전 지역 state가 반드시 돌아오는가? 보존 조건을 설명하라.
