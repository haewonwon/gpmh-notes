---
track: B
notion: https://app.notion.com/p/415178c4afac81739b03d42061d6ef36
---

# Controlled / Uncontrolled Component


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · React Basics 복습 · Controlled / Uncontrolled Component

초기값과 현재 값, DOM과 React의 소유권, 실시간 동기화와 제출 시 읽기를 구별하여 입력 흐름을 설계한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 제어·비제어 입력의 현재 값 기준과 변경 경로를 설명할 수 있다.
- 텍스트·체크·select·textarea·파일에서 적절한 값 API를 고를 수 있다.
- 비동기 데이터·초안·reset·대상 변경의 초기화 정책을 설명할 수 있다.
- 모드 전환·읽기 전용·숫자 해석·조합 입력의 문제를 분석할 수 있다.
- state·DOM 프로퍼티·FormData·포커스·입력 비용을 함께 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

사용자가 입력한 값을 DOM이 관리하게 둘 수도 있고 React state가 관리하도록 연결할 수도 있다. 화면이 비슷해도 값의 기준이 다르므로 변경·초기화·검증·성능의 구현 방식이 달라진다. 먼저 “현재 값의 기준은 어디에 있는가”를 정해야 두 관리 방식이 충돌하지 않는다.

---

## 3. 관련 계층과 역할 구분

브라우저의 input·select·textarea는 편집 가능한 DOM 상태와 기본 검증·제출 모델을 제공한다. React는 value·checked를 전달해 현재 값을 제어하거나 defaultValue·defaultChecked로 초기 기준만 제공할 수 있다. 애플리케이션은 서버 데이터·초안·검증·초기화 정책을 결정한다.

부모의 제어 prop과 자식 내부 상태의 관계는 일반적인 컴포넌트 설계이고, DOM 폼 제어는 그 구체적인 사례다. 이 페이지는 브라우저의 함수 컴포넌트 예제를 기준으로 한다.

---

## 4. 제어의 의미와 데이터 흐름

일반적으로 제어 컴포넌트는 중요한 정보를 부모의 props로 받고 변경을 콜백으로 알리는 방식이며, 비제어 컴포넌트는 그 정보를 내부에서 소유하는 방식이다. 이 페이지에서는 특히 DOM 폼 입력에서 React가 현재 값을 지정하는지에 초점을 맞춘다.

| 방식 | 현재 값의 기준 | React 입력 API | 읽는 시점 |
| --- | --- | --- | --- |
| 제어 입력 | React state 등 전달된 값 | `value` / `checked`와 변경 처리 | 렌더링·이벤트마다 |
| 비제어 입력 | DOM 입력 요소 | `defaultValue` / `defaultChecked`로 초기값 | 필요할 때 FormData·DOM 참조 등 |

“비제어”는 아무도 값을 관리하지 않는다는 뜻이 아니다. 브라우저가 DOM의 현재 값을 관리한다. “제어”도 항상 전역 상태를 쓴다는 뜻은 아니며 컴포넌트의 지역 state로 충분할 수 있다. [React input 문서](https://react.dev/reference/react-dom/components/input)가 두 방식을 비교한다.

---

## 5. 현재 값의 소유자를 기준으로 두 흐름을 그리기

### 제어 입력의 왕복

사용자 편집 → DOM 이벤트 → 현재 값 읽기 → state 갱신 요청 → 새 value·checked 전달의 흐름이다. React가 받은 값이 다음 표시 기준이므로 편집 이벤트에서 해당 state를 갱신하지 않으면 사용자가 입력한 값이 유지되지 않을 수 있다.

### 비제어 입력의 읽기

사용자 편집 → 브라우저가 DOM 현재 값 유지 → 필요한 시점에 FormData 등으로 읽기의 흐름이다. 입력마다 React state로 복사할 필요는 없지만 제출·검증·초기화에서 현재 DOM 값을 어떻게 얻을지 정해야 한다. 비제어 입력에 이벤트 핸들러를 추가한다고 자동으로 제어 입력이 되는 것은 아니다.

### 중요한 것은 단일한 기준이다

같은 값을 DOM과 React state가 각각 독립적으로 수정하고 서로 다른 시점에 맞추려 하면 충돌한다. 제어 모드를 정하고 데이터 변경 경로를 한쪽으로 모은다. 다른 필드나 제출 결과까지 모두 같은 소유 방식을 써야 하는 것은 아니다.

---

## 6. 예제: 즉시 미리보기와 제출 시 읽기

기존 React 프로젝트의 `App.jsx`에서 실행한다.

```jsx
import { useState } from 'react';

function ControlledNote() {
  const [title, setTitle] = useState('HTML');
  const [shared, setShared] = useState(false);

  return (
    <section>
      <h2>제어 입력</h2>
      <label htmlFor="controlled-title">제목</label>
      <input
        id="controlled-title"
        value={title}
        onChange={(event) => setTitle(event.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={shared}
          onChange={(event) => setShared(event.target.checked)}
        />
        공개
      </label>
      <p>미리보기: {title} / {shared ? '공개' : '비공개'}</p>
      <button type="button" onClick={() => { setTitle(''); setShared(false); }}>
        내용 지우기
      </button>
    </section>
  );
}

function UncontrolledNote() {
  const [submitted, setSubmitted] = useState('아직 제출하지 않았습니다.');

  function handleSubmit(event) {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    setSubmitted(`${data.get('title')} / ${data.has('shared') ? '공개' : '비공개'}`);
  }

  return (
    <section>
      <h2>비제어 입력</h2>
      <form onSubmit={handleSubmit}>
        <label htmlFor="uncontrolled-title">제목</label>
        <input id="uncontrolled-title" name="title" defaultValue="HTML" />
        <label>
          <input type="checkbox" name="shared" value="yes" defaultChecked={false} />
          공개
        </label>
        <button type="submit">제출 값 보기</button>
        <button type="reset">입력 초기값으로</button>
      </form>
      <p role="status">{submitted}</p>
    </section>
  );
}

export default function App() {
  return (
    <main>
      <h1>입력값의 소유 위치</h1>
      <ControlledNote />
      <UncontrolledNote />
    </main>
  );
}
```

제어 입력은 사용자가 편집하면 onChange가 현재 DOM 값을 읽어 state 갱신을 요청하고, 다음 렌더링이 value·checked와 미리보기를 지정한다. React가 전달하는 값이 현재 표시 값의 기준이다. 체크박스는 문자열 value가 아니라 `checked`로 선택 여부를 제어한다.

비제어 입력은 편집하는 동안 브라우저가 현재 값을 유지한다. 제출 시 FormData가 현재 DOM 상태를 읽고, 결과 문구만 React state로 표시한다. 제출 결과 state가 있다고 입력까지 제어 입력이 되는 것은 아니다. 같은 컴포넌트 안에서도 데이터별로 소유 방식이 다를 수 있다.

---

## 7. 초기값과 현재 값, 초기화는 다르다

`defaultValue`는 입력의 초기·기본값을 지정하며 현재 값을 매 렌더링마다 강제로 동기화하는 value와 다르다. 뒤늦게 받은 서버 데이터를 defaultValue로 넘기는 것만으로 사용자가 편집 중인 현재 값을 덮어쓰려 하지 않는다. 데이터를 받아 초기화할 시점과 사용자의 수정을 보존할 정책을 먼저 정한다.

위 비제어 폼의 reset은 입력을 기본값으로 되돌리지만 제출 결과 state는 그대로 남는다. DOM의 폼 초기화가 React state까지 자동으로 초기화하는 것은 아니다. 제어 입력의 초기화는 state 소유자가 명시적으로 값을 바꾸는 방식으로 설계한다. 앞서 배운 key를 바꾸어 전체 하위 상태를 초기화할 수도 있지만 포커스와 다른 지역 state까지 새로 만들어질 수 있으므로 필요한 범위를 고려한다.

---

## 8. 텍스트·체크·선택·파일의 차이

| 컨트롤 | 제어 기준 | 비제어 초기 기준 | 현재 값 읽기 |
| --- | --- | --- | --- |
| 텍스트 input | value 문자열 | defaultValue | target.value·FormData |
| 체크박스 | checked 불리언 | defaultChecked | target.checked, 선택 시 FormData 항목 |
| 라디오 | 같은 name 그룹의 checked | defaultChecked | 선택된 값 |
| textarea | value | defaultValue | 현재 value·FormData |
| select | 선택값에 맞는 value | defaultValue | 현재 선택값·FormData |
| 파일 input | 일반 문자열 대입으로 사용자 선택 제어 불가 | 사용자 선택 중심 | files·FormData의 File |

체크박스의 value는 선택 여부가 아니라 선택되었을 때 제출할 값이다. `checked="false"`처럼 문자열을 불리언 의미 대신 전달하지 않고 실제 boolean을 사용한다. 다중 선택 select는 여러 값과 getAll 등 제출 데이터의 복수 항목 처리도 확인한다.

React의 textarea는 자식 텍스트를 현재 값의 제어 수단으로 사용하지 않고 value 또는 defaultValue를 사용한다. select도 개별 option의 selected를 수동 관리하기보다 select의 해당 prop 계약을 사용한다. DOM의 초기 HTML 작성 방식과 React의 제어 prop 사용법을 구별한다.

---

## 9. 비동기 초기값과 사용자의 편집을 함께 다루기

### 데이터 도착 전후에도 모드를 유지한다

처음 undefined였던 value가 나중에 문자열이 되면 비제어·제어 전환 문제가 생길 수 있다. 제어할 필드라면 처음부터 문자열의 빈 상태를 유지한다. checkbox는 boolean의 기준을 유지한다. “값이 아직 없음”과 “사용자가 빈 문자열을 입력함”을 업무 데이터에서 구별해야 할 수도 있다.

### 서버 값으로 덮어쓸 시점을 결정한다

초기 데이터를 받은 직후 필드를 채우는 요구와, 사용자가 이미 수정한 초안을 계속 보존하는 요구는 다르다. prop이 바뀔 때마다 무조건 모든 입력 state를 덮으면 사용 중인 편집을 잃을 수 있다. 새 대상을 선택했을 때 초기화할지, 저장 전 경고를 할지, 변경된 필드만 유지할지 제품의 정책을 정한다.

이 문제를 defaultValue와 value를 동시에 지정하는 것으로 해결하지 않는다. 입력 소유권과 서버 데이터 동기화는 각각 결정할 조건이다.

### 빈 값과 숫자 해석

숫자 입력의 value도 문자열이다. 빈 상태를 Number로 변환하면 0이 되는 등의 차이가 생길 수 있으므로 사용자가 편집 중인 문자열과 제출할 숫자의 의미를 구별한다. valueAsNumber를 사용할 때도 NaN·빈 값·범위 실패를 처리해야 한다.

실시간 입력을 매번 강제로 포맷하거나 대문자로 바꾸면 커서 위치와 조합 입력에 영향을 줄 수 있다. 한국어 등 IME 조합 중인 상태를 포함하여 실제 타이핑을 시험하고, 필요하면 변환·검증 시점을 제출·포커스 이탈 등으로 구분한다.

---

## 10. 흔한 오류를 현재 값의 기준으로 찾는다

- `value={title}`을 지정하고 변경 처리 없이 편집을 기대하면 React가 전달한 값과 사용자 입력이 충돌한다. 읽기 전용 의도면 readOnly를 명시하고, 편집 가능 의도면 onChange에서 해당 값을 갱신한다.
- 처음 value가 undefined였다가 문자열이 되면 비제어에서 제어로 바뀌는 문제가 생길 수 있다. 텍스트 제어 값은 처음부터 `''` 같은 문자열로 유지하고, 비동기 데이터도 `value={data ?? ''}`처럼 일관되게 다룬다.
- value와 defaultValue를 같은 입력에 함께 주어 두 기준을 섞지 않는다. 체크박스도 checked와 defaultChecked를 구별한다.
- 숫자 타입 입력도 `event.target.value`는 문자열이다. 빈 입력·잘못된 값과 숫자로 변환할 시점을 별도로 설계한다.
- 파일 입력은 사용자의 파일 선택을 일반 텍스트 value처럼 임의로 지정하는 방식으로 제어하지 않는다. 선택된 FileList 등 해당 API를 사용한다.

---

## 11. 커스텀 제어 컴포넌트의 계약

```jsx
function TopicInput({ id, value, onValueChange }) {
  return (
    <>
      <label htmlFor={id}>학습 제목</label>
      <input
        id={id}
        value={value}
        onChange={(event) => onValueChange(event.target.value)}
      />
    </>
  );
}
```

이 컴포넌트는 부모가 현재 문자열을 전달하고 변경을 반영하는 계약이다. onValueChange는 개발자가 정한 API이며 브라우저의 새로운 이벤트 이름은 아니다. 내부에 같은 값을 복사한 별도 state를 만들지 않으면 부모의 값이 단일 기준으로 남는다.

부모가 이 값의 실시간 미리보기나 다른 필드 계산을 소유할 수 있다. 반대로 내부에서 독립 초안을 유지하는 컴포넌트가 필요하다면 그것을 별도의 계약으로 설명한다. 명칭만 Controlled라고 붙이는 것이 아니라 현재 값과 변경 권한의 위치를 명확히 한다.

---

## 12. 초기화·성능·접근성을 함께 검증하기

폼 reset, 명시적 state 초기화, key 변경에 의한 재생성은 영향을 주는 범위가 다르다. key를 바꾸면 입력뿐 아니라 하위 지역 state와 포커스까지 새로 만들어질 수 있다. 입력 하나를 비우려는 요구에 전체 화면을 다시 만들지 않는다.

제어 입력의 변경이 느리다면 state 소유 범위와 매 렌더링 계산을 먼저 측정한다. 비제어로 바꾸어도 제출 시 대량 검증이나 DOM 읽기 비용이 없어지는 것은 아니다. 어느 방식에서도 레이블·설명·오류·포커스와 저장 실패 처리 요구는 유지된다.

초기 로드, 빠른 타이핑, 한국어 조합, 체크 변경, 제출, reset, 대상 전환을 실제로 시험한다. state만 맞는지 또는 제출 데이터만 맞는지 따로 보지 말고 사용자가 입력한 내용과 최종 데이터가 일치하는지 확인한다.

---

## 13. 무엇을 제어하는지부터 정한다

### 일반 컴포넌트의 제어와 DOM 입력의 제어

‘제어’는 중요한 상태를 누가 소유하는지에 관한 설계 용어다. 펼침 패널이 `open`과 `onOpenChange`를 받으면 부모가 펼침 상태를 제어할 수 있다. 패널 내부의 다른 상태까지 모두 부모가 가져야 한다는 뜻은 아니다. React 공식 문서도 실제 컴포넌트에는 제어되는 정보와 내부에서 소유하는 정보가 함께 있을 수 있다고 설명한다. [컴포넌트의 제어와 비제어](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)를 참고한다.

DOM 입력에서 제어를 판단하는 대상은 입력의 현재 문자열 또는 선택 상태다. 입력이 바뀔 때 분석 로그를 보내는 `onChange`가 있어도 `value`·`checked`를 지정하지 않으면 그것만으로 제어 입력이 되지는 않는다. 반대로 입력의 값을 부모 prop에서 받아 전달한다면 그 값이 반드시 해당 입력 컴포넌트의 지역 state에 있어야 하는 것도 아니다.

### 제어 입력도 사용자의 편집은 DOM에서 시작된다

사용자가 한 글자를 입력하면 브라우저가 편집 동작을 수행하고 React의 변경 핸들러가 현재 값을 읽는다. 핸들러는 그 문자열로 state 갱신을 요청한다. 다음 렌더링에서 같은 문자열을 `value`로 주면 사용자 편집과 React의 표시 기준이 일치한다. 변경 핸들러가 값을 반영하지 않거나 뒤늦게 반영하면 React가 가진 값과 사용자의 편집 결과가 달라져 되돌아가는 증상이 생긴다.

이 흐름을 ‘키 입력이 있으면 React state가 자동 변경된다’고 줄이지 않는다. `onChange={(event) => setTitle(event.target.value)}`라는 연결을 애플리케이션이 작성해야 한다. 반면 비제어 입력은 브라우저의 현재 값이 유지되며, 애플리케이션이 필요한 시점에 읽는다. 두 방식 모두 DOM에 현재 값이 존재하지만 그 값을 다음 표시의 기준으로 정하는 주체가 다르다.

### 입력값과 제출 결과는 별개의 상태다

제출 결과를 state로 보여 준다고 폼의 모든 입력이 제어되는 것은 아니다. 비제어 입력의 편집값은 DOM에 남아 있고, 제출 순간 읽은 값은 결과 표시용 state로 복사될 수 있다. 이후 입력을 더 수정해도 이미 표시한 제출 결과는 자동으로 바뀌지 않는다. ‘현재 초안’과 ‘마지막 제출의 기록’이 서로 다른 정보를 나타내기 때문이다.

---

## 14. 체크·선택·여러 줄 입력을 실제 데이터로 확인하기

### checked와 value의 책임을 분리한다

체크박스의 `checked`는 선택 여부이며 불리언이다. `value`는 선택되었을 때 해당 이름으로 제출할 값이다. 라디오도 같은 이름의 후보 중 어느 값이 선택되었는지를 `checked`로 제어한다. 텍스트 입력을 제어하던 방식대로 체크박스의 value를 불리언으로 바꾸면 선택 상태를 제어한 것이 아니다.

다음은 독립된 `App.jsx` 예제다. 기존 React 프로젝트에서 실행한다. 선택한 현재 상태와 실제 FormData를 나란히 관찰한다.

```jsx
import { useState } from 'react';

export default function App() {
  const [shared, setShared] = useState(false);
  const [level, setLevel] = useState('basic');
  const [topics, setTopics] = useState(['html']);
  const [memo, setMemo] = useState('오늘의 학습');
  const [submitted, setSubmitted] = useState('아직 제출하지 않았습니다.');

  function handleSubmit(event) {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    setSubmitted(JSON.stringify({
      shared: data.get('shared'),
      level: data.get('level'),
      topics: data.getAll('topics'),
      memo: data.get('memo'),
    }, null, 2));
  }

  return (
    <main>
      <h1>컨트롤별 현재 값과 제출 값</h1>
      <form onSubmit={handleSubmit}>
        <label>
          <input type="checkbox" name="shared" value="yes"
            checked={shared}
            onChange={(event) => setShared(event.target.checked)} />
          공개
        </label>
        <fieldset>
          <legend>난이도</legend>
          {['basic', 'advanced'].map((option) => (
            <label key={option}>
              <input type="radio" name="level" value={option}
                checked={level === option}
                onChange={(event) => setLevel(event.target.value)} />
              {option === 'basic' ? '기초' : '심화'}
            </label>
          ))}
        </fieldset>
        <label htmlFor="topics">주제: 여러 개 선택 가능</label>
        <select id="topics" name="topics" multiple value={topics}
          onChange={(event) => {
            setTopics(Array.from(event.target.selectedOptions,
              (option) => option.value));
          }}>
          <option value="html">HTML</option>
          <option value="dom">DOM</option>
          <option value="events">Events</option>
        </select>
        <label htmlFor="memo">학습 메모</label>
        <textarea id="memo" name="memo" value={memo}
          onChange={(event) => setMemo(event.target.value)} />
        <button type="submit">제출 값 보기</button>
      </form>
      <p>현재 선택: {shared ? '공개' : '비공개'} / {level}</p>
      <pre>{submitted}</pre>
    </main>
  );
}
```

### 조작 전후의 값을 추적한다

첫 제출에서 shared는 `null`, level은 `basic`, topics는 `['html']`, memo는 `오늘의 학습`이다. 체크하지 않은 체크박스는 이름이 있는 값 항목을 만들지 않으므로 `get('shared')`가 null이다. 체크하면 state는 true가 되고 제출 값은 불리언 true가 아니라 문자열 `yes`다. 선택 여부와 전송값을 구분했기 때문에 두 값이 달라도 정상이다.

다중 select는 현재 선택을 문자열 배열로 제어한다. HTML과 DOM을 함께 선택하면 DOM의 `selectedOptions`에서 두 값을 읽어 새 배열을 만들고, 다음 렌더링이 두 옵션의 선택을 유지한다. FormData에는 같은 이름의 항목이 여러 개 들어가므로 `getAll('topics')`로 읽는다. 단일 `get`이나 `Object.fromEntries`만으로 여러 선택값 전체를 보존한다고 가정하지 않는다.

React의 textarea는 텍스트 자식 대신 `value` 또는 `defaultValue`를 사용한다. select 역시 각 option의 `selected`를 직접 관리하는 방식 대신 부모 select의 value·defaultValue로 선택을 지정한다. 이는 원시 HTML 작성법과 React API의 차이다. [React select](https://react.dev/reference/react-dom/components/select)와 [textarea](https://react.dev/reference/react-dom/components/textarea)의 계약을 각각 확인한다.

### 선택 API와 접근성을 함께 시험한다

라디오는 fieldset·legend로 후보의 공통 질문을 제공하고 각 후보에도 이름을 붙였다. 다중 select의 조작 방법은 브라우저·운영체제·기기에 따라 다를 수 있으므로 실제 대상 환경의 키보드와 터치로 여러 항목을 선택해 본다. 제어 문자열 배열이 올바르더라도 사용자에게 여러 선택이 가능하다는 사실을 전달하지 못하면 사용성 문제는 남는다.

---

## 15. 초기값·reset·대상 교체의 수명을 구분하기

### defaultValue는 현재 값의 동기화 통로가 아니다

처음 `defaultValue="HTML"`인 입력을 사용자가 DOM으로 편집하면 현재 값은 `DOM`이다. 부모가 다른 이유로 다시 렌더링되어도 value로 제어하지 않는 이 입력의 편집값을 매번 초기 문자열로 덮어쓰는 것이 목적은 아니다. 뒤늦게 받은 데이터로 현재 값을 바꾸려면 새 데이터와 사용자 초안 중 무엇을 우선할지 먼저 정한다.

제어 입력에서도 `useState(initialTitle)`의 인수는 처음 state를 만들 때 쓰인다. 이후 `initialTitle` prop이 바뀌었다고 이미 편집 중인 title state가 자동으로 교체되지 않는다. prop을 초기 초안으로 쓰는 정책과 prop을 계속 현재 값으로 쓰는 정책은 다르다.

### DOM reset과 React state 초기화를 함께 명시한다

다음 독립 예제에서 제목은 제어 입력이고 태그는 비제어 입력이다.

```jsx
import { useState } from 'react';

export default function App() {
  const [title, setTitle] = useState('HTML');
  const [result, setResult] = useState('');

  return (
    <main>
      <h1>폼 초기화</h1>
      <form onReset={() => {
        setTitle('HTML');
        setResult('');
      }} onSubmit={(event) => {
        event.preventDefault();
        const data = new FormData(event.currentTarget);
        setResult(`${data.get('title')} / ${data.get('tag')}`);
      }}>
        <label htmlFor="reset-title">제목</label>
        <input id="reset-title" name="title" value={title}
          onChange={(event) => setTitle(event.target.value)} />
        <label htmlFor="reset-tag">태그</label>
        <input id="reset-tag" name="tag" defaultValue="기초" />
        <button type="submit">결과 확인</button>
        <button type="reset">모두 초기화</button>
      </form>
      <p>현재 제목 state: {title}</p>
      <p role="status">{result}</p>
    </main>
  );
}
```

제목·태그를 편집하고 제출한 뒤 reset을 누른다. 브라우저의 폼 초기화는 비제어 태그를 기본값으로 돌린다. React가 소유한 title과 결과 문구는 onReset에 작성한 setter로 초기화한다. 이 코드를 보고 ‘HTML reset이 모든 React state를 자동 초기화한다’고 결론 내리지 않는다. onReset의 두 setter를 제거하는 대비 실행을 하면 state를 초기화하는 정책이 빠졌다는 사실을 확인할 수 있다. 제어 입력의 순간적인 DOM 값은 브라우저의 reset 처리와 React 반영 시점에 영향을 받을 수 있으므로 state와 다음 렌더링까지 함께 관찰한다.

### 다른 대상을 선택할 때 초안을 버릴지 보존할지 정한다

다음은 전환할 때 초안을 버리고 선택한 대상의 초기값으로 다시 시작하는 정책이다.

```jsx
import { useState } from 'react';

const notes = [
  { id: 'html', title: 'HTML 학습' },
  { id: 'dom', title: 'DOM 학습' },
];

function DraftEditor({ initialTitle }) {
  const [title, setTitle] = useState(initialTitle);
  return <label>
    편집 제목
    <input value={title} onChange={(event) => setTitle(event.target.value)} />
  </label>;
}

export default function App() {
  const [selectedId, setSelectedId] = useState('html');
  const selected = notes.find((note) => note.id === selectedId);
  return (
    <main>
      <h1>대상 전환 시 초안 초기화</h1>
      <p>다른 대상을 선택하면 저장하지 않은 수정은 버려집니다.</p>
      {notes.map((note) => <button type="button" key={note.id}
        aria-pressed={selectedId === note.id}
        onClick={() => setSelectedId(note.id)}>{note.title}</button>)}
      <DraftEditor key={selected.id} initialTitle={selected.title} />
    </main>
  );
}
```

HTML 제목을 편집한 뒤 DOM을 선택하면 key가 바뀌어 DraftEditor가 새로 시작한다. 다시 HTML로 돌아와도 저장한 초안이 없으므로 원래 제목으로 시작한다. key를 제거한 대비 실행에서는 같은 위치·타입의 DraftEditor state가 이어져 선택 대상이 바뀌었는데 이전 제목이 남을 수 있다. 이것은 useState의 초기 인수가 매 렌더링 동기화 규칙이 아니라는 사례다.

전환 후 돌아왔을 때 초안을 보존해야 하는 제품이라면 부모가 ID별 초안을 소유하는 등 별도 보존 정책이 필요하다. key 변경은 현재 컴포넌트의 입력·지역 state와 DOM을 함께 다시 만드는 선택이므로 포커스와 하위 UI도 영향을 받는다. [React의 key를 통한 폼 초기화](https://react.dev/learn/preserving-and-resetting-state#resetting-a-form-with-a-key)는 이 범위를 판단하는 근거다.

---

## 16. 편집 가능한 문자열과 제출할 데이터의 차이

### 숫자도 입력 중에는 문자열로 관리할 수 있다

사용자는 빈 입력에서 시작하고, 지우고, 소수점이나 부호를 입력하는 도중을 거친다. 매 변경마다 무조건 Number로 변환하면 빈 문자열이 0이 되는 등 편집 과정의 의미가 사라질 수 있다. 다음 독립 예제는 현재 문자열을 그대로 보관하고, 제출할 때 해석한다.

```jsx
import { useState } from 'react';

export default function App() {
  const [minutes, setMinutes] = useState('');
  const [result, setResult] = useState('');

  function handleSubmit(event) {
    event.preventDefault();
    if (minutes.trim() === '') {
      setResult('학습 시간을 입력해 주세요.');
      return;
    }
    const value = Number(minutes);
    if (!Number.isInteger(value) || value < 1 || value > 180) {
      setResult('1부터 180까지의 정수를 입력해 주세요.');
      return;
    }
    setResult(`기록할 시간: ${value}분`);
  }

  return (
    <main>
      <h1>학습 시간</h1>
      <form noValidate onSubmit={handleSubmit}>
        <label htmlFor="minutes">분 단위 학습 시간</label>
        <input id="minutes" name="minutes" type="number"
          min="1" max="180" step="1" value={minutes}
          aria-describedby="minutes-help"
          onChange={(event) => setMinutes(event.target.value)} />
        <p id="minutes-help">1~180분의 정수로 입력합니다.</p>
        <button type="submit">값 해석</button>
      </form>
      <p>현재 문자열: {JSON.stringify(minutes)}</p>
      <p role="status">{result}</p>
    </main>
  );
}
```

빈 입력을 제출하면 ‘입력해 주세요’로 분류하고, `0`·`2.5`·`181`은 범위·정수 조건으로 거부한다. 유효한 `30`은 숫자 30으로 해석한다. noValidate는 이 실험에서 브라우저가 제출을 먼저 막지 않게 하여 코드의 검증 결과를 관찰하기 위한 선택이다. 제품에서는 브라우저의 기본 검증과 오류 표시 정책을 함께 결정한다.

`type="number"`는 모든 임시 문자열을 그대로 보존하는 범용 텍스트 편집기가 아니다. 브라우저가 숫자로 해석할 수 없는 입력을 제한하거나 빈 value로 제공할 수 있으며 UI도 다를 수 있다. 정확한 원문 숫자 표기 자체가 필요한 요구라면 text 입력과 적절한 inputMode·검증 정책을 검토한다. `valueAsNumber`를 사용할 때도 빈 값의 NaN을 정상 숫자와 구별해야 한다. [HTML 숫자 입력 규칙](https://html.spec.whatwg.org/multipage/input.html#number-state-(type=number))과 [MDN valueAsNumber](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/valueAsNumber)를 근거로 판단한다.

### 저장용 변환을 타이핑 때마다 강제하지 않는다

텍스트를 매번 trim하거나 대문자로 바꾸어 value에 돌려주면 사용자의 커서 위치와 편집 의도를 바꿀 수 있다. 입력 중 문자열은 즉시 저장하고 정규화는 제출 등 명시한 시점에 수행하는 방식을 우선 검토한다. 한국어 조합 입력에서는 아직 완성되지 않은 글자도 있으므로 붙여넣기·문자열 중간 수정·한글 조합을 포함해 시험한다.

제어 입력의 state 갱신 자체를 타이머로 늦추는 것은 검색 요청을 늦추는 것과 다르다. 현재 입력의 backing value는 변경 이벤트에서 즉시 반영하고, 별도의 무거운 작업의 실행 시점을 따로 설계한다. [React 입력 문제 해결](https://react.dev/reference/react-dom/components/input#my-input-caret-jumps-to-the-beginning-on-every-keystroke)은 다른 값으로 갱신하거나 비동기로 갱신할 때의 편집 문제를 설명한다.

---

## 17. 읽기 전용·비활성화·파일 입력의 경계

### readOnly와 disabled는 제출 결과도 다르다

다음 독립 예제는 사용자가 편집할 수 없는 두 필드를 FormData로 비교한다.

```jsx
import { useState } from 'react';

export default function App() {
  const [result, setResult] = useState('');
  return (
    <main>
      <h1>편집 제한과 제출</h1>
      <form onSubmit={(event) => {
        event.preventDefault();
        const data = new FormData(event.currentTarget);
        setResult(JSON.stringify([...data.entries()]));
      }}>
        <label htmlFor="record-code">기록 코드</label>
        <input id="record-code" name="recordCode" value="R-001" readOnly />
        <label htmlFor="locked-topic">사용할 수 없는 주제</label>
        <input id="locked-topic" name="lockedTopic" value="DOM" disabled />
        <button type="submit">제출 항목 확인</button>
      </form>
      <p role="status">{result}</p>
    </main>
  );
}
```

결과에는 recordCode가 있고 lockedTopic은 없다. 읽기 전용 텍스트는 일반적으로 포커스·선택·복사가 가능하며 이름이 있으면 제출 대상이 될 수 있다. 비활성화한 컨트롤은 사용자 상호작용과 제출에서 제외되는 성격이 다르다. 편집할 수 없게 보이게 하는 CSS만 적용한 경우에는 두 의미가 자동으로 생기지 않는다.

텍스트 입력의 readOnly는 의도적으로 현재 값을 고정한다는 표현이며 React의 변경 핸들러 누락 경고를 피할 때도 사용한다. 이를 사용자가 바꿀 수 있는 제어 입력의 onChange 대용으로 쓰지 않는다. 또한 readonly·disabled 어느 쪽도 서버 데이터의 신뢰성을 보장하는 보안 경계가 아니다.

### 파일 입력의 선택은 일반 value로 만들지 않는다

파일 선택을 관찰하는 독립 예제다. names는 선택 결과를 표시하는 React state이고, 파일 입력의 현재 선택을 value prop으로 제어하는 state가 아니다.

```jsx
import { useState } from 'react';

export default function App() {
  const [names, setNames] = useState([]);
  return (
    <main>
      <h1>학습 자료 선택</h1>
      <label htmlFor="attachments">자료 파일</label>
      <input id="attachments" name="attachments" type="file" multiple
        onChange={(event) => {
          setNames(Array.from(event.target.files ?? [], (file) => file.name));
        }} />
      <p role="status">{names.length ? names.join(', ') : '선택한 파일 없음'}</p>
    </main>
  );
}
```

사용자가 고른 파일은 FileList를 통해 읽는다. 문자열 경로나 파일명을 value에 지정한다고 해당 로컬 파일이 선택되는 것은 아니다. HTML은 파일 입력의 value에 비어 있지 않은 문자열을 대입하는 동작을 제한한다. 파일명 표시만으로 실제 업로드가 일어나지도 않는다. [HTML 파일 입력](https://html.spec.whatwg.org/multipage/input.html#file-upload-state-(type=file))과 [MDN files](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/files)를 구분해서 읽는다.

---

## 18. 입력이 되돌아가는 버그를 소유권으로 추적하기

### 증상과 최소 재현

`const [title, setTitle] = useState('HTML')`인 입력에 `value={title}`만 지정했다고 가정한다. 사용자가 DOM을 입력해도 문자가 유지되지 않고 개발 환경에는 변경 처리 관련 경고가 나타날 수 있다. 상태 변수와 입력이 연결되어 있다는 사실만으로 사용자의 변경 경로까지 구현한 것은 아니다.

먼저 React DevTools에서 title이 입력 중에도 HTML인지 확인한다. 변경 이벤트에서 읽는 DOM value와 렌더링에서 전달하는 title을 따로 관찰한다. DOM에서는 새 문자열을 읽었는데 다음 렌더링의 value가 이전 문자열이면, 전달한 값의 소유자가 갱신되지 않는 것이 원인이다. 편집 가능 요구라면 `onChange`에서 현재 문자열을 즉시 setter로 반영한다. 고정 값 요구라면 readOnly를 명시한다. 수정 후 중간 삽입·전체 삭제·한글 입력을 다시 시험한다.

### 비동기 초기 데이터의 모드 전환도 검사한다

처음 `undefined`였다가 문자열이 되는 값을 그대로 value에 넘기면 비제어에서 제어로 바뀌는 문제가 생길 수 있다. 문자열 제어 입력은 처음부터 문자열을 받도록 `value={loadedTitle ?? ''}` 같은 기준을 둔다. 다만 null 대체만으로 서버의 새 값과 사용자 초안을 합치는 정책까지 해결되지는 않는다. 데이터 로드 후의 편집 기준과 대상 전환의 초기화 기준을 별도로 정한다.

### 느린 입력은 state 범위와 작업량으로 확인한다

제어 방식은 값이 바뀔 때 해당 state를 사용하는 UI도 다시 계산할 수 있다. 병목이 생기면 같은 긴 문자열을 같은 속도로 입력하며 React DevTools와 브라우저 Performance에서 입력 처리와 컴포넌트 계산 시간을 관찰한다. 현재 값과 무관한 큰 목록까지 함께 계산된다면 입력 state를 필요한 컴포넌트 가까이에 두는 선택을 검토한다. 비제어로 바꾸어도 제출 검증·저장 비용이나 접근성 요구는 남는다. ‘제어는 느리고 비제어는 빠르다’는 단일 규칙으로 결정하지 않는다.

---

## 19. 어떤 방식을 선택할까

입력 즉시 다른 UI를 계산하거나 여러 값의 관계를 표시해야 하면 제어 입력이 명확할 수 있다. 제출할 때만 필요한 단순 폼은 비제어 입력과 FormData로 충분할 수 있다. 어느 방식이든 이름·설명·오류·키보드 조작을 제공해야 하며 제어 입력이라는 사실 자체가 접근성을 보장하지 않는다.

제어 입력에서 매 입력마다 넓은 트리를 다시 계산하면 지연이 생길 수 있다. 값을 입력 가까이에 두고 병목을 측정한 뒤 범위를 조정한다. 성능을 이유로 모든 폼을 비제어로 바꾸거나, 일관성을 이유로 모든 입력을 전역 state로 올리는 방식으로 일반화하지 않는다.

DevTools에서 React state와 DOM 프로퍼티를 각각 확인하고, 초기 진입·편집·제출·초기화·데이터 교체를 따로 실험한다. 입력값이 되돌아간다면 어느 렌더링에서 어떤 value가 전달되었는지 추적한다.

---

## 20. 핵심 정리

<callout icon="✅" color="green_bg">
제어 입력은 React가 현재 값을 지정하고 비제어 입력은 DOM이 현재 값을 유지한다. 초기값과 현재 값, 폼 reset과 React state 초기화는 별개의 흐름이다. 요구되는 동기화와 읽기 시점에 따라 선택한다.
</callout>

---

## 출처

- [React — input](https://react.dev/reference/react-dom/components/input)
- [React — Sharing State: Controlled and Uncontrolled Components](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components)
- [React — Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state)
- [MDN — FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [React — select](https://react.dev/reference/react-dom/components/select)
- [React — textarea](https://react.dev/reference/react-dom/components/textarea)

### 이번 설명의 근거와 적용 범위

- [React — input](https://react.dev/reference/react-dom/components/input): value·checked와 defaultValue·defaultChecked의 구분, 일관된 제어 모드, 변경 핸들러의 즉시 갱신, 커서 이동 문제를 확인했다.
- [React — Sharing State](https://react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components): 일반 컴포넌트에서 props와 내부 state가 만드는 제어 범위를 확인했다.
- [React — select](https://react.dev/reference/react-dom/components/select): 문자열·문자열 배열의 value, option의 selected와의 차이, FormData의 다중 값 처리 범위를 확인했다.
- [React — textarea](https://react.dev/reference/react-dom/components/textarea): 자식 텍스트 대신 value·defaultValue를 사용하는 계약을 확인했다.
- [React — Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state#resetting-a-form-with-a-key): key 변경에 따른 폼 하위 상태의 초기화 범위를 확인했다.
- [HTML — input의 상태별 규칙](https://html.spec.whatwg.org/multipage/input.html): 숫자 입력의 값 정리와 파일 입력의 값 제한을 확인했다. [MDN valueAsNumber](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/valueAsNumber)는 문자열과 숫자·NaN의 차이를 보충한다.
- [HTML — FormData 항목 구성](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#constructing-the-entry-list): 체크되지 않은 체크박스·비활성화 컨트롤 제외와 여러 선택값의 제출 구조를 확인했다.

---

## 21. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. value만 전달한 텍스트 입력에서 글자가 되돌아간다. 현재 값의 기준과 변경 경로로 설명하라.
2. 비제어 입력을 FormData로 제출한 뒤 결과를 state에 저장하면 입력도 제어 방식이 되는가?
3. 비제어 폼 reset 이후 입력은 바뀌었는데 제출 결과 문구는 남았다. 두 상태의 소유자가 누구인지 설명하라.
4. 입력 즉시 미리보기가 필요한 폼과 제출 시에만 값을 읽는 폼에서 각각 어떤 방식을 선택하겠는가?
