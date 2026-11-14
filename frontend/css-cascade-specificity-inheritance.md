---
track: B
notion: https://app.notion.com/p/406178c4afac8059934dc846a06c55fb
---

# CSS Cascade / Specificity / Inheritance


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · CSS Cascade / Specificity / Inheritance

선언의 경쟁, 선택자 비교, 값의 상속을 각각 이해한 뒤 한 요소의 최종 스타일을 추적한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 규칙·선언·속성·값과 계산·사용 값의 차이를 구별할 수 있다.
- 출처·중요도·레이어·명시도·순서로 우선순위를 판단할 수 있다.
- is·where·인라인·축약 속성의 예외를 분석할 수 있다.
- 직접 지정·상속·초기값·revert를 구별하여 스타일 초기화를 설계할 수 있다.
- Styles·Computed에서 문제의 단계와 실제 비용을 확인할 수 있다.

---

## 2. 이 주제를 왜 배우는가

스타일이 먹지 않을 때 선택자를 더 길게 쓰거나 `!important`를 덧붙이기 쉽다. 그러나 원인이 레이어나 상속이라면 명시도만 올려도 해결되지 않는다. **후보를 고르고 → 경쟁을 해결하고 → 필요한 값을 채운다**는 흐름으로 봐야 예측과 디버깅이 가능하다.

---

## 3. 관련 계층과 역할 구분

| 계층 | 책임 |
| --- | --- |
| HTML·DOM | 선택자가 대상으로 삼을 요소와 관계 제공 |
| CSS 규칙 | 후보 선언·조건·레이어와 값 표현 |
| 캐스케이드·기본값 결정 | 각 요소·속성의 값 선택과 상속·초기값 보충 |
| 레이아웃·페인트 | 결정된 스타일로 크기·배치·표현 계산 |
| JavaScript·React | 클래스·속성·인라인 값의 입력을 바꿀 수 있으나 CSS 우선순위를 대체하지 않음 |

선택자가 일치하지 않는 문제, 선언 경쟁에서 진 문제, 최종 값은 맞지만 가려진 문제는 각각 다른 단계다.

---

## 4. 핵심 개념과 역할

선언은 `color: navy`처럼 속성과 값의 쌍이다. 선택자는 선언이 적용될 요소를 고른다. 캐스케이드는 동일 요소의 동일 속성에 적용되는 여러 선언의 우선순위를 결정한다. 명시도(specificity)는 그 비교 기준 중 하나다. 상속(inheritance)은 부모의 계산된 값을 자식의 기본값 결정에 사용하는 별도 과정이다.

HTML은 요소 구조를 제공하고 CSS는 그 구조를 대상으로 값을 계산한다. JavaScript가 클래스나 인라인 스타일을 바꿀 수는 있지만, 최종 우선순위는 여전히 CSS 처리 규칙에 따른다.

---

## 5. CSS 값을 읽기 위한 기본 용어

### 규칙·선택자·선언·속성을 구별하기

```css
.notice {
  color: navy;
  padding: 1rem;
}
```

전체가 스타일 규칙이고 `.notice`는 대상 요소를 고르는 선택자다. 중괄호 안의 `color: navy`는 선언이며 color는 속성, navy는 값이다. 캐스케이드는 규칙 전체에서 하나를 뽑는 절차가 아니라 각 요소의 각 속성에 대해 적용할 선언을 고른다. 같은 규칙의 color는 이겨도 padding은 다른 규칙에 질 수 있다.

선택자가 정확히 일치하는지와 명시도가 높은지는 다른 문제다. `.notice .title`은 notice의 자손 title에 일치하고 `.notice.title`은 두 클래스를 동시에 가진 같은 요소에 일치한다. 구조를 잘못 읽었다면 우선순위를 높여도 후보 자체가 되지 않는다.

### 선언 값에서 실제 사용 값까지

| 단계 | 묻는 질문 | 예 |
| --- | --- | --- |
| 선언된 값 | 일치하는 유효한 선언에 어떤 값들이 있는가? | color에 red와 navy 후보 |
| 캐스케이드 값 | 우선순위 경쟁에서 무엇이 남았는가? | navy |
| 지정 값 | 후보가 없으면 상속·초기값으로 무엇을 채우는가? | 부모 글꼴 상속 |
| 계산 값 | 상대 단위와 의존성을 가능한 범위까지 어떻게 해석하는가? | em 단위를 계산 |
| 사용 값 | 배치 문맥까지 반영해 실제로 무엇을 사용하는가? | 백분율 폭을 배치 기준으로 결정 |
| 실제 값 | 장치 제약·반올림 등으로 어떻게 표현되는가? | 화면 픽셀로 표시 |

이 표는 처리 흐름을 이해하기 위한 구분이다. 모든 속성이 같은 시점에 같은 형태로 확정되지는 않는다. `getComputedStyle`도 속성에 따라 계산 값 또는 사용 값에 가까운 resolved value를 반환할 수 있으므로 이름만 보고 원본 선언이 그대로 나온다고 기대하지 않는다.

---

## 6. 선언에서 실제 값까지의 처리 흐름

먼저 선택자가 일치하고 조건이 맞는 선언만 후보가 된다. `@media` 조건이 거짓이거나 속성·값이 유효하지 않으면 일반적인 선언 경쟁에 참여하지 못한다. 이후 출처와 중요도, 레이어, 명시도, 필요한 경우 스코프 근접도, 작성 순서 등을 비교한다. 모든 후보를 명시도 하나로 한꺼번에 정렬하지 않는다.

### 출처와 중요도를 먼저 구별한다

브라우저의 기본 스타일, 사용자가 지정한 스타일, 작성자가 지정한 스타일은 출처(origin)가 다르다. 낮은 우선순위부터 크게 정리하면 다음과 같다.

| 낮음 → 높음 | 종류 |
| --- | --- |
| 1 | 브라우저 일반 선언 |
| 2 | 사용자 일반 선언 |
| 3 | 작성자 일반 선언 |
| 4 | 키프레임 애니메이션 값 |
| 5 | 작성자 `!important` |
| 6 | 사용자 `!important` |
| 7 | 브라우저 `!important` |
| 8 | 실행 중인 트랜지션 값 |

사용자 중요 선언이 작성자 중요 선언보다 높은 것은 사용자의 표현 요구를 보호하는 의미가 있다. 애니메이션·트랜지션 구현 자체는 이 페이지에서 확장하지 않고 우선순위의 예외로만 구별한다. 정확한 규칙은 [CSS Cascade 표준](https://www.w3.org/TR/css-cascade-5/)에서 확인할 수 있다.

### 레이어는 명시도보다 앞선다

`@layer`는 작성자 스타일을 명시적인 우선순위 묶음으로 나눈다. 일반 선언은 뒤에 선언된 레이어가 앞선 레이어보다 높고, 레이어 밖의 일반 선언은 레이어 안의 일반 선언보다 높다. 중요 선언에서는 레이어 순서가 반대로 되며, 레이어 안의 중요 선언이 레이어 밖의 중요 선언보다 높다.

따라서 외부 라이브러리 스타일을 낮은 일반 레이어에 두면, 애플리케이션 스타일에서 더 긴 선택자를 만들지 않고도 덮어쓸 수 있다. 인라인 일반 스타일은 일반적인 작성자 스타일 규칙보다 우선하지만, 이를 모든 `!important`보다 강하다는 뜻으로 이해하면 안 된다.

---

## 7. 캐스케이드를 작은 비교 실험으로 이해하기

### 중요 선언은 선택자의 명시도를 올리는 기능이 아니다

```css
#app .message { color: red; }
.message { color: blue !important; }
```

같은 요소에 적용되는 작성자 선언이라면 blue가 적용된다. `.message`가 갑자기 ID보다 높은 명시도를 얻는 것이 아니라 중요도 단계에서 일반 선언보다 앞선다. 다른 중요 선언끼리는 다시 해당 단계의 레이어·명시도 등을 비교한다.

접근성 도구나 사용자의 중요 스타일도 이 경쟁에 참여할 수 있다. 작성자의 편의를 위해 모든 선언에 important를 붙이면 이후 예외를 표현하기 어려워지고 사용자의 표현 요구를 확인하기도 힘들어진다.

### 인라인 스타일도 무조건 마지막 승자가 아니다

```html
<p class="message" style="color: red">안내</p>
<style>
  .message { color: blue !important; }
</style>
```

인라인의 일반 color보다 작성자 중요 color가 우선한다. 인라인 스타일을 별도의 “ID 네 개” 같은 점수로 설명하면 중요도·출처 규칙을 놓친다. 인라인 위치의 우선순위와 선언 중요도를 나누어 판단한다.

### 레이어 순서를 먼저 선언하는 이유

```css
@layer reset, vendor, components, utilities;
@layer vendor {
  #app button.action { color: red; }
}
@layer components {
  .action { color: navy; }
}
```

일반 선언에서 components가 vendor보다 뒤 레이어이므로 navy가 이긴다. 파일이 나중에 추가되더라도 상단의 레이어 순서가 우선순위 의도를 보여준다. 레이어 밖 일반 선언과 important의 역전 규칙까지 포함하여 프로젝트의 규칙을 정해야 한다. 레이어가 있다고 파일 순서의 모든 의미가 사라지는 것은 아니다.

---

## 8. 같은 경쟁 단계에서 명시도를 비교한다

명시도는 보통 `(ID 수, 클래스 계열 수, 타입 계열 수)`로 비교한다. 왼쪽부터 비교하며 자릿수를 올리는 십진수 점수가 아니다.

| 선택자 | 명시도 |
| --- | --- |
| `p` | `(0, 0, 1)` |
| `.note` | `(0, 1, 0)` |
| `[data-kind="note"]:hover` | `(0, 2, 0)` |
| `#main .note` | `(1, 1, 0)` |

속성 선택자와 일반적인 의사 클래스는 클래스 열에, 의사 요소는 타입 열에 센다. `*`와 `>`, `+`, 공백 같은 결합자는 명시도를 더하지 않는다.

`:where(...)`는 자신과 인수의 명시도가 0이다. `:is(...)`, `:not(...)`, `:has(...)`는 일반적인 의사 클래스 하나로 세지 않고 인수 목록에서 가장 높은 명시도를 사용한다. `:nth-child(... of S)`는 의사 클래스 자신의 값에 `S` 목록의 가장 높은 값을 더한다. [Selectors 표준](https://www.w3.org/TR/selectors-4/#specificity)의 규칙에 따라 판단한다.

명시도까지 같으면 일반적인 전역 규칙에서는 나중 선언이 이긴다. `@scope`가 있는 스타일에서는 작성 순서 전에 스코프 근접도 비교도 고려해야 한다. “CSS는 마지막 줄이 이긴다”는 말은 앞의 비교가 모두 동률일 때만 성립한다.

---

## 9. 명시도 계산에서 자주 틀리는 경우

### 숫자를 이어 붙이는 점수로 계산하지 않는다

`.a.b.c.d.e.f.g.h.i.j.k`는 클래스가 많아도 ID 하나의 열을 넘어서는 자릿수 올림이 없다. `(0,11,0)`과 `(1,0,0)`은 왼쪽 열부터 비교한다. 다만 두 선언이 같은 출처·중요도·레이어 등에 있어 명시도를 비교하는 상황인지가 먼저다.

### is와 where는 목적이 다르다

```css
:where(#app, .page) .title { color: green; }
:is(#app, .page) .title { color: purple; }
```

첫 선택자는 where 인수가 0이므로 `(0,1,0)`이다. 둘째는 is 목록 중 ID의 명시도를 사용하여 `(1,1,0)`이 된다. 실제로 page 쪽에서 일치했더라도 목록의 높은 명시도가 영향을 줄 수 있다. 같은 그룹 표현처럼 보여도 스타일을 덮어쓸 수 있는 정도가 달라진다.

컴포넌트 기본 스타일을 쉽게 덮어쓰게 제공하려면 where가 유용할 수 있다. 반대로 의도 없이 is 안에 ID를 섞으면 전체 규칙의 명시도가 높아질 수 있다. 유지보수 기준까지 포함하여 선택한다.

### 선택자 목록의 쉼표는 전체 명시도를 합하지 않는다

```css
#dialog, .panel { color: navy; }
```

일반 선택자 목록은 일치하는 선택자별로 명시도를 다룬다. 이 규칙을 `.panel`로 일치한 요소가 자동으로 ID 명시도를 얻는 것으로 이해하지 않는다. is의 인수 목록에 대한 특별 규칙과 쉼표로 나열한 일반 선택자를 구별한다.

---

## 10. 예제: 레이어와 상속을 하나의 화면에서 추적하기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>CSS 값 결정</title>
  <style>
    @layer base, components;
    @layer base {
      #panel .message { color: crimson; }
    }
    @layer components {
      .message { color: navy; }
    }
    #panel { color: green; font-family: sans-serif; }
    .message { border: 2px solid; padding: 1rem; }
    .detail { color: inherit; }
  </style>
</head>
<body>
  <section id="panel">
    <h1>값 결정 실험</h1>
    <p class="message">메시지 <span>안쪽 문장</span></p>
    <p class="detail">부모의 색 사용</p>
  </section>
</body>
</html>
```

`.message`에는 레이어 `components`의 navy가 적용된다. `base`의 ID 선택자가 더 강해 보여도 레이어에서 먼저 승부가 난다. 레이어 밖의 `.message` 규칙에는 color 선언이 없으므로 color 경쟁의 후보가 아니다. 경쟁은 규칙 전체가 아니라 **속성마다** 따로 이루어진다.

`span`에는 직접 지정된 color가 없으므로 부모 `p`의 navy를 상속한다. `.detail`에는 명시적으로 `inherit`가 지정되어 `section`의 green을 사용한다. 글꼴은 상속되지만 padding과 border는 자동으로 자식에게 상속되지 않는다. `border`의 색을 따로 지정하지 않았으므로 현재 글자색을 뜻하는 `currentColor`가 사용되어 메시지 테두리도 navy가 된다.

---

## 11. 상속은 부모 선택자와 자식 선택자의 결승전이 아니다

```css
#panel { color: green; }
span { color: purple; }
```

부모의 ID 선택자와 자식의 타입 선택자 명시도를 비교해 green을 고르지 않는다. `span`에 직접 적용되는 purple이 먼저 그 요소의 캐스케이드로 결정된다. 부모의 색은 직접 값이 없거나 `inherit` 등을 사용했을 때 관여한다.

`color`, `font-family` 같은 속성은 기본적으로 상속되고 `margin`, `padding`, `border` 등은 기본적으로 상속되지 않는다. 속성별 정의를 확인해야 한다.

| 키워드 | 의미 |
| --- | --- |
| `inherit` | 부모의 계산된 값 사용 |
| `initial` | CSS에 정의된 그 속성의 초기값 사용 |
| `unset` | 상속 속성이면 inherit, 아니면 initial |
| `revert` | 현재 출처를 되돌리는 방식으로 값 결정 |
| `revert-layer` | 현재 레이어를 되돌리는 방식으로 값 결정 |

`initial`은 브라우저 기본 스타일로 돌아간다는 뜻이 아니다. 예를 들어 속성의 초기값과 특정 HTML 요소에 대한 브라우저 스타일은 다를 수 있다.

---

## 12. 상속을 직접 실험하고 초기값과 구별하기

### 부모의 계산된 값을 받는다

```html
<div class="parent">
  부모
  <p class="child">자식 <span>손자</span></p>
</div>
<style>
  .parent { color: darkgreen; font-size: 20px; }
  .child { font-size: 1.5em; padding: 10px; }
</style>
```

child는 부모의 색을 상속하고 글꼴 크기는 부모 기준으로 30px이 된다. span의 글꼴 크기에 직접 지정이 없다면 child의 계산된 글꼴 크기를 상속한다. 부모의 `1.5em` 문자열을 복사해서 손자에서 다시 곱하는 방식이 아니다. 다만 line-height처럼 값의 종류에 따라 상속 결과의 해석이 달라지는 속성도 있어 속성 정의를 확인해야 한다.

padding은 상속되지 않는다. 부모에게 10px padding이 있다는 것은 부모의 안쪽 공간이 10px이라는 뜻이지 모든 자식에게 10px이 복제된다는 뜻이 아니다.

### initial과 브라우저 기본 스타일은 다르다

```css
/* div의 브라우저 기본 배치와 CSS display 초기값은 구별한다. */
.reset-display { display: initial; }
```

display의 초기값은 inline이다. div가 기본적으로 block으로 보이는 이유는 해당 요소에 적용되는 브라우저 스타일과 연결된다. 따라서 initial은 “이 HTML 요소가 처음 보이던 모양”으로 돌아가는 범용 초기화가 아니다. revert는 출처를 되돌리는 관점, revert-layer는 레이어를 되돌리는 관점으로 비교한다.

### 축약 속성은 여러 하위 속성에 영향을 준다

```css
.card { border-color: red; }
.card { border: 1px solid; }
```

뒤의 border 축약 선언이 border-color도 지정한다. color를 쓰지 않았다고 이전 빨간 테두리만 무조건 유지하는 것은 아니다. background, font 같은 축약도 누락된 부분의 기본 설정까지 영향을 줄 수 있으므로 DevTools에서 하위 속성을 펼쳐 확인한다.

### 사용자 정의 속성을 사용할 때의 상속

```css
.theme { --accent: navy; }
.button { color: var(--accent, black); }
```

일반적인 `--name` 사용자 정의 속성은 상속되므로 테마 값을 아래 요소로 전달할 수 있다. var의 fallback은 접근 가능한 사용자 정의 속성이 없거나 해당 규칙에 따른 무효 상태일 때 쓰는 값이며, 어떤 잘못된 최종 값이든 자동으로 안전한 값으로 바꾸는 검사기는 아니다. 등록된 사용자 정의 속성은 상속 설정 등이 달라질 수 있다.

---

## 13. 캐스케이드를 속성별 후보 표로 계산하기

### 경쟁의 단위는 요소 하나의 속성 하나다

어떤 요소에 “A 규칙이 적용됐다”라고만 말하면 다른 속성의 결정을 놓친다. 한 규칙 안에서도 color는 이기고 padding은 질 수 있다. 실제 판정은 대상 요소를 하나 고르고 속성을 하나 고른 다음, 일치하고 유효한 선언만 모으는 방식으로 진행한다. 이 습관은 긴 스타일시트에서도 원인을 추적할 수 있게 한다.

스타일의 출처와 원소의 DOM 위치도 구별한다. 자식이 부모보다 나중에 적혔다는 이유로 부모의 규칙과 경쟁하는 것은 아니다. 각 요소에서 직접 적용되는 선언을 먼저 계산하고, 그 결과를 채우는 과정에서 상속이 관여한다. [CSS Cascade의 값 처리 단계](https://www.w3.org/TR/css-cascade-5/#value-stages)를 이 순서의 근거로 삼는다.

### 실험 1. 하나의 규칙이 통째로 승리하지 않는다

다음 파일을 브라우저에서 열고 Elements의 Styles와 Computed를 함께 확인한다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>속성별 캐스케이드</title>
  <style>
    #sample { color: navy; }
    .card { color: crimson; padding: 24px; }
    p { padding: 8px; border: 1px solid; }
  </style>
</head>
<body>
  <p id="sample" class="card">속성마다 후보를 모읍니다.</p>
</body>
</html>
```

| 속성 | 경쟁하는 선언 | 결과를 결정한 기준 |
| --- | --- | --- |
| color | ID의 navy, 클래스의 crimson | 같은 일반 작성자 단계에서 명시도 |
| padding | 클래스의 24px, 타입의 8px | 같은 단계에서 명시도 |
| border | 타입의 1px solid | 해당 선언이 후보이며 테두리 색은 currentColor |

ID 규칙이 color에서 이겼다고 그 규칙에 없는 padding까지 결정하지 않는다. 테두리 색은 color와 연결된 currentColor이므로 navy로 보인다. 최종 결과는 여러 규칙에서 선택한 값들의 조합이다. 실제 색은 DevTools에서 rgb 형식으로 표시될 수 있으므로 원본 표기와 계산 결과 표기를 구별한다.

### 선언이 후보가 되지 못하는 경우

선택자가 불일치하거나, media 조건이 거짓이거나, 속성 문법에 맞지 않는 선언은 뒤의 명시도 비교로 해결되지 않는다. `padding: very-big`처럼 해석할 수 없는 값을 넣고 ID를 늘려도 유효한 padding이 되지 않는다. 브라우저가 선언을 취소선으로 표시했는지, 구문 오류로 표시했는지 먼저 나눈다.

캐스케이드에서 탈락한 선언과 변수 치환 뒤 무효가 되는 선언은 서로 다른 단계다. 이 차이는 뒤의 변수 실험에서 결과를 바꾸므로, “잘못된 값이면 앞의 선언을 쓴다”를 모든 상황의 규칙으로 외우지 않는다.

---

## 14. 레이어·중요도·작성 순서를 분리하는 실험

### 실험 2. 일반 레이어와 중요 레이어는 방향이 다르다

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>레이어의 방향</title>
  <style>
    @layer foundation, application;
    @layer foundation {
      .normal { color: crimson; }
      .important { color: crimson !important; }
    }
    @layer application {
      .normal { color: navy; }
      .important { color: navy !important; }
    }
    .normal { color: darkgreen; }
    .important { color: darkgreen !important; }
  </style>
</head>
<body>
  <p class="normal">일반 선언</p>
  <p class="important">중요 선언</p>
</body>
</html>
```

normal은 레이어 밖 일반 선언인 darkgreen을 사용한다. important는 먼저 선언한 foundation 레이어의 crimson을 사용한다. 같은 단어 important를 붙였다고 작성 순서만 비교하지 않는다. 중요 선언은 레이어 우선순위가 반대이고, 레이어 안의 중요 선언이 레이어 밖 중요 선언보다 앞선다. [레이어 순서 규칙](https://www.w3.org/TR/css-cascade-5/#layer-ordering)을 각각의 속성에 적용한 결과다.

이 예제에서 important를 모두 제거하거나 레이어 순서를 바꾸면 결과가 달라진다. 한 번에 하나의 조건만 바꿔야 중요도와 레이어 중 어느 차이가 결과를 만들었는지 설명할 수 있다. 모든 실험을 한꺼번에 변경한 뒤 색만 비교하면 원인을 배우기 어렵다.

### 실험 3. 같은 명시도라야 마지막 순서가 의미 있다

```css
.notice { color: navy; }
.notice { color: crimson; }
```

같은 조건·출처·중요도·레이어이고 같은 선택자라면 뒤의 crimson이 선택된다. 두 규칙을 다른 파일로 나누어 연결해도 최종 스타일시트 순서가 관여한다. 네트워크에서 어떤 파일이 먼저 도착했는지를 마지막 선언 규칙과 동일시하지 않는다. DOM에서 연결한 스타일시트의 순서와 실제 캐스케이드 구성을 확인한다.

### 명시도 숫자를 올리기 전에 책임을 설계한다

외부 UI 라이브러리의 일반 스타일보다 애플리케이션의 일반 스타일이 우선해야 한다면 레이어로 그 관계를 표현할 수 있다. 이미 중요 선언이 많은 라이브러리에는 같은 설계를 그대로 적용해 원하는 결과가 나오는지 확인해야 한다. 스타일의 원래 계약을 읽지 않고 무조건 ID나 important로 덮으면 다음 수정의 비용이 커진다.

---

## 15. 명시도의 특별 규칙을 실제 선택자에 적용하기

### 실험 4. is 목록은 실제 일치한 분기만 세는 규칙이 아니다

```html
<div class="panel"><p class="title">목록 선택자</p></div>
<style>
  :is(#unused, .panel) .title { color: navy; }
  .panel .title { color: crimson; }
</style>
```

현재 문서에 unused ID가 없어도 첫 선택자는 is 인수 목록의 ID 명시도에 영향을 받는다. 첫 선택자의 명시도는 `(1,1,0)`, 둘째는 `(0,2,0)`이다. 따라서 같은 경쟁 단계라면 navy다. 실제로는 panel 분기를 통해 일치했다는 사실과 is의 명시도를 정하는 규칙을 구별한다.

is를 where로 바꾸면 첫 선택자는 `(0,1,0)`이 되어 둘째의 crimson이 이길 수 있다. 이것은 브라우저가 제목의 의미를 다르게 읽어서가 아니라, [Selectors의 명시도 규칙](https://www.w3.org/TR/selectors-4/#specificity)이 두 의사 클래스를 다르게 정의하기 때문이다.

### nth-child의 of 목록은 별도의 가산 부분이다

`:nth-child(2 of .active)`는 nth-child 자체의 의사 클래스 명시도와 of 목록의 명시도를 함께 고려한다. 단순히 괄호가 있다고 모두 0이거나, 모든 함수형 의사 클래스가 is와 똑같이 동작하는 것은 아니다. 어떤 함수인지 먼저 구별한 뒤 해당 규칙으로 계산한다.

### 가상 요소와 결합자는 다른 열에 있다

`li::marker`는 타입 li와 의사 요소 marker가 타입 열에 기여한다. 반면 `main > .card + .card`의 `>`·`+`는 관계를 정하지만 명시도를 추가하지 않는다. 관계가 까다롭다는 인상과 명시도는 다른 개념이다. 선택자의 길이를 성능이나 우선순위의 숫자로 곧바로 치환하지 않는다.

---

## 16. 상속되는 값의 종류가 자식의 표현을 바꾸는 경우

### 실험 5. 숫자 line-height와 길이 line-height

```html
<div class="unitless"><p>단위 없는 줄 높이</p></div>
<div class="length"><p>길이로 지정한 줄 높이</p></div>
<style>
  .unitless, .length { font-size: 20px; }
  .unitless { line-height: 1.5; }
  .length { line-height: 30px; }
  p { font-size: 40px; margin: 0; }
</style>
```

unitless의 자식은 숫자 1.5를 받아 자신의 글꼴 크기 40px에 맞는 줄 높이를 사용할 수 있다. length의 자식은 길이 30px을 상속하여 큰 글자가 좁은 줄 높이 안에 놓일 수 있다. 상속은 원본 CSS 문자열을 자식마다 임의로 다시 실행하는 과정이 아니라 속성에 정의된 계산 값의 전달이다. [MDN의 line-height 값 정의](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/line-height)로 숫자와 길이의 차이를 확인한다.

이 차이는 확대와 컴포넌트별 글꼴 크기에서 접근성 문제를 만들 수 있다. 단위 없는 값을 항상 정답으로 외우기보다, 하위 글꼴 크기가 바뀌어도 줄 간격이 읽기 가능한지 확인한다. 언어·폰트·이모지에 따라 실제 글리프의 범위도 달라진다.

### 실험 6. initial·inherit·unset을 같은 구조에 적용하기

```html
<div class="parent">
  <p class="a">initial</p>
  <p class="b">inherit</p>
  <p class="c">unset</p>
</div>
<style>
  .parent { color: darkgreen; padding: 20px; }
  p { margin: 0; border: 1px solid; }
  .a { color: initial; padding: initial; }
  .b { color: inherit; padding: inherit; }
  .c { color: unset; padding: unset; }
</style>
```

color는 상속 속성, padding은 비상속 속성이다. a는 두 속성의 CSS 초기값, b는 두 속성 모두 부모의 계산 값, c는 color를 상속하고 padding을 초기값으로 결정한다. 비상속 속성도 inherit를 명시하면 부모 값을 받을 수 있으므로 “padding은 절대 상속할 수 없다”도 틀리다.

초기 color의 구체적인 표시를 임의로 항상 black이라고 고정하지 않는다. 속성 정의·시스템 색·사용 환경의 영향을 확인한다. 이 실험의 목적은 정확한 색 이름 암기보다 값이 어느 경로에서 왔는지 구별하는 데 있다.

---

## 17. 무효한 선언과 무효한 계산 결과를 구별하기

### 실험 7. 파싱 시점의 오류

```css
.notice { background-color: crimson; }
.notice { background-color: 20px; }
```

20px은 background-color 값으로 유효하지 않아 해당 선언이 보통 후보에서 제외된다. 다른 유효 선언인 crimson을 사용할 수 있다. 이 경우에는 앞 선언으로 보이는 결과가 나와도 “어떤 실패든 이전 값으로 돌아간다”는 일반 규칙이 되는 것은 아니다.

### 실험 8. 변수 치환 뒤의 오류

```html
<div class="notice">변수 치환 실험</div>
<style>
  .notice { --bad-color: 20px; background-color: crimson; }
  .notice { background-color: var(--bad-color, navy); }
</style>
```

사용자 정의 속성에 20px을 보관하는 것 자체는 가능하다. 뒤의 선언은 var를 포함한 상태로 캐스케이드에서 선택되고, 치환 후에 background-color에 맞지 않는다는 문제가 드러난다. 그 시점에는 앞선 crimson을 다시 경쟁시키지 않는다. 비상속 background-color는 초기값을 사용하여 보통 투명한 배경이 된다.

fallback navy도 사용되지 않는다. bad-color가 존재하며 그 자체가 사용할 수 없는 변수 상태였던 것이 아니라, 치환 결과가 대상 속성에 맞지 않았기 때문이다. [CSS Variables의 무효 계산 값 규칙](https://www.w3.org/TR/css-variables-1/#invalid-variables)이 이 반례의 근거다.

### 실험 결과를 읽는 방법

Styles에서 보이는 원본 값과 Computed에서 얻은 결과를 함께 기록한다. 지원하지 않는 문법이라 선언을 버린 것인지, 조건에 불일치한 것인지, 캐스케이드에서 진 것인지, 계산 단계에서 무효가 된 것인지 구별한다. 결과 화면 한 장만으로는 이 네 경우를 구별하기 어렵다.

---

## 18. 스타일 설계와 검증을 하나의 흐름으로 만들기

### 덮어쓸 수 있는 기본값과 보호할 계약

재사용 UI의 기본 스타일은 낮은 명시도와 명확한 레이어로 제공하면 소비자가 의도를 표현하기 쉽다. 반면 접근성에 필요한 포커스 표시를 모든 화면에서 제거하도록 기본값을 정하면 덮어쓰기 편의와 별개로 기능 문제가 생긴다. CSS 우선순위 설계는 사용자가 실제로 읽고 조작할 조건까지 포함한다.

### 한 번에 바꿀 변수를 제한한다

버그를 재현할 최소 HTML과 필요한 규칙만 남긴다. 레이어·important·명시도·순서를 각각 바꾸고 최종 값과 이유를 기록한다. 이때 규칙을 제거하면서 문제가 사라졌다고 곧바로 그 선택자가 느리다는 결론을 내리지 않는다. 표현 오류와 성능 병목은 다른 증거가 필요하다.

### 표준 보장과 도구 표시를 구별한다

표준은 우선순위와 값 처리 규칙을 설명한다. DevTools는 그 규칙을 관찰하는 구현별 UI이고, 탭 이름·취소선 표시·속성 설명은 버전에 따라 달라질 수 있다. 따라서 특정 버튼의 위치보다 “원본 후보, 선택된 값, 상속 출처를 확인한다”는 검증 목적을 기억한다.

---

## 19. CSS가 적용되지 않을 때의 진단 절차

1. 실제 DOM에서 선택자가 일치하는지 확인한다. 파서의 구조 복구나 클래스 오타부터 배제한다.
2. 선언의 속성과 값이 유효한지, 미디어 쿼리 같은 조건이 참인지 확인한다.
3. 출처·important·레이어를 확인한 뒤 명시도를 비교한다.
4. 같은 속성의 축약·하위 속성이 서로 덮는지 확인한다.
5. 직접 값과 부모에서 받은 값을 구분하고 Computed에서 최종 값의 출처를 찾는다.
6. 최종 값이 맞으면 크기·가림·레이아웃처럼 다른 단계의 문제를 조사한다.

이 순서를 따르면 스타일이 안 보인다는 이유만으로 important를 더하는 시행착오를 줄일 수 있다. 성능 개선도 선택자를 무조건 짧게 바꾸는 대신 실제 스타일 재계산 구간과 변경 빈도를 측정한 뒤 판단한다.

---

## 20. 흔한 오해와 실무 디버깅

DevTools의 Styles에서 적용되지 않은 선언의 이유를 먼저 확인한다. 선택자 불일치인지, 조건 불일치인지, 중요도·레이어에서 진 것인지 구별하고 Computed에서 최종 값과 상속 경로를 확인한다. 축약 속성도 의도치 않게 하위 속성을 덮어쓸 수 있으므로 함께 확인한다.

접근성을 위해 사용자가 적용한 고대비·글자 크기 요구를 무분별한 중요 선언으로 방해하지 않는다. 유지보수 측면에서는 명시도를 계속 올리기보다 레이어와 낮은 명시도의 클래스 규칙으로 우선순위를 설계한다. 선택자의 모양만으로 성능 병목을 단정하지 않고 실제 스타일 재계산 비용을 측정한다.

---

## 21. 핵심 정리

<callout icon="✅" color="green_bg">
캐스케이드는 선언 경쟁 전체이고 명시도는 그 안의 비교 기준이다. 상속은 부모와 자식 선택자의 명시도 경쟁이 아니라 요소의 값을 채우는 과정이다. 이 순서를 지키면 “왜 이 색이 나왔는가”를 속성별로 설명할 수 있다.
</callout>

---

## 출처

- [W3C — CSS Cascade Level 5](https://www.w3.org/TR/css-cascade-5/)
- [W3C — Selectors Level 4: Specificity](https://www.w3.org/TR/selectors-4/#specificity)
- [W3C — CSS Cascade Level 6: Scope Proximity](https://www.w3.org/TR/css-cascade-6/#cascade-sort)
- [W3C — CSS Values and Units](https://www.w3.org/TR/css-values-4/)
- [W3C — CSS Custom Properties](https://www.w3.org/TR/css-variables-1/)

### 이번 설명의 근거와 적용 범위

- [W3C — CSS Cascade: 값 처리 단계](https://www.w3.org/TR/css-cascade-5/#value-stages): 선언·캐스케이드·지정·계산·사용 값의 구분.
- [W3C — Layer Ordering](https://www.w3.org/TR/css-cascade-5/#layer-ordering): 일반·중요 선언에서 레이어 순서가 달라지는 근거.
- [W3C — Selectors Specificity](https://www.w3.org/TR/selectors-4/#specificity): is·where·nth-child·의사 요소의 계산 기준.
- [W3C — Invalid Variables](https://www.w3.org/TR/css-variables-1/#invalid-variables): 계산 시점에 무효가 된 값이 앞선 선언으로 돌아가지 않는 이유.
- [MDN — line-height](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Properties/line-height): 숫자·길이와 상속 결과의 차이. MDN은 Mozilla가 제공하는 웹 플랫폼 문서다.

---

## 22. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 앞 레이어의 ID 선택자와 뒤 레이어의 클래스 선택자가 color를 지정했다. 일반 선언과 중요 선언일 때 각각 어떤 순서로 판단하는가?
2. 부모에 `#root { color: red }`, 자식에 `span { color: blue }`가 있다. 부모 ID가 자식 색을 이기지 않는 이유를 설명하라.
3. `:where(#root) .note`와 `.note:hover`의 명시도를 계산하고, 명시도 비교가 의미 있으려면 어떤 앞선 조건이 같아야 하는지 설명하라.
4. `initial`, `inherit`, `unset`을 color와 padding에 적용할 때 어떻게 다르게 작동하는가?
