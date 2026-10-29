---
track: B
notion: https://app.notion.com/p/406178c4afac802992b4c44f89f0d581
---

# Input

> 📍 **학습 범위**: Web Platform · Form / Input / Accessibility (2/3)
>
> 입력 요소의 종류와 현재 상태 이해하기
>
> 입력의 제출 흐름: [Form](form.md) · 레이블과 키보드 접근성: [Accessibility 기초](accessibility-basics.md)

- [1. 학습 목표](#1-학습-목표)
- [2. 입력 요소를 왜 깊게 배우는가](#2-입력-요소를-왜-깊게-배우는가)
- [3. 핵심 개념과 용어](#3-핵심-개념과-용어)
- [4. 관련 계층과 역할 구분](#4-관련-계층과-역할-구분)
- [5. 초기값과 현재 값](#5-초기값과-현재-값)
- [6. 코드 예제: 현재 상태를 화면에 표시하기](#6-코드-예제-현재-상태를-화면에-표시하기)
- [7. 기본 검증과 입력 제한](#7-기본-검증과-입력-제한)
- [8. 흔한 오해·호환성·실무 디버깅](#8-흔한-오해호환성실무-디버깅)
- [9. 후속 학습 연결](#9-후속-학습-연결)
- [10. 핵심 정리](#10-핵심-정리)
- [11. 이해도 확인 질문](#11-이해도-확인-질문)

## 1. 학습 목표

이 글을 학습한 뒤에는 다음을 할 수 있어야 한다.

- 데이터의 의미에 맞는 입력 타입을 선택한다.
- `label`, `placeholder`, `autocomplete`, `inputmode`의 역할을 구분한다.
- HTML 초기값과 사용자가 변경한 현재 값을 구분한다.
- 텍스트 입력의 `value`와 체크박스의 `checked`를 올바르게 읽는다.
- 기본 검증 조건과 입력 상태를 브라우저에서 확인한다.

---

## 2. 입력 요소를 왜 깊게 배우는가

`<input>`은 빈칸 모양을 그리는 태그가 아니다.

브라우저는 입력 타입을 바탕으로 기본 UI, 값의 해석, 검증, 모바일 키보드 등에 관여한다. 예를 들어 이메일 입력과 체크박스는 모두 `<input>`이지만, 사용자가 조작하는 방식과 읽어야 하는 상태가 다르다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

따라서 입력 구현은 다음 세 질문에서 시작한다.

1. 어떤 의미의 데이터를 받는가?
2. 사용자가 어떤 방식으로 입력해야 하는가?
3. 현재 값을 어떤 상태로 읽고 검증해야 하는가?

## 3. 핵심 개념과 용어

### 3.1 입력 타입

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

### 3.2 숫자처럼 보이는 값이 모두 수량은 아니다

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

### 3.3 레이블과 placeholder

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

### 3.4 자동완성

`autocomplete`은 입력 정보의 목적을 브라우저에 알려 주는 힌트다.

```html
<input type="text" autocomplete="name">
<input type="email" autocomplete="email">
<input type="password" autocomplete="current-password">
<input type="password" autocomplete="new-password">
```

기존 비밀번호를 받는 로그인과 새로운 비밀번호를 만드는 가입은 목적이 다르다. 목적에 맞게 작성하면 브라우저와 비밀번호 관리 도구가 입력을 지원하는 데 도움이 된다. 실제 자동완성 동작은 브라우저와 사용자 설정에 따라 달라진다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

## 4. 관련 계층과 역할 구분

| 계층 | 담당하는 일 |
| --- | --- |
| HTML | 입력 종류, 초기값, 이름, 검증 조건을 선언한다. |
| 브라우저 | 현재 값, 선택 여부, 입력 UI와 기본 검증을 관리한다. |
| DOM | 실행 중인 입력 요소의 현재 상태를 읽고 변경하는 인터페이스를 제공한다. |
| JavaScript | DOM을 통해 값을 읽거나 화면의 다른 부분에 반영한다. |
| CSS | 포커스, 선택, 오류 등 상태의 시각적 표현을 담당한다. |

입력창에 글자를 입력했을 때 화면이 바뀐다고 HTML 파일 자체가 수정되는 것은 아니다. 브라우저가 실행 중인 입력 컨트롤의 상태를 바꾸는 것이다.[5](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/defaultValue)[11](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)

## 5. 초기값과 현재 값

### 5.1 텍스트 입력의 `value`

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

### 5.2 체크박스의 `checked`

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

## 6. 코드 예제: 현재 상태를 화면에 표시하기

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

### 6.1 텍스트를 입력할 때

1. 사용자가 별명을 수정한다.
2. 브라우저가 입력 요소의 현재 `value`를 바꾼다.
3. 사용자 입력으로 `input` 이벤트가 발생한다.
4. JavaScript가 현재 `value`를 읽는다.
5. 미리보기 요소의 텍스트가 변경된다.

`input` 이벤트는 사용자 동작으로 입력값이 바뀔 때 발생한다. 텍스트 입력에서 `change`는 일반적으로 값을 바꾼 뒤 포커스를 떠나는 등 변경을 확정하는 시점에 발생하므로, 입력 중 즉시 반영과 구분해야 한다.[12](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)

### 6.2 체크박스를 누를 때

브라우저는 현재 `checked` 상태를 바꾼다. 코드는 그 참·거짓 상태에 따라 미리보기 문구를 변경한다.

`subscribe.value`는 여전히 `"weekly"`이므로 이를 조건으로 사용하면 선택 여부를 올바르게 판단할 수 없다.[7](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/checkbox)

### 6.3 코드로 값을 바꾸면

개발자 도구 Console에서 다음과 같이 실행할 수 있다.

```javascript
document.getElementById("nickname").value = "새 별명";
```

입력창의 값은 바뀌지만, 이것만으로 사용자 입력과 같은 `input` 이벤트가 자동 발생하지는 않는다. 따라서 예제의 미리보기는 그대로일 수 있다.[12](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event)

입력 상태가 바뀌는 것과, 변경을 알리는 이벤트가 발생하는 것은 구분해야 한다.

## 7. 기본 검증과 입력 제한

### 7.1 자주 사용하는 제약

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

### 7.2 `disabled`와 `readonly`

| 구분 | `disabled` | `readonly` |
| --- | --- | --- |
| 사용자 수정 | 불가능 | 불가능 |
| 일반적인 Tab 접근 | 제외된다. | 가능하다. |
| 폼 제출 데이터 | 제외된다. | `name` 등이 갖춰지면 포함된다. |
| 적용 범위 | 여러 폼 컨트롤 | 지원하는 텍스트 계열 입력 등 |

`readonly`는 체크박스나 `select`를 읽기 전용으로 만드는 범용 속성이 아니다. 사용할 요소와 타입의 지원 여부를 확인해야 한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

두 속성 모두 보안 장치가 아니다. 클라이언트 값을 신뢰해 권한이나 결제 금액을 결정해서는 안 된다.

## 8. 흔한 오해·호환성·실무 디버깅

### “`type="email"`이면 실제 이메일 주소임이 보장된다”

기본 검증은 형식을 검사한다. 주소의 존재나 소유 여부까지 확인하는 것은 아니다.[10](https://developer.mozilla.org/docs/Web/HTML/Guides/Constraint_validation)

### “숫자 입력의 `value`는 숫자다”

DOM의 `value`는 문자열이다. 수치 연산이 필요하면 타입에 맞는 수치 해석과 빈 값 처리를 별도로 고려해야 한다.[11](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)

### “입력 UI는 모든 브라우저에서 같다”

날짜 선택기, 숫자 입력의 증감 버튼, 가상 키보드 등은 브라우저와 운영체제에 따라 달라질 수 있다. 실제 대상 환경에서 사용성과 입력 가능성을 확인한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)

### “입력할 때마다 즉시 오류를 표시하는 것이 가장 좋다”

사용자가 아직 입력을 마치지 않은 상태를 계속 오류로 알리면 방해가 될 수 있다. 제출 시점이나 적절한 입력 완료 시점을 고려하고, 오류는 수정 가능한 문장으로 제공한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 디버깅 순서

1. 데이터의 의미에 맞는 `type`인가?
2. `label`과 입력이 연결되어 있는가?
3. 폼 제출에 필요한 `name`이 있는가?
4. 초기값과 현재 값을 혼동하지 않았는가?
5. 체크박스를 `checked`로 읽고 있는가?
6. `disabled` 때문에 접근이나 제출에서 제외되지 않았는가?
7. 검증 속성이 해당 타입에 적용되는가?
8. 모바일과 키보드에서도 사용할 수 있는가?

### 성능 관점

매번 입력할 때마다 무거운 작업을 실행하면 타이핑 반응이 느려질 수 있다. 먼저 입력 처리 코드에서 어떤 작업이 반복되는지 확인하고, 단순 미리보기처럼 현재 필요한 일만 수행한다.

작은 예제의 텍스트 갱신까지 미리 복잡하게 최적화할 필요는 없다. 문제가 관찰되었을 때 원인을 측정하는 접근이 중요하다.

## 9. 후속 학습 연결

> 🔭 **미리보기: Controlled / Uncontrolled Component**
>
> React에서는 입력의 현재 값을 React 상태가 주도할지, DOM의 입력 상태를 읽어 사용할지 구분한다. 구체적인 구현과 선택 기준은 이후 React Basics의 Controlled / Uncontrolled Component에서 다룬다.

---

## 10. 핵심 정리

> ✅ - 입력 타입은 화면 모양이 아니라 데이터의 의미로 선택한다.

- 레이블은 이름, placeholder는 보조 힌트다.
- 텍스트의 현재 값은 `value`, 체크박스의 현재 선택 여부는 `checked`다.
- 초기값과 현재 상태는 다를 수 있다.
- 입력 제한, 형식 검증, 데이터의 실제 유효성은 서로 다른 문제다.

---

## 11. 이해도 확인 질문

> 📝 답을 정리하며 개념을 점검해 보세요. 코드가 있는 문항은 실행 결과를 먼저 예측한 뒤 직접 확인해 보세요.

1. 우편번호를 `type="number"`로 구현했다. 데이터의 의미와 입력 경험을 기준으로 이 선택을 검토하고 대안을 제시하라.
2. HTML에는 `value="해원"`이 있지만 사용자는 `다른 이름`으로 수정했다. 초기값과 현재 입력을 각각 확인하려면 무엇을 살펴봐야 하는가?
3. 체크박스를 선택 해제했는데 `input.value`는 계속 `"weekly"`다. 이것이 정상인 이유와 확인해야 할 현재 상태를 설명하라.
4. JavaScript로 입력의 `value`를 바꿨지만 `input` 이벤트 기반 미리보기가 갱신되지 않았다. 어떤 두 동작을 혼동한 것인가?
5. 수정은 막되 사용자가 값을 선택·복사하고 제출 데이터에도 포함해야 하는 텍스트 입력이 있다. 어떤 속성을 선택하겠으며, 그 선택을 다른 입력 타입에도 그대로 적용할 수 있는가?

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
