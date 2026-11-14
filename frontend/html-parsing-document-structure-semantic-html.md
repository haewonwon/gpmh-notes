---
track: B
notion: https://app.notion.com/p/3fb178c4afac802080f4f928ea59409a
---

# HTML Parsing / Document 구조 / Semantic HTML


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · HTML Parsing / Document 구조 / Semantic HTML

HTML이 해석되는 과정, 만들어지는 문서의 구조, 사용자에게 전달되는 의미를 각각 충분히 배우고 하나의 문서 작성 흐름으로 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 바이트·문자·토큰·트리 구성의 역할과 오류 복구 결과를 설명할 수 있다.
- doctype, html, head, body와 주요 메타데이터의 역할을 구별할 수 있다.
- 노드 관계와 콘텐츠 모델로 올바른 문서 구조를 판단할 수 있다.
- 랜드마크·헤딩·목록·표·링크·버튼을 목적에 맞게 고를 수 있다.
- 소스와 실제 DOM이 다를 때 접근성·스타일 문제의 원인을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

브라우저는 HTML을 그림처럼 표시하지 않는다. 먼저 입력을 해석하여 객체들의 관계를 만들고, 이 구조에 스타일과 동작을 적용한다. 따라서 HTML을 이해한다는 것은 태그를 외우는 데서 끝나지 않는다. **문자열이 어떤 트리가 되고, 그 트리가 무엇을 뜻하는지**까지 이어져야 한다.

잘못 중첩한 HTML이 화면에 보인다는 이유로 올바른 문서가 되는 것도 아니다. 브라우저가 입력을 복구했을 수 있고, 그 결과가 작성자의 의도와 다를 수 있다.

---

## 3. 핵심 개념과 용어

| 용어 | 정의 | 왜 중요한가 |
| --- | --- | --- |
| **파싱(Parsing)** | 정해진 문법에 따라 텍스트를 읽고, 프로그램이 다룰 수 있는 구조(여기서는 트리)로 바꾸는 과정 | 브라우저는 텍스트가 아니라 파싱 결과(DOM)로 일한다 |
| **바이트 스트림 / 문자 인코딩** | 네트워크로 오는 것은 0과 1의 바이트이고, 이를 문자로 해석하는 규칙이 인코딩(예: UTF-8)이다 | 인코딩을 잘못 판단하면 한글이 깨진다 |
| **토큰(Token)** | 파서가 의미 단위로 잘라 낸 조각. DOCTYPE, 시작 태그, 종료 태그, 주석, 문자, 파일 끝(EOF) 토큰이 있다 | 트리 구성 단계의 입력이 된다 |
| **토크나이저(Tokenizer)** | 문자를 하나씩 읽으며 상태를 바꿔 가는 상태 기계(state machine). 토큰을 만들어 낸다 | `<script>` 안의 `<`를 태그로 볼지 글자로 볼지 등을 결정한다 |
| **트리 구성(Tree construction)** | 토큰을 받아 DOM 노드를 만들고 트리의 어디에 붙일지 결정하는 단계 | 오류 복구(자동 수정)가 대부분 여기서 일어난다 |
| **DOM(Document Object Model)** | 문서를 노드와 객체의 트리로 표현하고, 프로그램이 그 구조·스타일·내용을 바꿀 수 있게 하는 프로그래밍 인터페이스 | CSS·JS·React·접근성이 모두 DOM을 기준으로 동작한다 |
| **열린 요소 스택(Stack of open elements)** | 아직 닫히지 않은 요소들을 순서대로 쌓아 둔 목록. 맨 위가 "현재 노드"다 | 새 노드가 어느 부모 밑에 들어갈지 결정한다 |
| **삽입 모드(Insertion mode)** | 트리 구성 단계가 지금 문서의 어느 부분을 처리 중인지 나타내는 상태(예: "in head", "in body", "in table") | 같은 토큰도 모드에 따라 다르게 처리된다 |
| **파스 에러(Parse error)** | HTML 문법 오류. 표준은 각 오류마다 어떻게 복구할지 정해 둔다 | 에러가 나도 페이지는 뜨지만, 결과 DOM이 의도와 다를 수 있다 |
| **빈 요소(Void element)** | 내용과 종료 태그를 가질 수 없는 요소. `br`, `img`, `input`, `meta`, `link`, `hr` 등 | `<div />`처럼 아무 요소에나 `/`를 붙여도 닫히지 않는 이유와 연결된다 |
| **직렬화(Serialization)** | 파싱의 반대. DOM 트리를 다시 HTML 문자열로 바꾸는 것. `innerHTML`을 읽을 때 일어난다 | DevTools와 `innerHTML`이 보여 주는 HTML은 원본이 아니라 직렬화 결과다 |

파싱은 구조를 만드는 절차이고 Document는 그 결과를 담는 문서 단위다. 시맨틱은 그 구조 안의 요소가 나타내는 의미다. 이 셋을 연결하면 소스에 잘못된 중첩이 있을 때 왜 구조와 접근성 정보가 함께 달라질 수 있는지도 설명할 수 있다.

---

## 4. 관련 계층과 역할 구분

| 계층 | 다루는 것 | 이 페이지에서의 질문 |
| --- | --- | --- |
| HTML 소스 | 요소·속성·텍스트를 표현한 마크업 | 무엇을 작성했는가? |
| HTML 파서 | 입력을 토큰과 문서 트리로 변환 | 어떤 구조로 해석했는가? |
| DOM | 문서를 객체와 부모·자식 관계로 표현한 모델 | 실제 문서는 어떤 트리인가? |
| CSS | 표현과 배치 | 어떻게 보이는가? |
| 접근성 트리 | 보조 기술에 필요한 역할·이름·상태 등의 정보 | 어떤 대상이고 어떻게 탐색하는가? |

요소(element)는 문서를 구성하는 단위다. `<p>`와 `</p>`는 요소의 경계를 나타내는 태그이며, 태그 문자열과 실제 요소 객체는 같은 것이 아니다. DOM의 자세한 조작 API는 뒤의 DOM 학습에서 다룬다.

---

## 5. HTML Parsing: 입력이 트리가 되는 과정

[HTML 표준의 파싱 모델](https://html.spec.whatwg.org/multipage/parsing.html)은 크게 두 단계다. 입력 문자를 **토큰화(tokenization)** 하고, 그 토큰으로 **트리를 구성(tree construction)** 해 DOM `Document`를 만든다. 그 앞에는 바이트를 문자로 바꾸는 디코딩 단계가 있다. 하나씩 따라가 보자.

### 바이트 → 문자: 인코딩 결정

네트워크에서 오는 것은 글자가 아니라 바이트다. "안녕"이라는 글자도 UTF-8에서는 6바이트, 옛 한국어 인코딩(EUC-KR)에서는 4바이트다. 브라우저는 다음 단서를 순서대로 살펴 인코딩을 결정한다.

1. 파일 맨 앞의 BOM(Byte Order Mark)
2. HTTP 응답의 `Content-Type` 헤더에 있는 `charset`
3. 문서 앞부분의 `<meta charset="utf-8">` 선언 (표준은 이 선언이 문서의 처음 1024바이트 안에 있어야 한다고 정한다)
4. 그래도 없으면 브라우저의 추측

인코딩 판단이 틀리면 한글이 `ì•ˆë…•`처럼 깨진다. 그래서 `<meta charset="utf-8">`은 `<head>`의 **가장 첫 줄**에 두는 것이 관례다. 이 선언보다 앞에 긴 주석이나 다른 태그가 있으면, 브라우저가 이미 다른 인코딩으로 읽기 시작했을 수 있다.

### 문자 → 토큰: 토크나이저

토크나이저는 **상태 기계**다. 현재 상태에서 문자 하나를 읽고, 그 문자에 따라 다음 상태로 넘어가거나 토큰을 내보낸다. [HTML 표준](https://html.spec.whatwg.org/multipage/parsing.html)의 토크나이저는 여러 상태를 사용하며, 일반적인 본문에서는 "data state"에서 시작한다.

`<p class="note">Hi</p>`를 토큰화하면 대략 이렇게 흘러간다.

1. **data state**: `<`를 만나면 "태그가 시작되나 보다" → tag open state로 이동
2. **tag open state**: `p`(알파벳)를 만나면 새 시작 태그 토큰을 만들고 tag name state로 이동
3. **tag name state**: 공백을 만나면 태그 이름 `p` 확정 → 속성 이름을 기다리는 상태로 이동
4. 속성 관련 상태들을 거쳐 `class="note"`를 읽고, `>`를 만나면 **시작 태그 토큰** `{ p, class: "note" }`을 내보낸다
5. data state로 돌아와 `H`, `i`를 **문자 토큰**으로 내보낸다
6. `</p>`를 읽어 **종료 태그 토큰** `p`를 내보낸다

결과 토큰 열: `StartTag(p, class=note)` → `Character(H)` → `Character(i)` → `EndTag(p)`

중요한 점은 **토크나이저는 트리를 모른다**는 것이다. 토크나이저는 단지 글자를 잘라 줄 뿐이고, "이 `<p>`가 누구의 자식인가"는 다음 단계가 정한다.

#### 트리 구성 단계가 토크나이저의 상태를 바꾸는 경우

그런데 어떤 요소 안에서는 `<`를 태그 시작으로 보면 안 된다. 예를 들어 `<script>` 안의 `if (a < b)`의 `<`는 태그가 아니다. 그래서 트리 구성 단계는 특정 시작 태그를 만나면 **토크나이저의 상태를 직접 바꾼다**. [표준](https://html.spec.whatwg.org/multipage/parsing.html)에 따르면 `script` 시작 태그를 처리할 때 토크나이저는 "script data state"로 바뀐다.

| 요소 | 내용을 읽는 방식 | 결과 |
| --- | --- | --- |
| `script` | script data: `</script>`가 나올 때까지 거의 모든 것을 글자로 읽음 | `if (a < b)`가 태그로 오해되지 않는다 |
| `style` | RAWTEXT: 태그도, 문자 참조(`&amp;`)도 해석하지 않음 | CSS 코드가 그대로 보존된다 |
| `title`, `textarea` | RCDATA: 태그는 해석하지 않지만 `&amp;` 같은 문자 참조는 해석 | `<title><b>안내</b></title>`의 제목은 굵은 글씨가 아니라 문자 그대로 `<b>안내</b>` |

이것이 두 단계가 "한 방향 파이프라인"이 아니라 **서로 영향을 주는 구조**인 이유다.

### 토큰 → 트리: 트리 구성 단계

트리 구성 단계는 두 가지 상태를 들고 움직인다.

- **열린 요소 스택**: 지금 열려 있는 요소들. 새 노드는 보통 스택 맨 위(현재 노드)의 자식으로 들어간다.
- **삽입 모드**: 지금 문서의 어디쯤인지. 표준에는 "initial", "before html", "before head", "in head", "after head", "in body", "in table" 등의 삽입 모드가 있다.

`<html>`, `<head>`, `<body>` 태그를 전부 생략한 다음 문서를 따라가 보자.

```html
<!doctype html><title>할 일</title><p>장보기
```

| 입력 토큰 | 삽입 모드 | 파서가 하는 일 | 열린 요소 스택 (아래 → 위) |
| --- | --- | --- | --- |
| DOCTYPE | initial | DocumentType 노드를 붙이고 렌더링 모드 결정 → "before html"로 | (비어 있음) |
| StartTag `title` | before html | `html` 시작 태그가 없으므로 **`html` 요소를 직접 만들고** 토큰을 다시 처리 | html |
| StartTag `title` | before head | `head` 시작 태그가 없으므로 **`head` 요소를 만들고** 다시 처리 | html, head |
| StartTag `title` | in head | `title` 요소를 만들고, 토크나이저를 RCDATA 상태로 바꿈 | html, head, title |
| 문자 "할 일", EndTag `title` | text → in head | 텍스트 노드를 붙이고 `title`을 스택에서 꺼냄 | html, head |
| StartTag `p` | in head → after head | `p`는 head에 올 수 없으므로 `head`를 닫고, **`body`를 만들고** 다시 처리 | html, body |
| StartTag `p`, 문자 "장보기" | in body | `p`를 만들고 텍스트를 붙임 | html, body, p |
| EOF | in body | 파싱을 종료하고 문서 준비 절차로 진행 | — |

[표준](https://html.spec.whatwg.org/multipage/parsing.html)은 이처럼 "before html", "before head", "after head" 모드에서 빠진 `html`·`head`·`body` 요소를 파서가 직접 만들어 넣도록 규정한다. 그래서 **일반적인 HTML 본문 문서에서는 태그를 생략해도 파서가 `html`, `head`, `body`를 구성할 수 있다. 문서 생성 API나 파싱 진행 중의 문서까지 항상 같은 상태라고 일반화하지 않는다**.

### 오류 복구: 표준이 정한 "자동 수정" 규칙

과거에는 브라우저마다 잘못된 HTML을 다르게 처리해서, 같은 페이지가 브라우저마다 다른 DOM을 갖곤 했다. 현재 HTML 표준은 오류 처리 방법까지 알고리즘으로 정해 두었다. 그래서 같은 입력이면 주요 브라우저가 **같은 DOM**을 만든다. 대표적인 규칙들을 보자.

#### (1) 암묵적으로 닫히는 요소

`<p>`는 문단이고, 문단 안에는 문장 수준의 내용(phrasing content)만 올 수 있다. 그래서 `<p>`가 열린 상태에서 `<div>`, `<ul>`, `<h2>`, `<section>`, 또 다른 `<p>` 같은 블록 수준 시작 태그가 나오면, 파서는 **열린 `<p>`를 먼저 닫는다**.

- `<p>하나<p>둘` → `<p>하나</p><p>둘</p>` (형제)
- `<li>`도 비슷하다. `<li>사과<li>배` → 두 개의 형제 `li`
- 짝이 없는 `</p>`를 만나면 오류로 기록하고 **빈 `<p></p>`를 만든다**

#### (2) 표 안의 엉뚱한 내용: foster parenting

표(`table`)에는 caption, colgroup, 행 그룹과 행처럼 정해진 순서의 구조가 필요하며, 일반 본문을 임의로 직접 넣을 수 없다. 표 안에 일반 글자나 `<b>` 같은 요소가 직접 나오면, 파서는 그 내용을 **표 바로 앞으로 옮긴다**. 표준은 이를 "foster parenting(위탁 부모)"이라고 부른다. 또 `<table>` 바로 아래에 `<tr>`이 오면 `<tbody>`를 자동으로 만들어 끼워 넣는다.

#### (3) 엇갈린 서식 태그: adoption agency algorithm

`<b>굵게<i>둘 다</b>기울임</i>`처럼 서식 태그가 엇갈려 닫히면, 트리 구조로는 표현할 수 없다(트리에서는 부모가 자식보다 먼저 닫힐 수 없다). 표준의 "adoption agency algorithm"은 요소를 쪼개고 다시 만들어 `<b>굵게<i>둘 다</i></b><i>기울임</i>`로 재구성한다. 이 알고리즘은 `a`, `b`, `i`, `em`, `strong` 같은 서식 요소에 적용된다.

#### (4) 무시되거나 바뀌는 입력

- **중복 속성**: `<input id="a" id="b">`는 처음 나온 `id="a"`만 남는다.
- **빈 요소가 아닌 요소의 `/>`**: `<div />`의 `/`는 무시된다. 즉 `<div>`를 연 것과 같아서, 뒤의 내용이 이후 파싱 규칙과 종료 태그에 따라 이 `div` 안에 들어갈 수 있다. (JSX의 `<div />`는 HTML이 아니라 JavaScript 문법이라서 정상적으로 닫힌다. 두 언어를 혼동하지 말자.)
- **`<button>` 안의 `<button>`**: 안쪽 `button` 시작 태그를 만나면 바깥 `button`을 먼저 닫는다. 결과는 중첩이 아니라 **형제 버튼 두 개**다.
- **태그 이름·속성 이름의 대소문자**: HTML 요소에서는 소문자로 정규화된다. `<DIV CLASS="x">`의 요소 localName과 속성 이름은 `div`, `class`로 정규화된다. HTML 요소의 tagName은 보통 대문자로 노출되므로 표시 API를 구별한다.

> ⚠️ **파서가 모든 오류를 고쳐 주지는 않는다.** 위 규칙들은 "트리를 만들 수 없는" 상황을 처리하기 위한 것이다. 예를 들어 `<ul><div>항목</div></ul>`이나 `<a href="/"><button>열기</button></a>`는 HTML 콘텐츠 규칙상 올바르지 않지만 파서는 그대로 트리를 만든다. 이런 구조 오류는 DOM에 그대로 남아 접근성 문제를 일으키므로 **검사기(validator)로 따로 찾아야** 한다. 콘텐츠 규칙은 이 페이지의 Document 구조 설명에서 자세히 다룬다.

---

## 6. Document 구조: 문서의 뼈대와 콘텐츠 규칙

<callout icon="💡" color="blue_bg">
**파싱 → Document**

파싱 결과를 이해하려면 어떤 노드가 생성되고 어느 부모에 붙는지 알아야 한다. 이제 파서가 만든 트리를 문서 전체의 구조와 콘텐츠 규칙으로 읽는다.
</callout>

### 문서의 최소 골격

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>주문 내역 - 마이 쇼핑</title>
    <link rel="stylesheet" href="/styles.css">
  </head>
  <body>
    <h1>주문 내역</h1>
    <p>최근 3개월 주문입니다.</p>
  </body>
</html>
```

| 줄 | 역할 | 빠지면 생기는 일 |
| --- | --- | --- |
| `<!doctype html>` | 문서를 **no-quirks(표준) 모드**로 처리하게 한다 | quirks 모드가 되어 일부 CSS 동작이 옛 방식으로 바뀐다 |
| `<html lang="ko">` | 문서 요소. `lang`은 문서의 주 언어를 선언한다 | 스크린 리더 발음, 번역 제안, 언어별 글꼴·줄바꿈 처리가 틀릴 수 있다 |
| `<meta charset="utf-8">` | 바이트를 글자로 읽는 인코딩을 선언한다 | 서버 헤더가 없으면 한글이 깨질 수 있다 |
| `<meta name="viewport" ...>` | 모바일 브라우저에 "화면 폭을 기기 폭에 맞춰라"라고 알린다 | 모바일 브라우저가 데스크톱 폭(브라우저의 기본 레이아웃 viewport 폭)으로 가정해 축소해서 보여 준다 |
| `<title>` | 문서의 이름. 탭, 북마크, 검색 결과, 스크린 리더의 페이지 안내에 쓰인다 | 사용자가 탭·창을 구분할 수 없고, 접근성 기준(WCAG 2.4.2 Page Titled) 위반이다 |
| `<link rel="stylesheet">` | 외부 CSS를 연결한다 | — |
| `<body>` | 사용자에게 보이는 실제 내용을 담는다 | 파서가 만들어 주지만, 명시하는 편이 읽기 쉽다 |

### DOM 트리의 노드

위 문서를 파싱하면 DOM은 다음과 같은 트리가 된다. 태그 사이의 줄바꿈과 들여쓰기도 텍스트 노드가 된다는 점에 주목하자.

```text
Document
├─ DocumentType: html
└─ html
   ├─ head → meta, title, link
   └─ body → h1, p, 그 밖의 내용
```

(그림을 단순하게 하려고 `head` 안의 줄바꿈 텍스트 노드는 생략했다.) 노드는 `nodeType` 숫자로 구분할 수 있다.

| 노드 종류 | `nodeType` | 상수 | 예 |
| --- | --- | --- | --- |
| Element | 1 | `Node.ELEMENT_NODE` | `<p>`, `<body>` |
| Text | 3 | `Node.TEXT_NODE` | "주문 내역", 줄바꿈 공백 |
| Comment | 8 | `Node.COMMENT_NODE` | `<!-- 메모 -->` |
| Document | 9 | `Node.DOCUMENT_NODE` | `document` |
| DocumentType | 10 | `Node.DOCUMENT_TYPE_NODE` | `document.doctype` |
| DocumentFragment | 11 | `Node.DOCUMENT_FRAGMENT_NODE` | `<template>`의 `content` |

**속성(attribute)은 트리의 자식이 아니다.** `class="note"` 같은 속성은 요소에 붙은 정보이며, `childNodes`에 나타나지 않는다. 이 점은 흔히 헷갈린다.

### 트리 관계와 탐색 기준

트리에서는 가족 관계 용어를 쓴다.

- **부모(parent)**: 바로 위 노드. `node.parentNode`, `node.parentElement`
- **자식(child)**: 바로 아래 노드들. `node.childNodes`(모든 노드), `element.children`(요소만)
- **형제(sibling)**: 같은 부모를 가진 노드. `nextSibling` / `nextElementSibling`
- **조상(ancestor) / 자손(descendant)**: 여러 단계 위·아래까지 포함

API 이름에 `Element`가 들어가면 **요소만**, 들어가지 않으면 **텍스트·주석을 포함한 모든 노드**를 대상으로 한다고 기억하면 된다.

| 모든 노드 대상 | 요소만 대상 |
| --- | --- |
| `childNodes` | `children` |
| `firstChild` / `lastChild` | `firstElementChild` / `lastElementChild` |
| `nextSibling` / `previousSibling` | `nextElementSibling` / `previousElementSibling` |
| `parentNode` | `parentElement` |

### Document 객체의 진입점

`document`는 트리 전체에 들어가는 입구이자, 문서 단위 정보를 담은 객체다.

| 속성 | 값 |
| --- | --- |
| `document.doctype` | DocumentType 노드 (없으면 `null`) |
| `document.documentElement` | `html` 요소 |
| `document.head` / `document.body` | `head` / `body` 요소 (아직 파싱 전이면 `null`일 수 있다) |
| `document.title` | `title` 요소의 텍스트. 대입하면 탭 제목이 바뀐다 |
| `document.compatMode` | `"CSS1Compat"`(no-quirks 또는 limited-quirks) 또는 `"BackCompat"`(quirks) |
| `document.characterSet` | 실제로 적용된 인코딩. 예: `"UTF-8"` |
| `document.contentType` | 문서의 MIME 타입. 보통 `"text/html"` |

### doctype과 렌더링 모드

웹 초창기에는 브라우저마다 CSS를 다르게 해석했고, 많은 사이트가 그 "틀린" 동작에 맞춰 만들어졌다. 표준대로 바꾸면 기존 사이트가 깨지기 때문에, 브라우저는 **모드를 나누는 방법**을 택했다. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Quirks_mode_and_standards_mode)에 따르면 현재 렌더링 모드는 세 가지다.

| 모드 | 동작 |
| --- | --- |
| **Quirks mode** | Navigator 4, Internet Explorer 5 시절 동작을 흉내 낸다 |
| **Limited-quirks mode** | 아주 적은 수의 옛 동작만 남긴다 (예전 이름: almost-standards) |
| **No-quirks mode** | 현대 HTML·CSS 표준대로 동작한다 (예전 이름: full standards) |

모드는 문서 맨 앞의 doctype으로 결정된다. 파서가 "initial" 삽입 모드에서 DOCTYPE 토큰을 처리할 때 정해지므로, **doctype은 일반 요소보다 앞에 두어야** 한다. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Quirks_mode_and_standards_mode)은 `<!doctype html>`의 유일한 목적이 no-quirks 모드를 켜는 것이라고 설명한다. 대소문자는 상관없다(`<!DOCTYPE html>`도 같다).

### head: 문서에 대한 데이터

`head`에는 **메타데이터 콘텐츠**만 들어간다. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories)은 메타데이터 콘텐츠를 "문서의 표현이나 동작을 바꾸거나, 다른 문서와의 연결을 설정하거나, 그 밖의 부가 정보를 전달하는" 요소로 정의한다. 대표적으로 `title`, `meta`, `link`, `style`, `script`, `base`가 있다.

- `head`는 화면에 그려지지 않는다. 브라우저 기본 스타일시트가 `head`와 그 안의 요소를 `display: none`으로 처리한다.
- **권장 순서**: `charset`(1024바이트 안에서 최대한 빨리) → `viewport` → `title` → CSS → 스크립트. 인코딩이 먼저 확정되어야 뒤의 글자가 올바르게 읽히기 때문이다.
- **`head`에 메타데이터가 아닌 요소가 오면?** [HTML Parsing](https://html.spec.whatwg.org/multipage/parsing.html)에서 본 "in head" 규칙에 따라 파서가 `head`를 닫고 `body`를 시작할 수 있다. 뒤쪽 토큰의 종류와 삽입 모드에 따라 의도하지 않은 위치에 메타데이터가 놓일 수 있다. 소스의 들여쓰기보다 Elements의 실제 위치를 확인한다.

### lang: 문서의 언어

`<html lang="ko">`는 "이 문서의 기본 언어는 한국어"라는 선언이다. 언어 정보는 여러 곳에서 쓰인다.

- **스크린 리더**: 언어에 맞는 음성 엔진·발음 규칙을 고른다. `lang`이 틀리면 한국어를 영어 규칙으로 읽을 수 있다. WCAG 3.1.1(Language of Page)이 이를 요구한다.
- **브라우저 기능**: 번역 제안, 맞춤법 검사, 언어별 기본 글꼴 선택
- **CSS**: `:lang(ko)` 선택자, `hyphens`(자동 하이픈) 같은 언어 의존 기능
- 문서 일부가 다른 언어면 그 요소에 따로 `lang="en"`을 붙인다(WCAG 3.1.2 Language of Parts).

값은 언어 태그(BCP 47) 형식이다. 한국어는 `ko`이며, `kr`은 국가 코드(대한민국)라서 틀린 값이다. 필요하면 `ko-KR`처럼 지역을 붙인다.

### 콘텐츠 모델: 어디에 무엇을 넣을 수 있나

HTML 요소는 아무 데나 넣을 수 있는 블록이 아니다. 각 요소에는 **콘텐츠 모델**, 즉 "자식으로 무엇이 올 수 있는가"라는 규칙이 있다. 이 규칙을 짧게 표현하기 위해 요소를 **콘텐츠 카테고리**로 묶는다. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Content_categories)이 정리한 7가지 주요 카테고리는 다음과 같다.

| 카테고리 | 뜻 | 예 |
| --- | --- | --- |
| Metadata | 문서에 대한 정보, 다른 리소스 연결 | `title`, `meta`, `link`, `style` |
| Flow | `body` 안에 올 수 있는 대부분의 요소와 텍스트 | `div`, `p`, `ul`, `section`, 텍스트 |
| Sectioning | 문서에 구획을 만들고 `header`/`footer`의 범위를 정함 | `article`, `section`, `nav`, `aside` |
| Heading | 구획의 제목 | `h1`–`h6`, `hgroup` |
| Phrasing | 문장 안의 텍스트와 인라인 마크업. 이것이 모여 문단이 된다 | `span`, `a`, `strong`, `em`, `img`, 텍스트 |
| Embedded | 외부 리소스나 다른 마크업 언어를 삽입 | `img`, `video`, `iframe`, `svg` |
| Interactive | 사용자 상호작용을 위해 설계된 요소 | `a`(href 있음), `button`, `input`, `select`, `details` |

카테고리는 서로 겹친다. 예를 들어 `a`는 flow이면서 phrasing이고, `href`가 있으면 interactive이기도 하다. 이 규칙으로 앞에서 본 현상들을 설명할 수 있다.

- `p`의 콘텐츠 모델은 **phrasing content만** → `<p>` 안에 `<div>`(flow이지만 phrasing이 아님)를 넣을 수 없다. 그래서 파서가 `p`를 닫았다.
- `ul`, `ol`의 자식은 **`li`(와 스크립트 지원 요소)만** → `<ul><div>`는 규칙 위반이다. 하지만 파서는 이 경우를 고치지 않는다.
- `a`와 `button`은 **interactive content 자손을 가질 수 없다** → `<a><button>`, `<button><a>`는 규칙 위반이다.
- `a`의 콘텐츠 모델은 **transparent(투명)** → 부모가 허용하는 것이라면 `a` 안에도 넣을 수 있다. 그래서 `<div>` 안의 `<a>`는 `<h3>`, `<p>` 같은 블록 요소를 감쌀 수 있다(카드 전체를 링크로 만드는 패턴이 유효한 이유).

> 💡 **파서가 고치는 규칙 위반 vs 고치지 않는 규칙 위반**
>
> 파서는 "트리를 만들 수 없거나, 역사적으로 특별 처리가 필요했던" 경우만 구조를 바꾼다(`p` 자동 닫힘, 표, 중첩 `button`/`a` 등). 대부분의 콘텐츠 모델 위반은 DOM에 **그대로 남는다**. 그래서 화면이 정상이어도 [Nu HTML Checker](https://validator.w3.org/nu/)로 검사해야 한다.

---

## 7. Semantic HTML: 구조에 의미와 기본 동작 부여하기

<callout icon="💡" color="blue_bg">
**Document → Semantic HTML**

트리 형태가 만들어졌다는 사실만으로 목적에 맞는 문서가 되는 것은 아니다. 같은 위치에 div 대신 nav나 button을 선택하면 의미·탐색·기본 조작이 달라진다.
</callout>

### DOM에서 접근성 트리로

브라우저는 DOM을 만든 뒤, 각 노드를 **역할(role)**, **이름(name)**, **상태(state)** 정보로 바꾼 접근성 트리를 만든다. [HTML-AAM](https://www.w3.org/TR/html-aam-1.0/)은 HTML 요소와 속성을 플랫폼 접근성 API에 어떻게 매핑할지 정의해, 브라우저마다 같은 정보를 노출하도록 한다.

| HTML | 암묵적 역할 | 스크린 리더 안내 예 |
| --- | --- | --- |
| `<button>저장</button>` | button | "저장, 버튼" |
| `<a href="/help">도움말</a>` | link | "도움말, 링크" |
| `<a>도움말</a>` (href 없음) | generic | "도움말" (링크가 아님) |
| `<ul>`  • `<li>` 3개 | list / listitem | "목록, 항목 3개" |
| `<nav>` | navigation | "내비게이션" |
| `<h2>배송 정보</h2>` | heading (level 2) | "배송 정보, 제목 수준 2" |
| `<div>`, `<span>` | generic | 내용 글자만 읽는다 |

(위 역할 매핑 중 `button`, `a`, `ul`, `nav`는 [HTML-AAM](https://www.w3.org/TR/html-aam-1.0/)의 매핑 표를 따른다. 실제 안내 문구는 스크린 리더와 언어 설정에 따라 다르다.)

\[미리보기: Accessible Name\]

"저장, 버튼"의 "저장"처럼 보조 기술이 요소를 부르는 이름을 accessible name이라고 하며, 내용 텍스트·`label`·`aria-label` 등 정해진 순서로 계산된다. 계산 규칙은 바로 다음 학습인 "Form / Input / Accessibility 기초"에서 자세히 다룬다.

### 기본 동작: 의미가 공짜로 주는 기능

의미 요소를 쓰면 브라우저가 **해당 요소의 기본 조작과 상태를 제공**한다. 같은 동작을 임의의 요소와 JavaScript로 다시 구현할 필요를 줄일 수 있다.

**`<button>`**

- 키보드 포커스를 받는다(Tab 순서에 포함).
- Enter, Space로 실행하면 브라우저가 click 이벤트를 발생시킨다.
- `disabled`이면 클릭·포커스가 막히고 "비활성" 상태가 전달된다.
- 일반적인 폼의 버튼에서 type을 생략하면 제출 동작을 할 수 있다. 제출은 `type="submit"`, 보조 동작은 `type="button"`으로 의도를 명시한다.

**`<a href>`**

- 포커스, Enter로 이동
- 새 탭으로 열기(Ctrl/Cmd+클릭, 가운데 클릭), 링크 주소 복사, 방문 여부 표시
- 브라우저 뒤로 가기·앞으로 가기와 자연스럽게 연결

**링크 vs 버튼 선택 기준**

- **다른 위치(URL)로 이동**한다 → `<a href>`
- **현재 페이지에서 동작을 실행**한다(저장, 열기, 삭제, 토글) → `<button>`
- "버튼처럼 보이는 링크", "링크처럼 보이는 버튼"은 CSS로 해결한다. 모양이 아니라 **동작의 종류**로 요소를 고른다.

### 문서의 뼈대: 랜드마크

스크린 리더 사용자는 페이지 전체를 처음부터 끝까지 듣지 않는다. 랜드마크 목록을 불러 "주요 내용"으로 바로 건너뛰거나, 내비게이션만 골라 이동한다. [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/practices/landmark-regions/)가 정리한 HTML 요소와 랜드마크의 대응은 다음과 같다.

| HTML 요소 | 랜드마크 역할 | 조건 |
| --- | --- | --- |
| `header` | banner | `body` 문맥일 때만. `article`, `aside`, `main`, `nav`, `section` 안에 있으면 랜드마크가 아니다 |
| `nav` | navigation | 항상 |
| `main` | main | 항상. 페이지의 주요 내용은 하나 |
| `aside` | complementary | 항상 |
| `footer` | contentinfo | `body` 문맥일 때만. `header`와 같은 조건 |
| `section` | region | **접근 가능한 이름이 있을 때만** (`aria-labelledby`, `aria-label` 등) |
| `form` | form | 접근 가능한 이름이 있을 때만 |
| `search` | search | 항상 |

조건에 주목하자. `article` 안의 `header`는 "그 글의 머리말"일 뿐 페이지 전체의 banner가 아니다. 이름 없는 `section`은 랜드마크가 되지 않는다. 같은 종류의 랜드마크(예: `nav` 두 개)가 여러 개면 "주 메뉴", "푸터 메뉴"처럼 이름을 붙여 구분해야 사용자가 헷갈리지 않는다.

### 헤딩: 크기가 아니라 제목 관계

헤딩은 문서 내용의 계층을 h1–h6으로 표현한다. 페이지 중심 제목 아래의 큰 구획은 h2, 그 안의 하위 구획은 h3처럼 논리적인 관계를 직접 드러낸다. 구획 요소를 중첩한 것만으로 스크린 리더의 제목 수준이 원하는 대로 자동 보정된다고 가정하지 않는다.

문서 전체의 목적을 설명하는 중심 제목을 먼저 정하고 주제 구획마다 적절한 제목을 붙인다. 제목을 작게 보이게 하려고 h2 대신 h5를 고르지 않는다. 시각적 크기는 CSS로 조절한다. 최신 HTML의 추가적인 제목 기능을 사용하더라도 대상 브라우저와 보조 기술의 실제 노출을 확인한다.

### 구획 요소 선택하기: article, section, aside, nav, div

주요 이동 링크 묶음인지, 독립 콘텐츠인지, 보조 정보인지, 제목을 붙일 주제 구획인지 순서대로 판단한다. 이에 해당하지 않는 배치용 묶음에는 div를 사용할 수 있다.

- **`article`**: 독립적으로 배포·재사용해도 의미가 통하는 단위. 블로그 글, 뉴스 기사, 댓글, 상품 카드.
- **`section`**: 주제별로 묶인 구획. 보통 헤딩을 가진다. "스타일을 주려고 감싼 상자"라면 `section`이 아니라 `div`다.
- **`aside`**: 주 내용과 간접적으로 관련된 내용. 사이드바, 관련 글, 용어 설명 박스.
- **`div`**: 의미 없는 컨테이너. Flex/Grid 배치를 위한 래퍼처럼 **의미가 필요 없을 때는 `div`가 정답**이다. 시맨틱 HTML은 `div`를 쓰지 말자는 것이 아니라, **의미가 있는 곳에 의미 요소를 쓰자**는 것이다.

### 텍스트 수준의 의미

| 요소 | 의미 | 헷갈리는 짝 |
| --- | --- | --- |
| `strong` | 중요함, 심각함, 긴급함 | `b`: 중요하지 않지만 시각적으로 구분할 키워드 (예: 제품명) |
| `em` | 강세. 문장의 뜻을 바꾸는 강조 | `i`: 다른 목소리·분위기의 텍스트 (외래어, 학명, 생각) |
| `time` | 날짜·시간. `datetime` 속성으로 기계가 읽을 수 있는 값 제공 | 그냥 텍스트 "어제" |
| `code`, `kbd` | 코드 조각 / 사용자가 입력하는 키 | `span class="mono"` |
| `blockquote`, `q`, `cite` | 긴 인용 / 짧은 인라인 인용 / 작품 제목 | 들여쓰기용 `blockquote` 남용 |
| `abbr` | 약어 | — |

`b`와 `i`는 폐기된 요소가 아니다. HTML5에서 의미가 재정의되었을 뿐이다. 단순히 굵게·기울게 보이고 싶은 것이라면 CSS(`font-weight`, `font-style`)를 쓴다.

### 목록, 표, 이미지

- **목록**: 순서 없는 묶음은 `ul`, 순서가 중요하면 `ol`, 용어-설명 쌍은 `dl`/`dt`/`dd`. 스크린 리더는 "목록, 항목 5개"처럼 개수를 알려 주므로, 사용자가 얼마나 긴지 미리 알 수 있다. 내비게이션 메뉴도 보통 `nav > ul > li > a` 구조다.
- **표**: **표 형식의 데이터**에만 `table`을 쓴다. `caption`(표 제목), `th`(머리글 칸)와 `scope="col"`/`scope="row"`를 쓰면 스크린 리더가 칸을 이동할 때 "열 머리글: 가격, 12,000원"처럼 맥락을 읽어 준다. 레이아웃을 위해 `table`을 쓰지 않는다(그것은 CSS Grid의 역할).
- **이미지**: `img`의 `alt`는 이미지를 대신하는 텍스트다. 정보를 전달하는 이미지에는 그 정보를 쓰고, 장식용 이미지는 `alt=""`로 비워 보조 기술이 건너뛰게 한다. `alt` 속성 자체를 빼면 일부 스크린 리더가 파일 이름을 읽는다. 이미지와 설명을 묶을 때는 `figure`/`figcaption`을 쓴다.

### ARIA는 언제 쓰는가

[Using ARIA](https://www.w3.org/TR/using-aria/)의 첫 번째 규칙은 이렇다. 필요한 의미와 동작이 이미 내장된 네이티브 HTML 요소나 속성을 쓸 수 있다면, 다른 요소에 ARIA 역할·상태·속성을 붙여 바꾸는 대신 **네이티브 요소를 써라**. 두 번째 규칙은 **꼭 필요하지 않다면 네이티브 의미를 바꾸지 말라**는 것이다.

이유는 ARIA가 **의미만 바꾸고 동작은 하나도 추가하지 않기 때문**이다. `<div role="button">`은 스크린 리더에게 "버튼"이라고 말하지만, 포커스도, Enter/Space 실행도, `disabled`도 생기지 않는다. 결국 "버튼이라고 소개됐는데 눌리지 않는" 더 나쁜 상태가 될 수 있다. ARIA는 탭 패널, 콤보박스처럼 **HTML에 대응 요소가 없는 위젯**을 만들 때 쓰는 도구다.

\[미리보기: ARIA 속성과 위젯 패턴\]

`aria-expanded`, `aria-controls` 같은 상태·관계 속성과 복합 위젯의 키보드 패턴은 "Form / Input / Accessibility 기초"와 이후 접근성 심화 학습에서 다룬다.

---

## 8. 코드 예제와 DOM·화면 변화 해설

아래 예제는 모두 **브라우저 DevTools의 Console 탭**에 그대로 붙여 넣어 실행할 수 있다. 먼저 DOM 트리를 보기 쉽게 출력하는 도우미 함수를 만든다.

```javascript
// 요소와 (공백이 아닌) 텍스트 노드만 들여쓰기로 출력한다
function printTree(node, depth = 0) {
  const pad = '  '.repeat(depth);
  if (node.nodeType === Node.ELEMENT_NODE) {
    console.log(pad + node.nodeName.toLowerCase());
  } else if (node.nodeType === Node.TEXT_NODE && node.data.trim()) {
    console.log(pad + '#text "' + node.data.trim() + '"');
  }
  for (const child of node.childNodes) printTree(child, depth + 1);
}

// HTML 문자열을 "새 문서"로 파싱한다 (현재 페이지에는 영향 없음)
const parse = (html) => new DOMParser().parseFromString(html, 'text/html');
```

### 생략된 html / head / body

```javascript
const doc = parse('<!doctype html><title>할 일</title><p>장보기');
printTree(doc.documentElement);
```

```txt
html
  head
    title
      #text "할 일"
  body
    p
      #text "장보기"
```

**왜 이렇게 되는가**: 앞서 본 트리 구성 과정처럼 파서가 "before html", "before head", "after head" 모드에서 빠진 요소를 직접 만들었다. 소스에는 `body`가 없지만 `doc.body`는 `null`이 아니다.

### 문단 안에 div를 넣으면

```javascript
const doc = parse('<p>소개<div class="box">상자</div>마무리</p>');
console.log(doc.body.innerHTML);
// <p>소개</p><div class="box">상자</div>마무리<p></p>
```

**단계별 해설**

1. `<p>`가 열리고 "소개"가 들어간다.
2. `<div>` 시작 태그 → `p` 안에는 블록 요소가 올 수 없으므로 파서가 **`p`를 먼저 닫는다**. `div`는 `body`의 자식이 된다.
3. "상자"는 `div` 안, `</div>`로 `div`가 닫힌다.
4. "마무리"는 이제 `body`에 직접 들어간다.
5. `</p>`를 만났지만 열린 `p`가 없다 → 오류로 기록하고 **빈 `<p></p>`를 만든다**.

**화면과 코드에 미치는 영향**

- CSS `p .box { ... }`는 아무것도 선택하지 못한다. `.box`는 `p`의 자손이 아니다.
- `p { color: gray }`는 "소개"에만 적용된다. "마무리"는 회색이 아니다.
- `doc.querySelector('p').textContent`는 "소개"뿐이다.
- 문서에 빈 문단이 하나 생기며, 일부 스크린 리더는 이를 빈 항목으로 탐색할 수 있다.

### 표 안의 글자와 자동 tbody

```javascript
const doc = parse('<table>메모<tr><td>A</td></tr></table>');
console.log(doc.body.innerHTML);
// 메모<table><tbody><tr><td>A</td></tr></tbody></table>
```

**왜 이렇게 되는가**

- "메모"는 표 안에 올 수 없는 글자다 → foster parenting으로 **표 앞으로 이동**했다. 화면에서도 "메모"가 표 위에 보인다.
- `<table>` 바로 아래의 `<tr>` → 파서가 `<tbody>`를 만들어 끼워 넣었다.

**React와의 연결**: 클라이언트에서 React가 DOM API로 `<table><tr>`을 만들면 HTML 파서를 거치지 않으므로 `tbody`가 **생기지 않는다**. 반면 서버에서 같은 컴포넌트를 HTML 문자열로 만들어 보내면 브라우저 파서가 `tbody`를 넣는다. 두 결과가 달라지는 것이다. React가 개발 모드에서 "`<tr>`은 `<table>`의 자식이 될 수 없다"는 식의 중첩 경고를 출력하는 이유가 여기에 있다.

\[미리보기: SSR / Hydration\]

서버가 만든 HTML 위에 React가 이벤트와 상태를 연결하는 과정을 hydration이라고 하며, 이때 서버 DOM과 클라이언트가 기대하는 구조가 다르면 불일치 오류가 난다. 렌더링 전략은 이후 React 아키텍처와 React × Backend 학습에서 다룬다.

### 엇갈린 서식 태그

```javascript
const doc = parse('<b>굵게<i>둘 다</b>기울임</i>');
console.log(doc.body.innerHTML);
// <b>굵게<i>둘 다</i></b><i>기울임</i>
```

**왜 이렇게 되는가**: `</b>`를 만났을 때 `i`가 아직 열려 있다. adoption agency algorithm이 `i`를 일단 닫고, `b` 바깥에 **새 `i` 요소를 다시 만들어** "기울임"을 담는다. 그 결과 DOM에는 `i`가 두 개 있다. `document.querySelectorAll('i').length`는 1이 아니라 2다.



### "div 수프"를 시맨틱 구조로 바꾸기

**Before**: 모든 것이 `div`

```html
<div class="header">
  <div class="logo">마이 블로그</div>
  <div class="menu">
    <div onclick="location.href='/'">홈</div>
    <div onclick="location.href='/posts'">글 목록</div>
  </div>
</div>
<div class="content">
  <div class="post">
    <div class="title">HTML 파싱 정리</div>
    <div class="date">2026.10.17</div>
    <div class="body">파서는 토큰을 만들고...</div>
  </div>
</div>
<div class="sidebar">관련 글...</div>
<div class="footer">© 2026 마이 블로그</div>
```

**After**: 의미 요소 사용

```html
<header>
  <a href="/" class="logo">마이 블로그</a>
  <nav aria-label="주 메뉴">
    <ul>
      <li><a href="/">홈</a></li>
      <li><a href="/posts">글 목록</a></li>
    </ul>
  </nav>
</header>
<main>
  <article>
    <h1>HTML 파싱 정리</h1>
    <p><time datetime="2026-10-17">2026.10.17</time></p>
    <p>파서는 토큰을 만들고...</p>
  </article>
</main>
<aside aria-labelledby="related-title">
  <h2 id="related-title">관련 글</h2>
  ...
</aside>
<footer>© 2026 마이 블로그</footer>
```

**화면 변화**: CSS를 같게 맞추면 화면은 거의 같다. 바뀐 것은 **DOM의 요소 종류**와, 그로부터 만들어지는 **접근성 트리**다.

| 스크린 리더 기능 | Before | After |
| --- | --- | --- |
| 랜드마크 목록 | (없음) | 배너, 내비게이션 "주 메뉴", 기본(main), 보조 "관련 글", 콘텐츠 정보 |
| 헤딩 목록 | (없음) | 제목 1 "HTML 파싱 정리", 제목 2 "관련 글" |
| 링크 목록 | (없음 — 클릭 가능한 div는 링크가 아니다) | "마이 블로그", "홈", "글 목록" |
| 메뉴 안내 | 글자만 연속으로 읽음 | "목록, 항목 2개" |
| 키보드 | 메뉴에 Tab으로 갈 수 없다 | Tab으로 모든 링크에 이동, Enter로 이동 |

`header`와 `footer`가 `body`의 직계 영역에 있으므로 banner와 contentinfo가 되었다. 만약 `article` 안에 `<header>`를 넣는다면 그것은 글의 머리말일 뿐 banner가 아니다.

---

## 9. 흔한 오해·예외·주의점

- **"브라우저가 알아서 대충 고친다"** → 대충이 아니다. 표준이 정한 알고리즘대로 고치며, 결과는 예측 가능하다. 다만 그 결과가 **내 의도와 같다는 보장은 없다**.
- **"화면이 깨지지 않으면 HTML이 올바르다"** → 예제 2처럼 화면은 멀쩡해 보여도 DOM 구조가 바뀌어 CSS·JS·접근성에 영향을 줄 수 있다.
- **"Elements 패널 = 내 HTML 소스"** → Elements 패널은 파싱 결과(그리고 이후 JS가 바꾼 결과)를 보여 준다. 원본은 "소스 보기"나 Network 탭의 응답 본문에서 확인한다.
- **"`<div />`로 닫을 수 있다"** → HTML에서 `/`는 빈 요소가 아니면 무시된다. JSX와 혼동하지 말자.
- **"태그를 생략하면 잘못된 HTML이다"** → HTML 표준은 일부 태그(`html`, `head`, `body`의 시작·종료 태그, `</li>`, `</p>` 등)의 생략을 특정 조건에서 **허용**한다. 하지만 팀 코드에서는 가독성과 실수 방지를 위해 명시적으로 쓰는 편이 낫다.
- **"innerHTML로 넣은 script가 실행되지 않으니 안전하다"** → 이벤트 핸들러 속성은 실행된다(예제 5).
- **적용 범위 주의 — XHTML**: 문서를 `application/xhtml+xml`로 전송하면 HTML 파서가 아니라 **XML 파서**가 쓰인다. XML 파서는 오류를 고쳐 주지 않고 문서 처리를 중단하는 엄격한 방식이다. 이 페이지의 오류 복구 규칙은 `text/html` 문서에만 해당한다.
- **적용 범위 주의 — 렌더링 모드**: 일반 HTML 문서에 적절한 doctype이 없으면 quirks 모드가 될 수 있고 일부 CSS·파싱 동작에 영향을 준다. doctype 앞의 공백·주석이 있다는 사실만으로 무조건 잘못된 문서가 되는 것은 아니며, 일반 요소보다 앞에 선언하고 실제 compatMode를 확인한다.

---

## 10. 실무 적용·접근성·성능·디버깅

### 요소 선택 체크리스트

- [ ] 이동이면 `a href`, 동작이면 `button`인가 (`form` 안의 `button`은 `type`을 명시했는가)
- [ ] 페이지에 `main`이 하나 있고, 머리글·내비게이션·바닥글이 랜드마크로 표현되는가
- [ ] 같은 종류의 랜드마크가 여러 개라면 이름으로 구분했는가
- [ ] `h1`이 하나이고 헤딩 단계를 건너뛰지 않는가 (크기는 CSS로)
- [ ] 항목 묶음은 목록, 표 형식 데이터는 `table` + `th`로 표현했는가
- [ ] 정보 이미지에는 의미 있는 `alt`, 장식 이미지에는 `alt=""`가 있는가
- [ ] ARIA를 쓰기 전에 같은 일을 하는 네이티브 요소가 없는지 확인했는가

### 디버깅과 검증 방법

1. **키보드만으로 사용해 보기**: 마우스를 치우고 Tab, Shift+Tab, Enter, Space로 모든 기능을 쓸 수 있는지 확인한다. 가장 빠르고 효과 좋은 검사다.
2. **DevTools 접근성 패널**: Chrome의 Elements → Accessibility 탭에서 선택한 요소의 역할·이름·상태를 볼 수 있고, 전체 접근성 트리 보기를 켤 수도 있다. Firefox에는 Accessibility Inspector가 있다. "내가 의도한 역할이 실제로 노출되는가"를 여기서 확인한다.
3. **자동 검사**: Lighthouse의 접근성 감사, axe DevTools 같은 확장 프로그램으로 헤딩 순서, 랜드마크, 이름 없는 버튼 등을 찾는다. 자동 검사는 모든 문제를 찾지 못하므로 1·2번과 함께 쓴다.
4. **스크린 리더로 짧게 확인**: macOS VoiceOver(Cmd+F5)의 로터(VO+U)로 헤딩·랜드마크·링크 목록을 확인해 본다. Windows에서는 무료 스크린 리더 NVDA를 쓸 수 있다.
5. **HTML 검사기**: [Nu HTML Checker](https://validator.w3.org/nu/)로 `ul` 안의 `div`, `a` 안의 `button` 같은 콘텐츠 모델 위반을 찾는다.

### 성능 관점

- **네이티브 기본 기능을 다시 구현하는 코드를 줄인다**: 예제 1처럼 `div` 버튼은 이벤트 리스너와 상태 관리 코드가 필요하다. 네이티브 `button`은 이 코드를 내려받고 실행할 필요가 없다. 실제 성능 차이는 구현과 사용량에 따라 달라지므로 기능을 동등하게 맞춘 뒤 측정한다.
- **불필요한 래퍼 줄이기**: 의미 요소 자체가 렌더링을 빠르게 하지는 않는다. 하지만 "div 수프"를 정리하면서 래퍼가 줄면 DOM 노드 수가 줄어 스타일 계산·레이아웃 비용이 줄어든다. 측정은 Lighthouse DOM 크기 진단과 Performance 패널로 한다.
- **비용**: 시맨틱 마크업을 위해 디자인 시스템 컴포넌트의 API를 바꿔야 할 수 있다. 하지만 나중에 접근성을 덧붙이는 비용이 처음부터 올바른 요소를 쓰는 비용보다 훨씬 크다.



### 소스·DOM·표현·접근성을 나누어 확인하기

1. Network 또는 View Source로 실제 받은 HTML과 Content-Type·인코딩을 확인한다.
2. Elements에서 파서가 만든 부모·자식 관계와 자동 삽입 노드를 확인한다.
3. HTML 검사기로 파서가 복구하지 않은 콘텐츠 모델 위반을 찾는다.
4. 접근성 패널에서 역할·이름·헤딩·랜드마크를 확인한다.
5. 키보드와 실제 보조 기술로 목적한 탐색과 실행이 가능한지 확인한다.

파서가 만든 tbody 때문에 자식 선택자가 달라질 수도 있고, 잘못된 제목 단계 때문에 탐색이 어려울 수도 있다. 둘을 모두 “CSS가 이상하다”로 처리하지 않고 문제가 생긴 계층을 좁힌다.

---

## 11. 후속 학습 미리보기

\[미리보기: DOM / Node / Element / DOM Manipulation\]

문자열 파싱과 createElement로 노드를 직접 만드는 방법은 같은 경로가 아니다. DOM API의 생성·이동·수정은 1–3주 DOM 학습에서 다룬다.

\[미리보기: script / async / defer / module\]

스크립트 실행은 HTML 파싱의 진행과 DOM 접근 가능 시점을 바꿀 수 있다. 로딩 속성별 실행 순서는 1–3주 script 학습에서 다룬다.

---

## 12. 핵심 정리

<callout icon="✅" color="green_bg">
HTML 파서는 입력을 트리로 만들고, 문서 구조는 그 트리의 큰 관계를 정하며, 시맨틱 HTML은 각 부분의 의미와 기본 동작을 표현한다. 이 세 과정은 하나의 문서를 만드는 연속된 흐름이다. 소스, 실제 DOM, 화면, 접근성 정보는 서로 연결되어 있지만 동일한 결과물은 아니다.
</callout>

---

## 출처

- [WHATWG — HTML 파싱](https://html.spec.whatwg.org/multipage/parsing.html)
- [WHATWG — 문서 요소와 메타데이터](https://html.spec.whatwg.org/multipage/semantics.html)
- [WHATWG — 구획과 제목](https://html.spec.whatwg.org/multipage/sections.html)
- [W3C WAI — Page Structure](https://www.w3.org/WAI/tutorials/page-structure/)
- [WHATWG — 콘텐츠 모델과 Document](https://html.spec.whatwg.org/multipage/dom.html)
- [W3C — HTML Accessibility API Mappings](https://www.w3.org/TR/html-aam-1.0/)

---

## 13. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. `p` 안에 `div`를 작성한 소스와 Elements의 트리가 다르다. 브라우저가 어떤 단계에서 구조를 바꾸는지, 무엇을 비교하여 확인할지 설명하라.
2. 본문의 `h1`이 있어도 탭 제목이 원하는 이름이 아니다. 문서 구조의 어느 부분을 확인해야 하는가?
3. 카드의 이동 기능을 클릭 가능한 `div`로 만들었다. 적절한 요소를 고르고 키보드·보조 기술 관점의 차이를 설명하라.
4. 글자 크기가 같게 보이도록 스타일링한 `div`와 `h2`를 화면만으로 평가하면 무엇을 놓치는가? 검증 방법을 제시하라.
