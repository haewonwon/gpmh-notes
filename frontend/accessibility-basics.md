---
track: B
notion: https://app.notion.com/p/406178c4afac80feadbed959c81b8d47
---

# Accessibility 기초

> 📍 **학습 범위**: Web Platform · Form / Input / Accessibility (3/3)
>
> 누구나 인지하고 조작할 수 있는 UI 만들기
>
> 폼의 제출 구조: [Form](form.md) · 입력의 상태: [Input](input.md)

- [1. 학습 목표](#1-학습-목표)
- [2. 접근성을 왜 배우는가](#2-접근성을-왜-배우는가)
- [3. 핵심 개념과 용어](#3-핵심-개념과-용어)
- [4. 관련 계층과 역할 구분](#4-관련-계층과-역할-구분)
- [5. 사용자가 입력에 접근하는 흐름](#5-사용자가-입력에-접근하는-흐름)
- [6. 코드 예제: 접근성을 고려한 기본 폼](#6-코드-예제-접근성을-고려한-기본-폼)
- [7. 오류와 결과를 전달하는 방법](#7-오류와-결과를-전달하는-방법)
- [8. 흔한 오해와 주의점](#8-흔한-오해와-주의점)
- [9. 실무 적용·성능·검증 방법](#9-실무-적용성능검증-방법)
- [10. 핵심 정리](#10-핵심-정리)
- [11. 이해도 확인 질문](#11-이해도-확인-질문)

## 1. 학습 목표

이 글을 학습한 뒤에는 다음을 할 수 있어야 한다.

- 접근성을 시각적 꾸미기가 아니라 기능 요구사항으로 설명한다.
- 화면의 표현과 보조 기술에 전달되는 의미를 구분한다.
- 입력의 이름, 설명, 상태를 적절한 HTML로 표현한다.
- 키보드만으로 폼과 버튼을 사용할 수 있는지 검증한다.
- 오류와 완료 상태를 색상에만 의존하지 않고 전달한다.
- 기본 HTML과 ARIA의 역할을 구분한다.

---

## 2. 접근성을 왜 배우는가

사용자는 모두 같은 방식으로 웹을 사용하지 않는다.

- 화면을 확대해 보는 사용자가 있다.
- 마우스 대신 키보드나 다른 입력 장치를 사용하는 사용자가 있다.
- 화면의 정보를 음성 등으로 전달하는 스크린 리더를 사용하는 사용자가 있다.
- 작은 글씨, 모호한 설명, 복잡한 오류 메시지 때문에 작업하기 어려운 사용자가 있다.

버튼이 화면에 보인다고 모두가 사용할 수 있는 것은 아니다. 입력창 옆에 글자가 있다고 그 글자가 보조 기술에도 입력의 이름으로 전달되는 것은 아니다.

접근성은 사용자가 정보를 이해하고, 기능을 찾아 조작하며, 결과를 확인할 수 있도록 구현하는 일이다. 폼에서는 레이블, 그룹, 안내, 검증, 결과 알림이 함께 갖춰져야 한다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

## 3. 핵심 개념과 용어

### 3.1 의미 있는 HTML

의미 있는 HTML은 요소의 역할에 맞는 태그를 사용하는 것이다.

- 이동은 `<a href="...">`
- 실행은 `<button>`
- 입력은 `<input>`, `<textarea>`, `<select>`
- 입력 이름은 `<label>`
- 관련 입력 그룹은 `<fieldset>`과 `<legend>`

기본 요소는 역할 정보뿐 아니라 브라우저의 조작 규칙도 제공한다. 반면 일반 요소에 ARIA 역할을 붙이는 것만으로 네이티브 요소의 키보드 동작이 생기지는 않는다.[1](https://www.w3.org/WAI/tutorials/forms/)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 3.2 접근 가능한 이름과 설명

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

### 3.3 역할·상태·값

사용자는 컨트롤을 만났을 때 다음을 알아야 한다.

- 이름: 무엇을 위한 것인가?
- 역할: 버튼인가, 입력인가, 체크박스인가?
- 상태: 선택되어 있는가, 비활성화되었는가?
- 값: 현재 무엇이 입력되어 있는가?

기본 HTML 입력은 이러한 정보의 상당 부분을 브라우저가 관리한다. 체크박스를 직접 그리는 것보다 기본 체크박스를 사용하는 편이 역할과 상태를 일관되게 제공하기 쉽다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 3.4 포커스

포커스는 현재 키보드 입력의 대상이 되는 위치다.

일반적인 키보드 사용에서는 다음 동작을 확인한다.

- Tab: 다음 조작 대상으로 이동
- Shift+Tab: 이전 조작 대상으로 이동
- Enter 또는 Space: 컨트롤 종류에 맞는 실행
- 방향키: 라디오 그룹 등에서 선택 이동

모든 컨트롤이 동일한 키로 동작하는 것은 아니다. 예를 들어 여러 라디오 버튼은 보통 Tab으로 그룹에 접근한 뒤 방향키로 선택을 이동한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 3.5 ARIA

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

## 4. 관련 계층과 역할 구분

접근성에서는 화면에 그려지는 결과와 보조 기술에 전달되는 정보가 모두 중요하다.

| 계층 | 역할 |
| --- | --- |
| HTML | 문서 구조, 컨트롤의 의미, 레이블 관계를 표현한다. |
| DOM | 실행 중인 문서 구조와 상태를 제공한다. |
| CSS | 화면 배치, 대비, 포커스 표시 등을 만든다. |
| 브라우저 | 구조와 상태를 해석해 화면과 접근성 정보를 제공한다. |
| 보조 기술 | 브라우저가 노출한 정보를 사용자가 이용할 수 있는 형태로 전달한다. |
| JavaScript | 필요한 동작을 구현하고 변경된 상태를 일관되게 반영한다. |

브라우저는 보조 기술에 전달할 구조를 구성한다. 이를 접근성 트리라고 부르며, DOM을 그대로 복제한 목록으로 생각해서는 안 된다. 요소의 의미, 레이블, 상태, 숨김 여부 등이 영향을 준다.

따라서 “눈에 잘 보인다”와 “보조 기술에 올바르게 전달된다”를 각각 검증해야 한다.[1](https://www.w3.org/WAI/tutorials/forms/)[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

## 5. 사용자가 입력에 접근하는 흐름

레이블과 설명이 연결된 입력으로 키보드 포커스가 이동하는 상황을 생각해 보자.

1. 브라우저가 입력의 역할과 이름을 해석한다.
2. 연결된 도움말이 있다면 설명 관계를 구성한다.
3. 필수 여부와 현재 값 등 관련 상태를 제공한다.
4. 보조 기술은 이 정보를 사용자에게 전달한다.
5. 사용자는 입력하고 결과를 확인한다.

실제 읽는 순서와 문구는 브라우저·스크린 리더·설정에 따라 달라질 수 있다. 특정 음성 출력 문장을 보장하기보다는 이름, 설명, 역할, 상태의 관계가 정확한지 확인한다.[1](https://www.w3.org/WAI/tutorials/forms/)[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

## 6. 코드 예제: 접근성을 고려한 기본 폼

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

### 6.1 HTML이 제공하는 의미

- `<main>`과 `<h1>`이 본문의 위치와 제목을 표현한다.
- `label`이 이메일과 문의 내용의 이름을 제공한다.
- `fieldset`과 `legend`가 라디오 버튼의 공통 질문을 표현한다.
- `required`가 브라우저에 필수 조건을 전달한다.
- `aria-describedby`가 이메일 입력과 도움말을 연결한다.[1](https://www.w3.org/WAI/tutorials/forms/)[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

### 6.2 화면과 상태의 변화

사용자가 Tab으로 이동하면 브라우저가 포커스를 이동하고, CSS가 포커스 위치를 표시한다.

라디오 버튼을 선택하면 브라우저가 선택 상태를 바꾼다. 별도의 JavaScript로 체크 모양만 바꾸는 방식이 아니라 실제 입력 상태가 바뀐다.

필수 값이 없거나 이메일 형식이 맞지 않으면 기본 검증이 제출을 막는다. 이 예제는 자체 오류 UI를 구현하지 않으므로 구체적인 메시지는 브라우저가 제공한다.[3](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input)[4](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)

### 6.3 CSS의 역할

CSS는 포커스가 어디에 있는지 눈으로 확인할 수 있게 만든다. 하지만 CSS가 레이블 연결이나 입력 검증을 대신하지는 않는다.

또한 예제의 포커스 스타일을 복사했다고 모든 배경과 배치에서 접근성 기준을 충족한다고 단정해서는 안 된다. 실제 주변 색상, 잘림, 가림 여부를 함께 확인해야 한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)[14](https://www.w3.org/WAI/WCAG22/Understanding/focus-appearance)

## 7. 오류와 결과를 전달하는 방법

### 7.1 색상만으로 오류를 전달하지 않는다

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

### 7.2 오류를 표시하는 시점도 설계한다

필수 입력이 비어 있다는 이유만으로 페이지가 열리자마자 모두 오류 처리하면, 사용자는 아직 시작하지도 않은 작업에서 실패한 것처럼 느낄 수 있다.

어떤 시점에 검사하고 안내할지 정하고, 오류가 수정되면 오래된 메시지와 상태도 함께 갱신한다. 오류 메시지는 문제가 있는 항목을 식별하고 해결 방법을 알려야 한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

### 7.3 완료 상태도 알려 준다

제출 버튼을 누른 뒤 아무 변화가 없으면 사용자는 작업이 끝났는지 알기 어렵다.

성공과 실패 모두 명확한 피드백이 필요하다. 다만 “브라우저 검증 통과”, “요청 전송”, “서버 저장 성공”은 서로 다른 상태이므로 실제로 확인된 결과만 안내해야 한다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

> 🔭 **미리보기: 동적 상태 알림**
>
> 페이지 이동 없이 결과가 갱신될 때는 화면 변화가 보조 기술에도 전달되도록 별도 알림 설계가 필요할 수 있다. 구체적인 구현은 이후 DOM / Event 학습을 거쳐 동적인 UI를 구현할 때 확장한다.

## 8. 흔한 오해와 주의점

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

## 9. 실무 적용·성능·검증 방법

### 9.1 키보드만으로 검사하기

마우스를 사용하지 않고 다음을 수행한다.

1. Tab과 Shift+Tab으로 필요한 컨트롤에 접근한다.
2. 현재 포커스 위치를 눈으로 확인한다.
3. 입력값을 수정한다.
4. 체크박스와 라디오 버튼을 조작한다.
5. 제출을 시도한다.
6. 오류를 찾고 수정한 뒤 다시 제출한다.

라디오 그룹처럼 여러 요소가 하나의 키보드 조작 단위를 이루는 경우가 있으므로, 모든 항목에 Tab이 한 번씩 멈추는지만으로 판단하지 않는다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

### 9.2 개발자 도구로 검사하기

브라우저의 Accessibility 관련 패널에서 다음을 확인한다.

- 입력에 이름이 있는가?
- 이름이 화면의 레이블과 일치하는가?
- 역할이 의도에 맞는가?
- 도움말이 설명으로 연결되어 있는가?
- 필수·선택·오류 상태가 실제 화면과 일치하는가?

도구의 패널 이름과 표시 방식은 브라우저마다 다를 수 있다.

### 9.3 스크린 리더로 검사하기

가능하다면 운영체제에서 제공하는 스크린 리더 등으로 실제 사용 흐름을 확인한다.

음성으로 들리는 문장을 외우기보다 다음을 확인한다.

- 입력 목적을 이해할 수 있는가?
- 그룹의 질문을 이해할 수 있는가?
- 현재 선택 상태를 알 수 있는가?
- 오류를 찾고 수정할 수 있는가?
- 작업 결과를 알 수 있는가?

### 9.4 유지보수와 성능 관점

기본 HTML 컨트롤을 사용하면 키보드 조작과 상태 관리를 직접 구현해야 하는 범위를 줄일 수 있다. 커스텀 컨트롤은 외형뿐 아니라 기본 컨트롤이 제공하던 동작과 정보를 재현해야 한다.[13](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)

접근성 개선을 불필요한 애니메이션이나 잦은 알림 추가로 오해하지 않는다. 현재 위치, 입력 조건, 오류, 완료 결과처럼 작업에 필요한 정보를 적절한 시점에 제공하는 것이 우선이다.[9](https://www.w3.org/WAI/tutorials/forms/notifications/)

---

## 10. 핵심 정리

> ✅ - 접근성은 UI의 필수 기능 요구사항이다.

- 화면의 표현과 보조 기술에 전달되는 의미를 함께 설계한다.
- 기본 HTML 요소가 제공하는 의미와 조작 규칙을 우선 활용한다.
- 입력의 이름, 설명, 상태는 서로 구분하여 표현한다.
- 오류는 색상뿐 아니라 수정 가능한 문장으로 전달한다.
- 자동 검사에 더해 키보드와 실제 보조 기술로 검증한다.

---

## 11. 이해도 확인 질문

> 📝 답을 정리하며 개념을 점검해 보세요. 코드가 있는 문항은 실행 결과를 먼저 예측한 뒤 직접 확인해 보세요.

1. 입력 위에 “이메일”이라는 문장이 보이지만 스크린 리더에서는 입력의 목적을 알기 어렵다. 어떤 HTML 관계를 확인하고 수정해야 하는가?
2. `<div role="button">저장</div>`이 마우스 클릭으로만 동작한다. 역할 선언과 실제 동작을 구분해 문제를 분석하고 우선적인 대안을 제시하라.
3. 디자인 시스템에서 모든 요소에 `outline: none`을 적용했다. 어떤 사용자 작업이 어려워지며, 어떤 방법으로 문제를 검증하겠는가?
4. 잘못된 입력에 빨간 테두리와 `aria-invalid="true"`만 추가했다. 사용자가 오류를 이해하고 수정하려면 무엇이 더 필요한가?
5. 화면에는 입력 순서가 자연스럽게 보이지만 Tab으로 이동하면 위치가 크게 오간다. 어떤 두 순서가 어긋났을 가능성이 있으며, 수정 후 무엇을 테스트해야 하는가?

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
