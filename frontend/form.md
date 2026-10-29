---
track: B
notion: https://app.notion.com/p/406178c4afac80d09c38e02f96e103e3
---

# Form

> 📍 **학습 범위**: Web Platform · Form / Input / Accessibility (1/3)
>
> 사용자의 입력을 제출 가능한 데이터로 묶기
>
> 입력 요소의 종류와 상태: [Input](input.md) · 폼 접근성: [Accessibility 기초](accessibility-basics.md)

- [1. 학습 목표](#1-학습-목표)
- [2. 폼을 왜 배우는가](#2-폼을-왜-배우는가)
- [3. 핵심 개념과 용어](#3-핵심-개념과-용어)
- [4. 관련 계층과 역할 구분](#4-관련-계층과-역할-구분)
- [5. 브라우저의 기본 제출 흐름](#5-브라우저의-기본-제출-흐름)
- [6. 최소 실행 예제: 검색 조건 제출하기](#6-최소-실행-예제-검색-조건-제출하기)
- [7. 흔한 오해와 주의점](#7-흔한-오해와-주의점)
- [8. 실무 적용·접근성·성능·디버깅](#8-실무-적용접근성성능디버깅)
- [9. 후속 학습 연결](#9-후속-학습-연결)
- [10. 핵심 정리](#10-핵심-정리)
- [11. 이해도 확인 질문](#11-이해도-확인-질문)

## 1. 학습 목표

이 글을 학습한 뒤에는 다음을 설명하고 구현할 수 있어야 한다.

- `<form>`이 단순한 화면 묶음이 아니라 입력 제출을 위한 구조임을 설명한다.
- `action`, `method`, `name`, `value`의 역할을 구분한다.
- 제출 버튼을 누른 뒤 브라우저가 검증하고 제출하는 흐름을 설명한다.
- 제출되는 입력과 제출되지 않는 입력을 구분한다.
- 접근성 있는 폼을 만들고 브라우저 개발자 도구로 제출 결과를 확인한다.

---

## 2. 폼을 왜 배우는가

검색창, 로그인 화면, 회원가입 화면, 배송지 입력 화면에는 공통점이 있다. 사용자가 여러 값을 입력하고, 하나의 작업으로 제출한다는 점이다.

화면에 입력창과 버튼만 배치하면 이 작업이 완성되는 것은 아니다.

- 어떤 입력들이 같은 작업에 속하는가?
- 각 입력값은 어떤 이름으로 전달되는가?
- 필수 항목이 비어 있으면 어떻게 할 것인가?
- 마우스 없이도 제출할 수 있는가?
- 제출 후 어느 주소에서 처리하는가?

HTML의 `<form>`은 이러한 입력과 제출의 관계를 표현한다. 브라우저가 제공하는 폼 기능을 사용하면 검증, 제출, 키보드 사용을 처음부터 모두 직접 구현하지 않아도 된다. 접근성 있는 폼에서는 여기에 명확한 레이블, 안내, 오류 피드백을 함께 제공한다.[1](https://www.w3.org/WAI/tutorials/forms/)[2](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)

## 3. 핵심 개념과 용어

### 3.1 폼과 폼 컨트롤

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

### 3.2 `action`: 어디로 제출하는가

```html
<form action="/search">
  <!-- 입력 요소 -->
</form>
```

`action`은 폼을 처리할 주소다.

`/search`처럼 `/`로 시작하는 경로는 현재 사이트의 루트를 기준으로 해석한다. 실제 검색 결과를 제공하려면 해당 주소에서 요청을 처리하는 기능이 있어야 한다.

HTML만 작성했다고 검색 서비스나 데이터 저장 기능까지 생기는 것은 아니다.

### 3.3 `method`: 어떤 방식으로 제출하는가

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

### 3.4 `name`과 `value`: 어떤 데이터로 제출하는가

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

## 5. 브라우저의 기본 제출 흐름

일반적인 네이티브 폼 제출은 다음과 같이 진행된다.

1. 사용자가 제출 버튼을 활성화한다.
2. 브라우저가 `required`, 입력 타입 등의 제약을 검사한다.
3. 유효하지 않은 입력이 있으면 제출을 막고 문제를 안내한다.
4. 검증을 통과하면 폼에서 `submit` 이벤트가 발생한다.
5. 제출이 취소되지 않았다면 제출 데이터를 구성한다.
6. `action`과 `method`에 따라 요청하고, 일반적인 페이지 폼에서는 응답으로 이동한다.

> 💡 유효성 검사를 통과하지 못한 제출 시도에서는 `submit` 이벤트도 발생하지 않는다는 점이 중요하다.[2](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)

### 5.1 버튼의 종류를 명시해야 하는 이유

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

### 5.2 제출 데이터에 포함되지 않는 경우

대표적으로 다음 입력은 일반적인 제출 데이터에서 제외된다.

- `name`이 없는 입력
- 비활성화된 `disabled` 입력
- 선택되지 않은 체크박스와 라디오 버튼

체크박스는 선택되었을 때만 자신의 `name`과 `value`를 제출한다. 선택하지 않았다고 자동으로 `false`가 제출되는 것은 아니다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

## 6. 최소 실행 예제: 검색 조건 제출하기

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

### 6.1 실행 전 구조

- `<form>`은 검색 조건을 하나의 작업으로 묶는다.
- `label`의 `for="query"`는 `id="query"`인 입력을 가리킨다.
- 두 라디오 버튼은 `name="kind"`가 같으므로 같은 선택 그룹에 속한다.
- `checked`가 있는 `전체`가 초기 선택값이다.
- `fieldset`과 `legend`는 라디오 그룹의 목적을 표현한다.[1](https://www.w3.org/WAI/tutorials/forms/)[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### 6.2 빈 상태로 제출하면

검색어에 `required`가 있으므로 브라우저가 제출을 막는다.

이때 JavaScript가 없어도 브라우저의 기본 검증이 동작한다. 오류 안내의 문구와 시각적 표현은 브라우저 및 언어 설정에 따라 다를 수 있다.[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

### 6.3 값을 입력하고 제출하면

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

## 7. 흔한 오해와 주의점

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

## 8. 실무 적용·접근성·성능·디버깅

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

## 9. 후속 학습 연결

> 🔭 **미리보기: submit 이벤트 / preventDefault**
>
> JavaScript는 폼 제출을 감지하고 브라우저의 기본 제출을 취소할 수 있다. 이벤트의 전달 과정과 기본 동작 취소는 이후 Web Platform의 Event 및 `preventDefault / stopPropagation` 학습에서 자세히 다룬다.

---

## 10. 핵심 정리

> ✅ - 폼은 여러 입력을 하나의 제출 작업으로 묶는다.

- `action`은 목적지, `method`는 제출 방식, `name`은 데이터의 키다.
- 브라우저는 기본 제출 전에 HTML 제약을 검증한다.
- 화면에 보이는 모든 입력이 제출되는 것은 아니다.
- 좋은 폼은 입력뿐 아니라 레이블, 그룹, 오류, 제출 결과까지 설계한다.

---

## 11. 이해도 확인 질문

> 📝 답을 정리하며 개념을 점검해 보세요. 코드가 있는 문항은 실행 결과를 먼저 예측한 뒤 직접 확인해 보세요.

1. 검색어 입력에 `id`와 `label`은 있지만 `name`이 없다. 사용자는 정상적으로 입력하고 제출할 수 있다. 제출 데이터에는 어떤 문제가 생기며, 어떤 도구로 확인하겠는가?
2. 검색 폼 안에 `<button>검색 도움말</button>`을 추가했더니 도움말을 누를 때 검색이 실행된다. 원인을 분석하고 HTML을 어떻게 수정할지 설명하라.
3. 필수 입력이 비어 있을 때 폼의 `submit` 처리 코드가 실행되지 않는다. 브라우저의 검증 순서를 기준으로 원인을 설명하라.
4. 선택하지 않은 체크박스를 서버가 항상 `false`로 받을 것이라고 가정했다. 이 설계의 문제와 제출 데이터에서 확인해야 할 점은 무엇인가?
5. 검색 기능과 회원가입 기능에 각각 어떤 제출 방식을 선택하겠는가? URL 노출, 작업의 성격, 보안의 한계를 구분하여 설명하라.

---

<details>
<summary>출처 · 첨부 원문 전체 링크 (30개)</summary>

- [1. Forms Tutorial \| Web Accessibility Initiative (WAI) \| W3C](https://www.w3.org/WAI/tutorials/forms/)
- [2. HTMLFormElement: submit event - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/submit_event)
- [3. \&lt;input&gt; HTML input element - HTML \| MDN - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)
- [4. Client-side form validation - Learn web development \| MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)
- [5. HTMLInputElement: defaultValue property - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultValue)
- [6. HTMLInputElement: defaultChecked property - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultChecked)
- [7. \&lt;input type=&quot;checkbox&quot;&gt; HTML attribute value - HTML \| MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)
- [8. HTMLFormElement: requestSubmit() method - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/requestSubmit)
- [9. User Notification \| Web Accessibility Initiative (WAI) \| W3C](https://www.w3.org/WAI/tutorials/forms/notifications/)
- [10. Using HTML form validation and the Constraint Validation API](https://developer.mozilla.org/docs/Web/HTML/Guides/Constraint_validation)
- [11. HTMLInputElement - Web APIs \| MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)
- [12. Element: input event - Web APIs \| MDN - MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)
- [13. Developing a Keyboard Interface \| APG \| WAI \| W3C](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- [14. Understanding Success Criterion 2.4.13: Focus Appearance ...](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance)
- [15. Web Accessibility Initiative (WAI) W3C](https://w3c.github.io/wai-website-components/components/preview/example-home.html)
- [16. JavaScript Form Validation - W3Schools](https://www.w3schools.com/js/js_validation.asp)
- [17. Forms Accessibility Tutorials \| w3c/wai-tutorials \| DeepWiki](https://deepwiki.com/w3c/wai-tutorials/3.1-forms-accessibility-tutorials)
- [18. Accessibility Labels - W3Schools](https://www.w3schools.com/accessibility/accessibility_labels.php)
- [19. Accessibility Forms Introduction - W3Schools](https://www.w3schools.com/accessibility/accessibility_forms_intro.php)
- [20. ARIA Test Plan - Protocols and Formats Working Group Wiki - W3C](https://www.w3.org/WAI/PF/wiki/ARIA_Test_Plan)
- [21. Web Content Accessibility Guidelines Working Group Teleconference](https://www.w3.org/2013/07/30-wai-wcag-minutes.html)
- [22. Passed Example 8](https://www.w3.org/WAI/content-assets/wcag-act-rules/testcases/c487ae/e277de30edb9e550d8f9d5a72e1e3adde961d01d.html)
- [23. ARIA13: Using aria-labelledby to name regions and landmarks \| WAI](https://waic.jp/translations/WCAG22/Techniques/aria/ARIA13)
- [24. HTML & CSS Accessibility - W3Schools](https://www.w3schools.com/htmlcss/htmlcss_accessibility.asp)
- [25. Question: Why is WCAG 4.1.2 "name" treated as accessible ... - GitHub](https://github.com/w3c/wcag/discussions/4957)
- [26. How to Submit a Form with JavaScript in 2026 \| Static Forms](https://www.staticforms.dev/blog/submit-a-form-with-javascript)
- [27. Confirm attributes prefixed with aria- are recognized and are valid ...](https://www.accessibilitychecker.org/ace/engine/ace-aria-valid-attr/)
- [28. WCAG 2.4.13: Focus Appearance](https://www.wcag.com/designers/2-4-13-focus-appearance/)
- [29. WAI-ARIA에 관하여](https://yunjeoming.dev/blog/wai-aria)
- [30. html - Disabled form inputs do not appear in the request ...](https://stackoverflow.com/questions/7357256/disabled-form-inputs-do-not-appear-in-the-request)

</details>
