---
track: B
notion: https://app.notion.com/p/406178c4afac80d09c38e02f96e103e3
---

# Form / Input / Accessibility 기초


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · Form / Input / Accessibility 기초

Form의 제출 모델, Input의 상태와 제약, Accessibility의 이름·조작·피드백을 각각 익힌 뒤 하나의 입력 흐름으로 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- action·method·name·value로 실제 제출 데이터를 설명할 수 있다.
- 입력 타입, 초기 속성, 현재 프로퍼티, disabled·readonly의 차이를 판단할 수 있다.
- 레이블·설명·그룹·포커스·오류 안내를 포함한 폼을 만들 수 있다.
- 사용자 클릭·Enter·프로그램 제출에서 검증과 이벤트의 차이를 설명할 수 있다.
- 키보드·접근성 트리·Network로 사용 가능성과 실제 전송을 함께 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

폼은 값을 담는 상자 몇 개가 아니다. 사용자가 무엇을 입력해야 하는지 이해하고, 입력하고, 오류를 고쳐 목적을 달성하는 하나의 인터페이스다. 접근성은 이 흐름의 각 단계가 실제로 사용 가능한지를 묻는다. 따라서 Form, Input, Accessibility를 따로 붙이는 대신 함께 설계해야 한다.

---

## 3. 핵심 개념과 용어

`form`은 입력 컨트롤을 제출 흐름으로 묶는다. `input`, `select`, `textarea`, `button`은 각기 다른 입력·선택·실행을 제공한다. `label`은 컨트롤의 이름을 연결하고, `fieldset`과 `legend`는 관련 입력의 그룹과 제목을 나타낸다.

| 속성·정보 | 역할 |
| --- | --- |
| `id` | 문서에서 요소를 식별하고 `label for` 등과 연결 |
| `name` | 제출 데이터의 키 |
| `value` | 텍스트 입력의 현재 값, 또는 선택 컨트롤이 제출할 값 |
| `checked` | 체크박스·라디오의 현재 선택 여부를 다루는 DOM 프로퍼티 |
| 접근성 이름 | 보조 기술이 컨트롤을 식별하는 이름 |

`id`가 있다고 값이 제출되는 것은 아니다. `name`이 있어야 일반적인 폼 데이터의 항목이 된다. HTML 속성은 초기 상태에 관여하고, 사용자가 편집한 현재 값은 DOM 프로퍼티로 관리된다. 체크박스의 HTML `checked` 속성과 현재 `.checked`가 계속 같은 값일 것이라고 가정하면 안 된다.

---

## 4. 관련 계층과 역할 구분

| 계층 | 폼에서 담당하는 역할 |
| --- | --- |
| HTML | 입력 종류, 레이블, 그룹, 제출 주소, 검증 조건을 표현한다. |
| 브라우저 | 입력 상태 관리, 기본 검증, 키보드 조작, 제출 요청을 처리한다. |
| CSS | 배치, 간격, 오류와 포커스의 시각적 표현을 담당한다. |
| JavaScript | 필요할 때 추가 검증이나 제출 처리 방식을 구현한다. |
| 서버 | 실제 요청을 처리하고 데이터를 검증·저장한다. |

CSS로 빨간 테두리를 그렸다고 입력이 실제로 무효 상태가 되지는 않는다. 반대로 HTML의 `required`로 필수 조건을 선언하면 브라우저는 자체 검증에 그 조건을 사용한다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

클라이언트에서 검증했더라도 서버 검증을 생략해서는 안 된다. 브라우저의 제약은 우회할 수 있기 때문이다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

---

## 5. Form: 입력을 제출 가능한 데이터로 묶기

### 폼과 폼 컨트롤

폼은 사용자가 입력한 정보를 하나의 제출 작업으로 묶는 HTML 구조다.

폼 컨트롤은 입력하거나 선택하거나 제출하는 데 사용하는 요소다.

| 요소 | 역할 |
| --- | --- |
| `<form>` | 입력 제출의 범위와 제출 방법을 정의한다. |
| `<input>` | 한 줄 텍스트, 체크박스, 라디오 등 다양한 입력을 제공한다. |
| `<textarea>` | 여러 줄의 텍스트를 입력받는다. |
| `<select>` | 선택지 중 값을 선택하게 한다. |
| `<button>` | 제출 등 사용자의 동작을 실행한다. |
| `<label>` | 입력의 이름을 제공한다. |
| `<fieldset>` | 관련된 입력들을 의미상 묶는다. |
| `<legend>` | `fieldset`으로 묶은 입력 그룹의 이름을 제공한다. |

입력의 이름과 그룹을 HTML로 표현해야 화면을 보는 사용자뿐 아니라 보조 기술을 사용하는 사용자도 입력의 목적과 관계를 이해할 수 있다.[1](https://www.w3.org/WAI/tutorials/forms/)

### `action`: 어디로 제출하는가

```html
<form action="/search">
  <!-- 입력 요소 -->
</form>
```

`action`은 폼을 처리할 주소다.

`/search`처럼 `/`로 시작하는 경로는 현재 사이트의 루트를 기준으로 해석한다. 실제 검색 결과를 제공하려면 해당 주소에서 요청을 처리하는 기능이 있어야 한다.

HTML만 작성했다고 검색 서비스나 데이터 저장 기능까지 생기는 것은 아니다.

### `method`: 어떤 방식으로 제출하는가

기본적인 폼에서는 `GET`과 `POST`를 구분한다.

| 방식 | 데이터가 전달되는 위치 | 대표적인 용도 |
| --- | --- | --- |
| `GET` | URL의 쿼리 문자열 | 검색, 필터처럼 조회 조건을 전달하는 작업 |
| `POST` | HTTP 요청 본문 | 가입, 등록처럼 데이터를 처리하거나 변경하는 작업 |

예를 들어 `q`라는 이름으로 `browser`를 제출하면 GET 요청의 주소는 다음과 같은 형태가 된다.

```txt
/search?q=browser
```

POST는 입력 데이터를 URL 대신 요청 본문에 담지만, 그 자체가 암호화를 의미하지는 않는다. 비밀번호처럼 민감한 정보를 다루는 서비스는 HTTPS를 사용해야 한다.

### `name`과 `value`: 어떤 데이터로 제출하는가

```html
<label for="search-query">검색어</label>
<input id="search-query" name="q" type="search">
```

각 속성은 서로 다른 일을 한다.

- `id`: 문서 안에서 요소를 식별하고 `label`과 연결한다.
- `name`: 제출 데이터의 키가 된다.
- 현재 입력값: 제출 데이터의 값이 된다.

사용자가 `browser`를 입력하면 `q=browser`라는 이름과 값의 쌍이 만들어진다.

`id="search-query"`만 있고 `name`이 없다면 일반적인 폼 제출 데이터에 해당 입력값은 포함되지 않는다. 화면에서 입력할 수 있다는 것과 제출 대상이라는 것은 별개다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 기본 제출 흐름

일반적인 네이티브 폼 제출은 다음과 같이 진행된다.

1. 사용자가 제출 버튼을 활성화한다.
2. 브라우저가 `required`, 입력 타입 등의 제약을 검사한다.
3. 유효하지 않은 입력이 있으면 제출을 막고 문제를 안내한다.
4. 검증을 통과하면 폼에서 `submit` 이벤트가 발생한다.
5. 제출이 취소되지 않았다면 제출 데이터를 구성한다.
6. `action`과 `method`에 따라 요청하고, 일반적인 페이지 폼에서는 응답으로 이동한다.

> 💡 유효성 검사를 통과하지 못한 제출 시도에서는 `submit` 이벤트도 발생하지 않는다는 점이 중요하다.[2](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)

### 버튼의 종류를 명시해야 하는 이유

```html
<button type="submit">검색</button>
<button type="button">검색 도움말</button>
<button type="reset">초기화</button>
```

- `submit`: 폼 제출을 시도한다.
- `button`: 자체적인 제출 동작이 없는 일반 버튼이다.
- `reset`: 폼 컨트롤을 초기 상태로 되돌린다.

폼 안에서 일반적인 `<button>`의 `type`을 생략하면 제출 버튼으로 동작할 수 있다. 따라서 도움말, 미리보기, 항목 추가 버튼에도 종류를 명시하는 습관이 필요하다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

초기화는 “모든 값을 빈 문자열로 만든다”는 뜻이 아니다. 초기값이 있으면 그 값으로 돌아간다. 사용자가 입력한 내용을 실수로 잃을 수 있으므로 초기화 버튼은 꼭 필요한 경우에만 제공한다.[5](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultValue)[6](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultChecked)

### 제출 데이터에 포함되지 않는 경우

대표적으로 다음 입력은 일반적인 제출 데이터에서 제외된다.

- `name`이 없는 입력
- 비활성화된 `disabled` 입력
- 선택되지 않은 체크박스와 라디오 버튼

체크박스는 선택되었을 때만 자신의 `name`과 `value`를 제출한다. 선택하지 않았다고 자동으로 `false`가 제출되는 것은 아니다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

### 제출 API와 자주 놓치는 경로

### “버튼의 클릭만 처리하면 폼 제출을 모두 처리한 것이다”

버튼 클릭과 폼 제출은 같지 않다. 일부 입력에서는 Enter 키로도 제출할 수 있다.

따라서 제출 처리가 필요할 때는 버튼 클릭만 바라보지 말고 폼의 제출을 기준으로 생각해야 한다. Enter 제출의 구체적인 동작은 폼 구성과 입력 종류에 따라 달라진다.[2](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)

### “POST이면 비밀번호가 안전하다”

POST는 데이터의 위치를 바꾼다. 통신 암호화나 서버의 안전한 저장까지 보장하지 않는다.

### “`required`를 넣었으니 검증은 끝났다”

브라우저 검증은 사용자에게 빠른 피드백을 제공하는 장치다. 서버의 신뢰 경계를 대신하지 않는다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

### “폼 안에 폼을 넣어 그룹을 나누면 된다”

`form`은 중첩하지 않는다. 관련 입력의 그룹은 `fieldset`과 `legend`로 표현한다. 잘못 중첩하면 브라우저가 해석한 실제 구조가 작성 의도와 달라질 수 있다.[1](https://www.w3.org/WAI/tutorials/forms/)[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### “프로그램에서 제출하는 API는 모두 같다”

`requestSubmit()`은 검증과 `submit` 이벤트를 거치는 제출 경로를 제공한다. 반면 `submit()`은 이 과정을 건너뛴다. 두 방법을 동일한 동작으로 취급하면 검증이나 제출 처리 코드가 예상대로 실행되지 않을 수 있다.[8](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/requestSubmit)

---

## 6. Input: 입력 종류·현재 상태·검증 제약

### 입력 타입

| 타입 | 목적 | 주의점 |
| --- | --- | --- |
| `text` | 일반적인 한 줄 문자열 | 형식 검증을 자동으로 제공하지 않는다. |
| `search` | 검색어 | 부가 UI는 브라우저마다 다를 수 있다. |
| `email` | 이메일 형식 문자열 | 실제 존재하는 주소인지는 확인하지 않는다. |
| `password` | 화면에서 가려서 표시할 문자열 | 전송·저장을 암호화하지 않는다. |
| `tel` | 전화번호 문자열 | 국가별 전화번호 형식을 자동 검증하지 않는다. |
| `number` | 수량처럼 수치 연산이 필요한 값 | 문자열 식별자와 구분한다. |
| `date` | 날짜 | 선택 UI가 환경마다 다를 수 있다. |
| `checkbox` | 독립적인 선택 여부 | 상태는 `checked`로 읽는다. |
| `radio` | 그룹 중 하나 선택 | 같은 `name`이 그룹을 형성한다. |
| `file` | 사용자가 선택한 파일 | 일반 문자열 입력과 다르게 다룬다. |

입력 타입은 모양보다 데이터의 의미를 기준으로 고른다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 숫자처럼 보이는 값이 모두 수량은 아니다

전화번호, 우편번호, 회원번호는 숫자로만 구성되어도 보통 문자열 식별자다.

다음 질문이 판단에 도움이 된다.

> 이 값에 1을 더하거나 빼는 것이 의미 있는가?

상품 수량이라면 의미가 있다. 우편번호라면 보통 의미가 없다.

```html
<label for="postal-code">우편번호</label>
<input
  id="postal-code"
  name="postalCode"
  type="text"
  inputmode="numeric"
  autocomplete="postal-code"
>
```

이렇게 하면 값은 문자열로 유지하면서 모바일 환경에 숫자 입력에 적합한 키보드를 요청할 수 있다. `inputmode`는 입력 방식에 대한 힌트이지, 숫자만 허용하는 검증 규칙이 아니다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 레이블과 placeholder

```html
<label for="email">이메일</label>
<input
  id="email"
  name="email"
  type="email"
  placeholder="name@example.com"
>
```

- `label`: 무엇을 입력하는지 알려 주는 이름이다.
- `placeholder`: 입력이 비어 있을 때 보여 주는 예시나 짧은 힌트다.

placeholder는 입력하면 사라지므로 지속적인 이름이나 필수 안내를 맡기기에 적절하지 않다. 입력에는 별도의 레이블을 제공한다.[1](https://www.w3.org/WAI/tutorials/forms/)[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 자동완성

`autocomplete`은 입력 정보의 목적을 브라우저에 알려 주는 힌트다.

```html
<input type="text" autocomplete="name">
<input type="email" autocomplete="email">
<input type="password" autocomplete="current-password">
<input type="password" autocomplete="new-password">
```

기존 비밀번호를 받는 로그인과 새로운 비밀번호를 만드는 가입은 목적이 다르다. 목적에 맞게 작성하면 브라우저와 비밀번호 관리 도구가 입력을 지원하는 데 도움이 된다. 실제 자동완성 동작은 브라우저와 사용자 설정에 따라 달라진다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 초기값과 현재 값

### 텍스트 입력의 `value`

```html
<input id="nickname" type="text" value="해원">
```

이 상태에서 사용자가 화면의 값을 `해원님`으로 수정했다고 생각해 보자.

| 구분 | 의미 | 예시 결과 |
| --- | --- | --- |
| HTML의 `value` 속성 | 기본값을 선언한다. | `"해원"` |
| DOM의 `defaultValue` | `value` 속성을 반영하는 기본값이다. | `"해원"` |
| DOM의 `value` | 현재 입력값이다. | `"해원님"` |

> 💡 사용자 입력을 읽을 때는 현재 상태인 `value`가 기준이다. 기본값과 현재 값은 처음에는 같아 보여도 사용자가 수정한 뒤에는 달라질 수 있다.[5](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultValue)[11](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)

### 체크박스의 `checked`

```html
<input
  id="news"
  type="checkbox"
  name="news"
  value="weekly"
  checked
>
```

체크박스에는 두 종류의 정보가 있다.

- `value`: 선택되었을 때 제출할 값인 `"weekly"`
- `checked`: 현재 선택되어 있는지 나타내는 참·거짓 상태

따라서 `value`를 읽어서는 선택 여부를 판단할 수 없다.

HTML의 `checked` 속성과 DOM의 `defaultChecked`는 기본 선택 상태를 표현한다. 사용자가 선택을 해제한 뒤 현재 상태를 알려면 DOM의 `checked`를 확인해야 한다.[6](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultChecked)[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

또한 `checked`와 `required` 같은 HTML 불리언 속성은 존재 여부로 해석한다.

```html
<!-- 기본 선택 상태다. -->
<input type="checkbox" checked="false">
```

`checked="false"`라고 적어도 속성이 존재하므로 기본 선택 상태가 된다. 기본 선택을 없애려면 속성을 제거한다.[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

### 입력 제약과 편집 가능 상태

### 자주 사용하는 제약

```html
<label for="account-name">계정 이름 (필수, 3~20자)</label>
<input
  id="account-name"
  name="accountName"
  type="text"
  required
  minlength="3"
  maxlength="20"
>
```

| 속성 | 의미 |
| --- | --- |
| `required` | 해당 입력을 필수로 만든다. |
| `minlength` | 텍스트의 최소 길이를 정한다. |
| `maxlength` | 텍스트의 최대 길이를 정한다. |
| `min` | 수치나 날짜 등의 최솟값을 정한다. |
| `max` | 수치나 날짜 등의 최댓값을 정한다. |
| `step` | 허용하는 수치 간격 등을 정한다. |
| `pattern` | 지원하는 텍스트 입력에 형식 조건을 정한다. |

이 속성들은 모든 입력 타입에 동일하게 적용되지 않는다. 예를 들어 수량 범위는 `min`과 `max`, 텍스트 길이는 `minlength`와 `maxlength`로 표현한다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)[10](https://developer.mozilla.org/docs/Web/HTML/Guides/Constraint_validation)

`minlength`만으로 필수 입력이 되는 것도 아니다. 빈 값을 금지하려면 `required`가 필요하다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

### `disabled`와 `readonly`

| 구분 | `disabled` | `readonly` |
| --- | --- | --- |
| 사용자 수정 | 불가능 | 불가능 |
| 일반적인 Tab 접근 | 제외된다. | 가능하다. |
| 폼 제출 데이터 | 제외된다. | `name` 등이 갖춰지면 포함된다. |
| 적용 범위 | 여러 폼 컨트롤 | 지원하는 텍스트 계열 입력 등 |

`readonly`는 체크박스나 `select`를 읽기 전용으로 만드는 범용 속성이 아니다. 사용할 요소와 타입의 지원 여부를 확인해야 한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

두 속성 모두 보안 장치가 아니다. 클라이언트 값을 신뢰해 권한이나 결제 금액을 결정해서는 안 된다.

---

## 7. Accessibility: 이름·조작·오류를 사용자에게 전달하기

### 의미 있는 HTML

의미 있는 HTML은 요소의 역할에 맞는 태그를 사용하는 것이다.

- 이동은 `<a href="...">`
- 실행은 `<button>`
- 입력은 `<input>`, `<textarea>`, `<select>`
- 입력 이름은 `<label>`
- 관련 입력 그룹은 `<fieldset>`과 `<legend>`

기본 요소는 역할 정보뿐 아니라 브라우저의 조작 규칙도 제공한다. 반면 일반 요소에 ARIA 역할을 붙이는 것만으로 네이티브 요소의 키보드 동작이 생기지는 않는다.[1](https://www.w3.org/WAI/tutorials/forms/)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 접근 가능한 이름과 설명

접근 가능한 이름은 보조 기술이 컨트롤을 식별하는 데 사용하는 이름이다.

```html
<label for="contact-email">이메일</label>
<input id="contact-email" type="email">
```

여기서 입력의 이름은 `이메일`이다.

설명은 이름에 덧붙는 도움말이나 조건이다.

```html
<label for="contact-email">이메일</label>

<p id="email-help">
  답변을 받을 수 있는 주소를 입력하세요.
</p>

<input
  id="contact-email"
  type="email"
  aria-describedby="email-help"
>
```

`aria-describedby`는 설명 요소의 `id`를 참조해 관계를 표현한다. 이름과 설명은 같은 역할이 아니므로, 설명만 연결하고 레이블을 생략하지 않는다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 역할·상태·값

사용자는 컨트롤을 만났을 때 다음을 알아야 한다.

- 이름: 무엇을 위한 것인가?
- 역할: 버튼인가, 입력인가, 체크박스인가?
- 상태: 선택되어 있는가, 비활성화되었는가?
- 값: 현재 무엇이 입력되어 있는가?

기본 HTML 입력은 이러한 정보의 상당 부분을 브라우저가 관리한다. 체크박스를 직접 그리는 것보다 기본 체크박스를 사용하는 편이 역할과 상태를 일관되게 제공하기 쉽다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 포커스

포커스는 현재 키보드 입력의 대상이 되는 위치다.

일반적인 키보드 사용에서는 다음 동작을 확인한다.

- Tab: 다음 조작 대상으로 이동
- Shift+Tab: 이전 조작 대상으로 이동
- Enter 또는 Space: 컨트롤 종류에 맞는 실행
- 방향키: 라디오 그룹 등에서 선택 이동

모든 컨트롤이 동일한 키로 동작하는 것은 아니다. 예를 들어 여러 라디오 버튼은 보통 Tab으로 그룹에 접근한 뒤 방향키로 선택을 이동한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### ARIA

ARIA는 기본 HTML만으로 충분히 표현하기 어려운 역할, 상태, 관계 등을 보조 기술에 전달하는 속성 체계다.

하지만 ARIA는 실제 동작을 구현하는 도구가 아니다.

```html
<div role="button">저장</div>
```

이렇게 작성해도 자동으로 다음이 생기지는 않는다.

- Tab으로 접근하는 동작
- Enter와 Space로 실행하는 동작
- 실제 저장 기능

> 💡 기본 `<button>`으로 표현할 수 있다면 먼저 기본 요소를 사용한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 사용자가 폼을 완료하는 순서

레이블과 설명이 연결된 입력으로 키보드 포커스가 이동하는 상황을 생각해 보자.

1. 브라우저가 입력의 역할과 이름을 해석한다.
2. 연결된 도움말이 있다면 설명 관계를 구성한다.
3. 필수 여부와 현재 값 등 관련 상태를 제공한다.
4. 보조 기술은 이 정보를 사용자에게 전달한다.
5. 사용자는 입력하고 결과를 확인한다.

실제 읽는 순서와 문구는 브라우저·스크린 리더·설정에 따라 달라질 수 있다. 특정 음성 출력 문장을 보장하기보다는 이름, 설명, 역할, 상태의 관계가 정확한지 확인한다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 오류와 완료 결과 전달

### 색상만으로 오류를 전달하지 않는다

빨간 테두리는 시각적 보조 신호일 수 있지만, 무엇이 잘못되었는지 충분히 설명하지 못한다.

```html
<label for="signup-email">이메일</label>

<input
  id="signup-email"
  name="email"
  type="email"
  value="wrong-address"
  aria-invalid="true"
  aria-describedby="signup-email-error"
>

<p id="signup-email-error">
  이메일에 @를 포함해 주세요.
</p>
```

이 코드는 오류가 확인된 상태를 표현하는 정적 예시다.

- `aria-invalid="true"`는 현재 입력이 잘못되었다는 상태를 전달한다.
- 오류 문장은 수정 방법을 알려 준다.
- `aria-describedby`는 오류 문장과 입력을 연결한다.

`aria-invalid` 자체가 검증을 실행하거나 제출을 막지는 않는다. 실제 검증 결과와 화면의 오류, 접근성 상태가 일치하도록 구현해야 한다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 오류를 표시하는 시점도 설계한다

필수 입력이 비어 있다는 이유만으로 페이지가 열리자마자 모두 오류 처리하면, 사용자는 아직 시작하지도 않은 작업에서 실패한 것처럼 느낄 수 있다.

어떤 시점에 검사하고 안내할지 정하고, 오류가 수정되면 오래된 메시지와 상태도 함께 갱신한다. 오류 메시지는 문제가 있는 항목을 식별하고 해결 방법을 알려야 한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 완료 상태도 알려 준다

제출 버튼을 누른 뒤 아무 변화가 없으면 사용자는 작업이 끝났는지 알기 어렵다.

성공과 실패 모두 명확한 피드백이 필요하다. 다만 “브라우저 검증 통과”, “요청 전송”, “서버 저장 성공”은 서로 다른 상태이므로 실제로 확인된 결과만 안내해야 한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

> 🔭 **미리보기: 동적 상태 알림**
>
> 페이지 이동 없이 결과가 갱신될 때는 화면 변화가 보조 기술에도 전달되도록 별도 알림 설계가 필요할 수 있다. 구체적인 구현은 이후 DOM / Event 학습을 거쳐 동적인 UI를 구현할 때 확장한다.

### 이름·설명·상태를 한 입력에 연결하기

```html
<label for="account-email">이메일 (필수)</label>
<p id="email-hint">신청 결과를 받을 주소입니다.</p>
<input id="account-email" name="email" type="email" required
       autocomplete="email" aria-describedby="email-hint email-error">
<p id="email-error"></p>
```

레이블은 “무엇을 입력하는가”를, 힌트는 “어떻게 입력하고 어디에 쓰는가”를 설명한다. 오류가 생기면 email-error에 원인과 수정 방법을 넣고 해당 입력에 aria-invalid를 지정하는 흐름을 설계할 수 있다. 아직 오류가 없는 필드까지 처음부터 invalid로 표시하지 않는다.

aria-describedby에 연결한 텍스트가 있다고 값 자체의 검증이 실행되지는 않는다. 반대로 HTML 검증이 있다고 제품이 원하는 오류 안내가 모두 전달되는 것도 아니다. 브라우저 검증, 사용자에게 보이는 설명, 보조 기술에 연결되는 상태를 함께 확인한다.

### 포커스와 알림을 중복시키지 않기

포커스는 현재 키보드 조작의 위치다. aria-live나 role=status는 포커스를 옮기지 않고 상태 변화를 알리는 수단이다. 오류 필드로 포커스를 옮길 때 같은 문장을 여러 live 영역에 동시에 출력하면 중복 안내가 생길 수 있다. 사용자가 수정해야 할 위치와 읽어야 할 안내를 구별하여 테스트한다.

기본 폼에서는 DOM 순서가 논리적인 Tab 순서가 되도록 구성하고 :focus-visible 등으로 현재 위치를 식별할 수 있게 한다. 단순히 outline을 지운 뒤 색상 변화만 남겨두지 않는다. 실제 키보드와 스크린 리더의 안내는 환경에 따라 달라지므로 지원 대상에서 검증한다.

---

## 8. 세 주제가 연결되는 제출 흐름

1. 사용자는 label·legend·설명을 읽고 입력 목적을 이해한다.
2. 브라우저의 네이티브 컨트롤을 키보드·마우스·보조 기술로 조작한다.
3. 현재 value·checked와 적용되는 제약에 따라 유효성이 결정된다.
4. 유효하지 않은 제출에는 원인·수정 방법과 수정 위치를 제공한다.
5. 제출 대상 컨트롤의 name·현재 값으로 데이터가 구성된다.
6. 요청의 성공·실패를 사용자가 인지할 수 있게 안내한다.

각 단계는 독립된 품질 조건이다. 예를 들어 화면의 이름이 있어도 name이 없으면 전송 데이터가 빠질 수 있고, 서버에 데이터가 도착해도 레이블이나 포커스가 없으면 일부 사용자는 제출까지 도달하지 못한다.

---

## 9. 코드 예제와 화면·상태·제출 변화 해설

### 예제 A. 검색 폼의 검증과 GET 데이터

다음 코드를 하나의 HTML 파일로 저장해 브라우저에서 연다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>문서 검색</title>
</head>
<body>
  <main>
    <h1>문서 검색</h1>

    <form method="get">
      <p>
        <label for="query">검색어 (필수)</label>
        <input
          id="query"
          name="q"
          type="search"
          required
        >
      </p>

      <fieldset>
        <legend>문서 종류</legend>

        <label>
          <input
            type="radio"
            name="kind"
            value="all"
            checked
          >
          전체
        </label>

        <label>
          <input
            type="radio"
            name="kind"
            value="guide"
          >
          가이드
        </label>
      </fieldset>

      <p>
        <label>
          <input
            type="checkbox"
            name="archived"
            value="yes"
          >
          보관된 문서 포함
        </label>
      </p>

      <button type="submit">검색</button>
    </form>
  </main>
</body>
</html>
```

#### 실행 전 구조

- `<form>`은 검색 조건을 하나의 작업으로 묶는다.
- `label`의 `for="query"`는 `id="query"`인 입력을 가리킨다.
- 두 라디오 버튼은 `name="kind"`가 같으므로 같은 선택 그룹에 속한다.
- `checked`가 있는 `전체`가 초기 선택값이다.
- `fieldset`과 `legend`는 라디오 그룹의 목적을 표현한다.[1](https://www.w3.org/WAI/tutorials/forms/)[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

#### 빈 상태로 제출하면

검색어에 `required`가 있으므로 브라우저가 제출을 막는다.

이때 JavaScript가 없어도 브라우저의 기본 검증이 동작한다. 오류 안내의 문구와 시각적 표현은 브라우저 및 언어 설정에 따라 다를 수 있다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

#### 값을 입력하고 제출하면

검색어로 `browser`를 입력하고 `가이드`를 선택하며 체크박스를 그대로 두면, 제출 조건은 다음 형태가 된다.

```txt
?q=browser&kind=guide
```

체크박스까지 선택하면 다음 항목이 추가된다.

```txt
&archived=yes
```

이 예제는 `action`을 생략했으므로 현재 문서 주소를 대상으로 제출한다. 실제 검색 서버가 없어도 주소 표시줄의 쿼리 문자열에서 데이터 구성 결과를 관찰할 수 있다. 다만 로컬 파일 실행과 웹 서버 실행의 세부 동작에는 차이가 있으므로, 제출 요청까지 확인하려면 로컬 개발 서버에서 실행한다.

제출은 입력창을 동적으로 결과 목록으로 바꾸는 동작이 아니다. 이 예제에서는 브라우저가 주소를 구성하고 문서 탐색을 수행한다.

### 예제 B. 입력의 현재 상태를 관찰하기

다음 코드를 HTML 파일 하나에 저장해 실행한다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>입력 상태 확인</title>
</head>
<body>
  <main>
    <h1>프로필 입력</h1>

    <p>
      <label for="nickname">별명</label>
      <input id="nickname" type="text" value="해원">
    </p>

    <p>
      현재 별명:
      <span id="nickname-preview">해원</span>
    </p>

    <p>
      <label>
        <input
          id="subscribe"
          type="checkbox"
          value="weekly"
        >
        주간 소식 받기
      </label>
    </p>

    <p>
      현재 수신 상태:
      <span id="subscribe-preview">받지 않음</span>
    </p>
  </main>

  <script>
    const nickname = document.getElementById("nickname");
    const nicknamePreview =
      document.getElementById("nickname-preview");

    const subscribe = document.getElementById("subscribe");
    const subscribePreview =
      document.getElementById("subscribe-preview");

    nickname.addEventListener("input", () => {
      nicknamePreview.textContent = nickname.value;
    });

    subscribe.addEventListener("change", () => {
      subscribePreview.textContent =
        subscribe.checked ? "받음" : "받지 않음";
    });
  </script>
</body>
</html>
```

> 🔭 **미리보기: DOM 조작 / 이벤트**
>
> 이 예제의 JavaScript는 입력 요소를 찾아 사용자의 변경을 감지하고 다른 요소의 글자를 바꾼다. 요소 탐색, `textContent`, 이벤트 등록의 원리는 이후 DOM / DOM Manipulation과 Event 학습에서 자세히 다룬다.

#### 텍스트를 입력할 때

1. 사용자가 별명을 수정한다.
2. 브라우저가 입력 요소의 현재 `value`를 바꾼다.
3. 사용자 입력으로 `input` 이벤트가 발생한다.
4. JavaScript가 현재 `value`를 읽는다.
5. 미리보기 요소의 텍스트가 변경된다.

`input` 이벤트는 사용자 동작으로 입력값이 바뀔 때 발생한다. 텍스트 입력에서 `change`는 일반적으로 값을 바꾼 뒤 포커스를 떠나는 등 변경을 확정하는 시점에 발생하므로, 입력 중 즉시 반영과 구분해야 한다.[12](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)

#### 체크박스를 누를 때

브라우저는 현재 `checked` 상태를 바꾼다. 코드는 그 참·거짓 상태에 따라 미리보기 문구를 변경한다.

`subscribe.value`는 여전히 `"weekly"`이므로 이를 조건으로 사용하면 선택 여부를 올바르게 판단할 수 없다.[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

#### 코드로 값을 바꾸면

개발자 도구 Console에서 다음과 같이 실행할 수 있다.

```javascript
document.getElementById("nickname").value = "새 별명";
```

입력창의 값은 바뀌지만, 이것만으로 사용자 입력과 같은 `input` 이벤트가 자동 발생하지는 않는다. 따라서 예제의 미리보기는 그대로일 수 있다.[12](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)

입력 상태가 바뀌는 것과, 변경을 알리는 이벤트가 발생하는 것은 구분해야 한다.

### 예제 C. 접근 가능한 가입 폼의 구조와 피드백

다음 예제는 서버 전송 없이 브라우저에서 입력 검증과 키보드 사용을 확인하기 위한 페이지다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>문의 작성</title>

  <style>
    body {
      max-width: 40rem;
      margin: 2rem auto;
      padding: 0 1rem;
      color: #111;
      background: #fff;
      font-family: sans-serif;
      line-height: 1.6;
    }

    input,
    textarea,
    button {
      font: inherit;
    }

    input[type="email"],
    textarea {
      box-sizing: border-box;
      width: 100%;
      padding: 0.5rem;
    }

    fieldset {
      margin-block: 1rem;
    }

    :focus-visible {
      outline: 3px solid #005fcc;
      outline-offset: 3px;
    }
  </style>
</head>
<body>
  <main>
    <h1>문의 작성</h1>

    <p>이메일과 문의 내용은 필수 항목입니다.</p>

    <form method="get">
      <p>
        <label for="email">이메일 (필수)</label>
      </p>

      <p id="email-help">
        답변을 받을 수 있는 이메일 주소를 입력하세요.
      </p>

      <input
        id="email"
        name="email"
        type="email"
        autocomplete="email"
        aria-describedby="email-help"
        required
      >

      <fieldset>
        <legend>문의 종류</legend>

        <label>
          <input
            type="radio"
            name="category"
            value="general"
            checked
          >
          일반 문의
        </label>

        <label>
          <input
            type="radio"
            name="category"
            value="account"
          >
          계정 문의
        </label>
      </fieldset>

      <p>
        <label for="message">문의 내용 (필수)</label>
      </p>

      <textarea
        id="message"
        name="message"
        rows="6"
        required
      ></textarea>

      <p>
        <button type="submit">입력 확인</button>
      </p>
    </form>
  </main>
</body>
</html>
```

실제 개인정보를 입력하지 말고 테스트용 값으로 실행한다. GET 방식은 입력을 URL에 포함하므로 실제 문의 전송 구현에 그대로 사용하지 않는다.

#### HTML이 제공하는 의미

- `<main>`과 `<h1>`이 본문의 위치와 제목을 표현한다.
- `label`이 이메일과 문의 내용의 이름을 제공한다.
- `fieldset`과 `legend`가 라디오 버튼의 공통 질문을 표현한다.
- `required`가 브라우저에 필수 조건을 전달한다.
- `aria-describedby`가 이메일 입력과 도움말을 연결한다.[1](https://www.w3.org/WAI/tutorials/forms/)[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

#### 화면과 상태의 변화

사용자가 Tab으로 이동하면 브라우저가 포커스를 이동하고, CSS가 포커스 위치를 표시한다.

라디오 버튼을 선택하면 브라우저가 선택 상태를 바꾼다. 별도의 JavaScript로 체크 모양만 바꾸는 방식이 아니라 실제 입력 상태가 바뀐다.

필수 값이 없거나 이메일 형식이 맞지 않으면 기본 검증이 제출을 막는다. 이 예제는 자체 오류 UI를 구현하지 않으므로 구체적인 메시지는 브라우저가 제공한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

#### CSS의 역할

CSS는 포커스가 어디에 있는지 눈으로 확인할 수 있게 만든다. 하지만 CSS가 레이블 연결이나 입력 검증을 대신하지는 않는다.

또한 예제의 포커스 스타일을 복사했다고 모든 배경과 배치에서 접근성 기준을 충족한다고 단정해서는 안 된다. 실제 주변 색상, 잘림, 가림 여부를 함께 확인해야 한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)[14](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance)

---

## 10. 흔한 오해·예외·호환성

### “ARIA를 많이 붙이면 접근성이 좋아진다”

불필요하거나 잘못된 ARIA는 실제 UI와 다른 정보를 전달할 수 있다. 기본 HTML로 표현하고, 부족한 의미나 관계만 보완한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### “입력 옆에 텍스트가 있으니 레이블이 있다”

시각적 근접성과 프로그램상 연결은 다르다.

```html
<p>이메일</p>
<input type="email">
```

이 구조만으로 앞의 문장이 입력의 레이블이 되지는 않는다. `label`과 `for`·`id` 연결을 사용한다.[1](https://www.w3.org/WAI/tutorials/forms/)

### “포커스 테두리는 보기 싫으니 없앤다”

대체 표시 없이 포커스 표시를 제거하면 키보드 사용자가 현재 위치를 파악하기 어렵다. 디자인을 변경할 수는 있지만, 명확하게 보이는 표시를 유지해야 한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)[14](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance)

### “CSS 배치를 바꾸면 키보드 순서도 같이 바뀐다”

화면의 시각적 순서와 키보드 이동 순서가 어긋날 수 있다. 우선 문서 자체를 논리적인 순서로 작성하고, 화면 배치가 그 순서를 혼란스럽게 만들지 않는지 확인한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### “자동 검사에서 오류가 없으면 접근성이 완성됐다”

자동 검사는 유용하지만, 레이블이 실제 목적에 맞는지, 키보드로 작업을 완료할 수 있는지, 오류 메시지가 이해되는지까지 모두 판단하지는 못한다. 실제 조작 테스트를 함께 진행한다.[1](https://www.w3.org/WAI/tutorials/forms/)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### “`type="email"`이면 실제 이메일 주소임이 보장된다”

기본 검증은 형식을 검사한다. 주소의 존재나 소유 여부까지 확인하는 것은 아니다.[10](https://developer.mozilla.org/docs/Web/HTML/Guides/Constraint_validation)

### “숫자 입력의 `value`는 숫자다”

DOM의 `value`는 문자열이다. 수치 연산이 필요하면 타입에 맞는 수치 해석과 빈 값 처리를 별도로 고려해야 한다.[11](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)

### “입력 UI는 모든 브라우저에서 같다”

날짜 선택기, 숫자 입력의 증감 버튼, 가상 키보드 등은 브라우저와 운영체제에 따라 달라질 수 있다. 실제 대상 환경에서 사용성과 입력 가능성을 확인한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### “입력할 때마다 즉시 오류를 표시하는 것이 가장 좋다”

사용자가 아직 입력을 마치지 않은 상태를 계속 오류로 알리면 방해가 될 수 있다. 제출 시점이나 적절한 입력 완료 시점을 고려하고, 오류는 수정 가능한 문장으로 제공한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

---

## 11. 실무 적용·접근성·성능·디버깅

### 키보드만으로 검사하기

마우스를 사용하지 않고 다음을 수행한다.

1. Tab과 Shift+Tab으로 필요한 컨트롤에 접근한다.
2. 현재 포커스 위치를 눈으로 확인한다.
3. 입력값을 수정한다.
4. 체크박스와 라디오 버튼을 조작한다.
5. 제출을 시도한다.
6. 오류를 찾고 수정한 뒤 다시 제출한다.

라디오 그룹처럼 여러 요소가 하나의 키보드 조작 단위를 이루는 경우가 있으므로, 모든 항목에 Tab이 한 번씩 멈추는지만으로 판단하지 않는다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 개발자 도구로 검사하기

브라우저의 Accessibility 관련 패널에서 다음을 확인한다.

- 입력에 이름이 있는가?
- 이름이 화면의 레이블과 일치하는가?
- 역할이 의도에 맞는가?
- 도움말이 설명으로 연결되어 있는가?
- 필수·선택·오류 상태가 실제 화면과 일치하는가?

도구의 패널 이름과 표시 방식은 브라우저마다 다를 수 있다.

### 스크린 리더로 검사하기

가능하다면 운영체제에서 제공하는 스크린 리더 등으로 실제 사용 흐름을 확인한다.

음성으로 들리는 문장을 외우기보다 다음을 확인한다.

- 입력 목적을 이해할 수 있는가?
- 그룹의 질문을 이해할 수 있는가?
- 현재 선택 상태를 알 수 있는가?
- 오류를 찾고 수정할 수 있는가?
- 작업 결과를 알 수 있는가?

### 유지보수와 성능 관점

기본 HTML 컨트롤을 사용하면 키보드 조작과 상태 관리를 직접 구현해야 하는 범위를 줄일 수 있다. 커스텀 컨트롤은 외형뿐 아니라 기본 컨트롤이 제공하던 동작과 정보를 재현해야 한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

접근성 개선을 불필요한 애니메이션이나 잦은 알림 추가로 오해하지 않는다. 현재 위치, 입력 조건, 오류, 완료 결과처럼 작업에 필요한 정보를 적절한 시점에 제공하는 것이 우선이다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

---

### 접근성 점검

- 모든 입력에 목적을 알 수 있는 레이블이 있는가?
- 관련 입력을 그룹으로 묶었는가?
- 필수 조건을 화면에도 안내하는가?
- 오류가 발생하면 어느 항목을 어떻게 고쳐야 하는지 알 수 있는가?
- 제출 성공과 실패를 모두 명확하게 안내하는가?

오류 메시지는 단순히 “오류”라고 쓰는 대신 “이메일에 `@`를 포함해 주세요”처럼 수정 방법을 알려야 한다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 성능과 유지보수

기본 검증으로 충분한 조건까지 모두 JavaScript로 다시 작성하면 구현량과 테스트 범위가 늘어난다. 우선 HTML로 표현 가능한 조건을 사용하고, 실제 요구사항이 더 복잡할 때 확장한다.[10](https://developer.mozilla.org/docs/Web/HTML/Guides/Constraint_validation)

### 개발자 도구로 확인하기

1. Elements에서 입력에 `name`이 있는지 확인한다.
2. 레이블의 `for`와 입력의 `id`가 일치하는지 확인한다.
3. 빈 값과 잘못된 값으로 제출해 본다.
4. Network에서 제출 주소, 요청 방식, 전달 데이터를 확인한다.
5. 주소 표시줄에서 GET 쿼리 문자열을 확인한다.
6. 마우스 없이 입력과 제출이 가능한지 확인한다.

---

## 12. 검증 API가 검사하는 조건과 남는 책임

### 같은 문자열이어도 사용자 편집과 스크립트 대입의 검사가 다를 수 있다

`minlength`·`maxlength`가 모든 값 대입을 막는 범용 데이터 스키마라고 생각하면 오류가 생긴다. HTML의 길이 제약은 사용자가 편집한 값에 적용되는 조건을 포함한다. JavaScript로 `.value`를 대입한 뒤 `checkValidity()`를 호출해도 사용자 편집과 같은 길이 실패가 나타난다고 가정할 수 없다. [HTML의 최소 길이 검증 조건](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fe-minlength)과 [MDN 제약 검증의 주의점](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation#constraint_validation_process)을 함께 확인한다.

다음 코드를 별도 HTML 파일로 열어 비교한다. 실행 결과를 보기 위한 실험이며 서버 제출은 없다.

```html
<!doctype html>
<html lang="ko">
<meta charset="utf-8">
<title>길이 제약과 값 변경 경로</title>
<main>
  <h1>입력 길이 실험</h1>
  <label for="title">제목: 입력할 때 4~8자</label>
  <input id="title" minlength="4" maxlength="8"
    aria-describedby="help">
  <p id="help">먼저 스크립트로 값을 넣고, 다음에는 직접 두 글자를 입력합니다.</p>
  <button id="assign" type="button">스크립트로 ab 대입</button>
  <button id="inspect" type="button">검사 결과 보기</button>
  <pre id="result" role="status"></pre>
</main>
<script>
  const titleInput = document.querySelector('#title');
  const result = document.querySelector('#result');

  function inspect() {
    result.textContent = JSON.stringify({
      value: titleInput.value,
      tooShort: titleInput.validity.tooShort,
      valid: titleInput.checkValidity(),
    }, null, 2);
  }

  document.querySelector('#assign').addEventListener('click', () => {
    titleInput.value = 'ab';
    inspect();
  });
  document.querySelector('#inspect').addEventListener('click', inspect);
</script>
</html>
```

‘스크립트로 ab 대입’을 누르면 value는 ab지만 tooShort는 false이고 valid는 true다. 그다음 입력 내용을 모두 선택해 지우고 키보드로 ab를 직접 입력한 뒤 검사하면 tooShort는 true이고 valid는 false다. 문자열은 같지만 마지막 변경 경로가 다르다. 반대로 이 예제의 빈 값은 required가 없으므로 minlength만으로 필수 실패가 되지 않는다.

이는 클라이언트 검증을 신뢰 경계로 삼으면 안 되는 구체적인 이유이기도 하다. 제출 데이터는 브라우저 UI를 거치지 않고도 만들 수 있다. 서버는 실제 데이터의 길이·형식·허용 범위를 자체 검사해야 하며, 클라이언트 검증은 사용자에게 빨리 오류를 알려 주는 역할을 맡는다.

### customValidity의 오류는 스스로 지워지지 않는다

`setCustomValidity(message)`는 그 입력에 사용자 정의 검증 오류 메시지를 남긴다. 빈 문자열이 아닌 메시지가 남아 있으면 다른 조건이 맞아도 검증에 실패한다. 문제가 해결되면 애플리케이션이 `setCustomValidity('')`로 해제해야 한다. [HTML 검증 API](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#dom-cva-setcustomvalidity)와 [MDN setCustomValidity](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setCustomValidity)의 규칙이다.

다음은 브라우저 콘솔에서 독립적으로 실행한다. DOM에 연결하지 않은 텍스트 입력 객체를 검사하므로 오류 팝업·포커스 이동을 실험하는 코드는 아니다.

```javascript
{
  const input = document.createElement('input');
  input.required = true;
  input.value = 'HTML';
  input.setCustomValidity('중복된 제목입니다.');
  console.log(input.checkValidity(), input.validity.customError);

  input.value = '다른 제목';
  console.log(input.checkValidity(), input.validity.customError);

  input.setCustomValidity('');
  console.log(input.checkValidity(), input.validity.customError);
}
```

결과는 false·true, false·true, true·false다. 처음에는 명시한 사용자 정의 오류 때문에 실패한다. 두 번째는 문자열만 바꿨으므로 오류 메시지가 그대로 남는다. 세 번째에 메시지를 비워야 해당 실패가 해제된다. 여기서는 required도 이미 만족하므로 최종 검증이 통과한다. 다른 기본 제약까지 어긴 경우에는 사용자 정의 오류를 해제해도 전체 검증은 여전히 실패할 수 있다.

### 값은 고쳤는데 제출이 계속 막히는 문제를 찾는다

증상이 생기면 현재 value만 확인하지 말고 `validity`의 실패 종류와 `validationMessage`를 함께 본다. 위 코드를 재현했을 때 value가 바뀌어도 customError가 true라면 오래된 사용자 정의 오류가 남았다는 근거다. 검증 함수가 정상 경로에서도 먼저 메시지를 비우거나 조건에 맞는 메시지를 새로 설정하도록 수정한다. 다시 잘못된 값 → 올바른 값 → 빈 값 순서로 편집하여 사용자 정의 조건과 기본 required 조건이 각각 해제·발생하는지 확인한다.

사용자에게 보여 주는 오류도 같은 결과와 연결한다. 오류 메시지만 없애고 검증 오류는 남겨 두거나, 검증 오류만 해제하고 화면에는 예전 오류를 남기지 않는다. 입력과 오류를 연결하는 설명·상태, 키보드로 오류를 수정할 수 있는 경로까지 함께 점검한다.

---

## 13. 후속 학습 미리보기

\[미리보기: preventDefault / stopPropagation\]

직접 제출을 처리할 때 기본 동작 취소와 이벤트 전파 중단을 구별한다. 두 흐름의 제어는 1–3주 preventDefault / stopPropagation에서 다룬다.

\[미리보기: Controlled / Uncontrolled Component\]

React가 현재 입력값을 지정하는 방식과 브라우저가 유지하는 값을 읽는 방식은 소유자가 다르다. 같은 모듈 React Basics 마지막 페이지에서 연결한다.

---

## 14. 핵심 정리

<callout icon="✅" color="green_bg">
폼은 입력을 이해하는 단계에서 시작하여 검증과 제출로 끝난다. `label`·그룹·입력 타입은 사용을 돕고, `name`과 현재 값은 제출 데이터를 결정한다. 접근성은 이 모든 단계를 키보드와 보조 기술로도 수행할 수 있게 만드는 조건이다.
</callout>

---

## 출처

- [WHATWG — Forms](https://html.spec.whatwg.org/multipage/forms.html)
- [WHATWG — Input](https://html.spec.whatwg.org/multipage/input.html)
- [WHATWG — 폼 제출과 제약 검증](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html)
- [W3C WAI — Forms Tutorial](https://www.w3.org/WAI/tutorials/forms/)
- [W3C — Accessible Name and Description Computation](https://www.w3.org/TR/accname-1.2/)
- [W3C WAI — User Notifications](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 이번 설명의 근거와 적용 범위

- [HTML — 최소·최대 길이](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fe-minlength): 사용자 편집 여부·빈 값·길이에 따른 검증 조건을 확인했다. [MDN Constraint Validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation)은 프로그램 대입 시 길이 검사가 다른 점을 보충한다.
- [HTML — 제약 검증 API](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#dom-cva-setcustomvalidity): customValidity 메시지가 빈 문자열인지에 따른 사용자 정의 오류 상태를 확인했다.
- [MDN — setCustomValidity](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setCustomValidity): 오류 해제 시 빈 문자열을 명시해야 하는 사용 규칙을 확인했다.

---

## 15. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 입력에 `id="email"`만 있고 `name`이 없다. 사용자는 입력했지만 서버에는 값이 없다. 어느 단계의 문제인가?
2. 체크하지 않은 체크박스, disabled 입력, readonly 텍스트 입력의 폼 데이터 포함 여부를 비교하라.
3. placeholder와 빨간 테두리만 있는 오류 폼을 키보드·보조 기술 관점에서 어떻게 개선하겠는가?
4. 폼의 “도움말” 버튼을 누르면 페이지가 이동한다. 버튼 타입과 제출 흐름을 근거로 원인과 수정 방법을 설명하라.
