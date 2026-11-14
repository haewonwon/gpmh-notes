---
track: B
notion: https://app.notion.com/p/415178c4afac81fb9d88ff6d3ad4f228
---

# DOM / Node / Element / DOM Manipulation


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · DOM / Node / Element / DOM Manipulation

문서 모델과 노드 종류를 먼저 구별한 뒤 조회·생성·이동·수정·제거와 화면·접근성의 변화를 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- Node·Element·Text·Document의 관계와 탐색 범위를 설명할 수 있다.
- 정적·live 조회 결과와 객체 참조의 변화를 구별할 수 있다.
- 생성·이동·복제·교체 API를 목적에 맞게 사용하고 속성·프로퍼티를 구별할 수 있다.
- 텍스트와 HTML 삽입의 안전성·파싱 문맥 차이를 설명할 수 있다.
- 변경 후 DOM·화면·포커스와 실제 처리 비용을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

HTML 파싱이 초기 문서를 만들었다면 사용자와 상호작용하는 페이지는 그 문서를 계속 바꾼다. DOM은 이 작업을 가능하게 하는 모델과 API다. React를 쓰더라도 최종 화면의 요소 관계와 잘못된 변경을 이해하려면 DOM의 기본 단위를 알아야 한다.

---

## 3. 관련 계층과 역할 구분

HTML 파서는 문자열을 초기 DOM으로 바꾸고 DOM API는 그 객체 트리를 읽고 바꾸는 접점을 제공한다. JavaScript는 이 API를 호출하는 언어다. 브라우저는 변경된 트리와 CSS를 바탕으로 화면과 접근성 정보를 갱신한다.

문서 트리가 변경된 시점과 픽셀이 실제로 표시된 시점을 구별한다. React 같은 라이브러리는 DOM 변경을 대신 관리할 수 있지만 최종 요소의 구조·기본 동작·접근성 요구는 그대로 남는다.

---

## 4. 문서를 객체의 관계로 본다

DOM은 Document Object Model의 약자다. `document`는 현재 문서를 나타내는 객체이며 트리의 시작점이다. Node는 트리를 구성하는 공통 단위다. Element는 Node의 한 종류이고, Text·Comment·Document 등도 Node이므로 모든 Node가 Element는 아니다. [DOM 표준](https://dom.spec.whatwg.org/#node-trees)이 이 구조를 정의한다.

```html
<p id="message">안녕 <strong>DOM</strong></p>
```

`p`에는 Text 노드와 `strong` 요소가 자식으로 있다. `strong` 안에는 다시 Text 노드가 있다. HTML의 줄바꿈·공백도 Text 노드가 될 수 있다. `childNodes`는 텍스트·주석까지 포함하지만 `children`은 요소만 대상으로 한다. 그래서 `firstChild`와 `firstElementChild`의 결과가 다를 수 있다.

---

## 5. Node와 Element의 관계를 코드로 확인하기

### 문서의 모든 대상이 태그 요소인 것은 아니다

```html
<div id="sample">
  안녕
  <!-- 설명 -->
  <strong>DOM</strong>
</div>
```

sample의 children에는 strong만 있지만 childNodes에는 공백과 글자를 가진 Text, Comment, strong 요소가 있다. nodeType은 종류를, nodeName은 노드에 맞는 이름을 보여준다. Element에는 tagName·classList 같은 요소 기능이 있지만 Text에 같은 기능이 있다고 가정하면 안 된다.

```javascript
const sample = document.querySelector('#sample');
console.log(sample instanceof Node); // true
console.log(sample instanceof Element); // true
for (const node of sample.childNodes) {
  console.log(node.nodeType, node.nodeName);
}
```

이 실험은 위 마크업을 문서에 넣은 뒤 콘솔에서 실행한다. `document`도 Node이지만 Element는 아니다. `document.documentElement`는 html 요소로, 문서 객체와 루트 요소는 서로 다른 객체다.

### 트리 관계와 조회 범위를 구별하기

부모는 바로 한 단계 위, 조상은 여러 단계 위까지 포함한다. 자식과 자손도 같은 차이가 있다. `element.querySelector`는 해당 요소 아래의 범위를 조회하고, `closest`는 자신부터 조상 방향으로 조건을 찾는다. 현재 요소 자체가 조건에 맞으면 closest는 자신을 반환할 수 있다.

선택자의 `>`는 직접 자식, 공백은 자손 관계다. 파서가 tbody를 삽입한 표에서 `table > tr`이 일치하지 않는 경우처럼, 선택자 판단은 원본 문자열이 아니라 실제 트리를 기준으로 한다.

---

## 6. 조회 결과의 종류와 수명

| API | 결과 | 변경 이후 자동 반영 |
| --- | --- | --- |
| `querySelector` | 첫 Element 또는 null | 객체 참조 하나를 반환 |
| `querySelectorAll` | 정적 NodeList | 조회 당시 구성 유지 |
| `getElementsByClassName` | live HTMLCollection | 조건에 맞는 현재 구성 반영 |
| `children` | live HTMLCollection | 현재 요소 자식 반영 |
| `childNodes` | live NodeList | 현재 모든 자식 노드 반영 |

정적 목록의 “정적”은 목록 구성에 대한 말이다. 목록에 든 요소 객체 자체가 변경 불가능해지는 것은 아니다. 조회 뒤 문구를 바꾸면 저장한 목록에서도 같은 객체의 새 문구를 읽을 수 있다.

```javascript
const items = document.querySelectorAll('#notes li');
if (items.length > 0) {
  items[0].textContent = '변경된 제목';
  console.log(items[0].textContent);
}
```

live 컬렉션을 앞에서부터 제거하면 다음 항목이 앞으로 당겨져 인덱스를 건너뛸 수 있다. `Array.from(collection)`으로 목록을 고정한 뒤 순회하거나, 첫 항목을 반복 제거하는 등 의도에 맞는 방법을 사용한다.

---

## 7. 찾기에서 변경까지의 API 흐름

`querySelector`는 첫 일치 요소 또는 `null`, `querySelectorAll`은 일치 요소의 정적인 NodeList를 반환한다. 한 번 조회한 정적인 목록에 새 요소가 자동으로 추가되지는 않는다. 반면 `children`이나 `getElementsByClassName`처럼 현재 구조에 반응하는 live 컬렉션도 있다. NodeList라는 이름만 보고 항상 정적이라고 판단해서는 안 된다.

요소를 찾은 다음 `createElement`로 새 요소를 만들고, `textContent`·`classList`·속성·프로퍼티로 상태를 지정한 뒤 `append`, `prepend`, `before`, `replaceWith`, `remove` 등으로 트리에 반영한다. JavaScript 문법을 실행하는 것은 엔진이고, 이 문서 조작 기능을 제공하는 것은 브라우저의 DOM API다.

---

## 8. 예제: 목록 항목을 만들고 이동하고 제거하기

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>DOM 실험</title></head>
<body>
  <h1>오늘의 학습</h1>
  <ul id="notes"><li>HTML</li></ul>
  <p id="status" role="status"></p>
  <script>
    const list = document.querySelector('#notes');
    const status = document.querySelector('#status');
    const before = list.querySelectorAll('li');
    const item = document.createElement('li');

    item.textContent = '<strong>DOM</strong>';
    item.classList.add('new-note');
    list.append(item);

    console.log(before.length); // 1: 조회 당시의 정적인 목록
    console.log(list.children.length); // 2: 현재 요소 자식
    status.textContent = '학습 항목이 추가되었습니다.';

    list.prepend(item); // 복제하지 않고 같은 노드를 첫 위치로 이동
    console.log(list.children.length); // 여전히 2
  </script>
</body>
</html>
```

`item`은 생성 직후에는 문서에 연결되지 않은 요소다. append하면 트리에 들어가고 목록 항목이 화면에 보일 수 있게 된다. `textContent`에 넣은 태그 모양의 문자열은 굵은 글씨 요소가 아니라 문자 그대로 표시된다. prepend는 같은 객체를 이동하므로 항목 수가 증가하지 않는다. 복제하려면 별도로 `cloneNode` 등을 써야 한다.

콘솔에서 `item.remove()`를 실행하면 트리에서 빠진다. 변수는 여전히 그 객체를 가리킬 수 있으므로, 트리에서 제거하는 것과 메모리에서 즉시 사라지는 것은 다르다. 제거한 컨트롤에 포커스가 있었다면 사용자가 다음에 어디서 작업할지도 고려해야 한다.

---

## 9. 생성·이동·복제·교체의 차이

### 새 노드와 기존 노드 이동

createElement는 객체를 만들 뿐 문서 안에 붙이지 않는다. append에 기존 노드를 넘기면 같은 객체의 위치가 바뀐다. 한 노드는 일반 트리에서 두 부모의 자식으로 동시에 존재할 수 없다. 다른 위치에도 같은 내용을 두려면 새 노드를 만들거나 복제한다.

### cloneNode가 복사하는 것과 남는 조건

`cloneNode(true)`는 하위 노드까지 복제한다. id 속성까지 복제될 수 있으므로 같은 문서에 붙일 때 중복 id를 정리해야 한다. 이후 배우는 addEventListener로 등록한 리스너 같은 객체의 모든 실행 상태까지 복제하는 API는 아니다.

```javascript
const original = document.querySelector('#notes li');
if (original) {
  const copy = original.cloneNode(true);
  copy.removeAttribute('id');
  copy.textContent = '복제한 항목';
  original.after(copy);
}
```

### replaceChildren와 DocumentFragment

replaceChildren는 기존 자식들을 새 구성으로 교체한다. DocumentFragment는 문서에 직접 연결되지 않은 자식 묶음을 구성할 수 있는 컨테이너다. fragment를 append하면 fragment 객체 자체가 일반 요소처럼 들어가는 것이 아니라 그 자식들이 옮겨진다.

```javascript
const list = document.querySelector('#notes');
const fragment = document.createDocumentFragment();
for (const title of ['HTML', 'DOM', 'Events']) {
  const item = document.createElement('li');
  item.textContent = title;
  fragment.append(item);
}
list.replaceChildren(fragment);
console.log(fragment.childNodes.length); // 0: 자식들이 list로 이동
```

fragment는 변경 구성을 읽기 쉽게 묶는 데 도움이 된다. 실제 성능 이점은 엔진·문서·호출 패턴에 따라 달라지므로 모든 반복 append보다 빠르다고 단정하지 않는다. 교체된 영역에 포커스나 사용자의 입력이 있었는지도 확인한다.

---

## 10. 속성과 프로퍼티는 구별해서 다룬다

HTML 속성은 마크업의 설정을 표현하고 DOM 프로퍼티는 객체의 상태를 다룬다. 두 값이 연동되는 경우도 있지만 모두 같지는 않다. 입력의 현재 `.value`와 원래 `value` 속성은 사용자 편집 후 달라질 수 있다. boolean 속성인 `disabled`는 문자열이 `"false"`여도 속성이 존재하면 비활성화에 영향을 준다. 해제하려면 `.disabled = false` 또는 `removeAttribute('disabled')`를 쓴다.

---

## 11. 텍스트·클래스·데이터·HTML을 목적에 맞게 바꾸기

| 목적 | 예 | 주의할 점 |
| --- | --- | --- |
| 문자 표시 | `textContent` | 하위 마크업을 문자로 대체할 수 있음 |
| 클래스 상태 | `classList.add/remove/toggle` | 스타일과 의미 상태의 동기화 확인 |
| 사용자 정의 데이터 | `dataset.noteId` | data-* 값은 문자열로 다룸 |
| 속성의 존재·문자열 설정 | `setAttribute/removeAttribute` | boolean 속성은 존재 의미를 구별 |
| 현재 컨트롤 상태 | `value`, `checked`, `disabled` | 초기 HTML 속성과 연동 방식 확인 |
| HTML 조각 파싱 | `innerHTML`, `insertAdjacentHTML` | 파싱 문맥·보안·기존 노드 교체 고려 |

`textContent`와 `innerText`도 동일하지 않다. textContent는 노드의 텍스트 내용을 다루고 innerText는 렌더링과 숨김·줄바꿈의 영향을 받는 표현을 다룬다. 화면과 무관한 데이터 읽기에 렌더링 의존 API를 습관적으로 사용하지 않는다.

### 보안은 script 태그 실행 여부 하나로 판단하지 않는다

innerHTML로 넣은 script가 실행되지 않는 경우라도 이벤트 핸들러 속성 등 다른 경로의 위험이 남는다. 신뢰할 수 없는 사용자 입력을 HTML로 파싱하지 않는 것이 기본이다. 마크업이 필요한 요구라면 검증된 정화 처리 등 별도의 보안 설계가 필요하며 문자열 치환 몇 개로 완전한 방어를 구현했다고 가정하지 않는다.

### DOM 변경이 사용자 경험을 바꾸는 지점

항목을 삭제하면 화면뿐 아니라 키보드 포커스와 보조 기술이 탐색하는 구조도 바뀐다. 목록 전체를 매번 교체하면 변경되지 않은 입력까지 새 객체가 되어 편집 위치를 잃을 수 있다. 의미 있는 기존 노드를 유지할지, 전체 교체할지 데이터와 조작 흐름을 기준으로 결정한다.

성능 문제는 변경 횟수뿐 아니라 변경 영역·스타일 의존성·이후 측정 호출과 연결된다. 코드의 작성 편의와 실제 사용자 응답 시간을 함께 비교한다.

---

## 12. DOM 변경과 화면 갱신의 관계

스크립트가 DOM을 바꾸면 객체 트리의 변화는 그 시점에 확인할 수 있다. 다만 픽셀 표시가 매 변경 직후 동기적으로 끝나는 것은 아니다. 브라우저는 필요한 스타일 계산·레이아웃·페인트를 진행할 기회에 화면을 갱신한다. DOM이 바뀌어도 계산 결과나 가시성에 따라 눈에 보이는 변화가 없을 수 있다.

문자열을 HTML로 파싱하려고 `innerHTML`을 사용하면 새 요소를 만들 수 있지만, 신뢰하지 않는 입력을 넣으면 악성 마크업이 동작할 위험이 있다. 단순 텍스트는 `textContent`를 사용한다. `innerHTML += ...`는 기존 자식들을 다시 파싱·교체할 수 있어 객체 참조·리스너·포커스를 잃는 문제도 일으킬 수 있다.

---

## 13. 문서 모델을 구조적 불변식으로 이해하기

### DOM이 허용하는 트리와 HTML이 허용하는 내용은 다르다

DOM은 부모·자식 연결, 노드 종류, 문서 루트와 같은 구조 조건을 가진다. 한 노드는 일반 트리에서 두 부모의 자식으로 동시에 들어갈 수 없고, 자기 자신이나 조상을 자기 아래에 넣어 순환을 만들 수도 없다. 반면 “ul 안에는 어떤 HTML 요소가 와야 하는가” 같은 콘텐츠 모델은 HTML의 별도 규칙이다.

따라서 DOM API가 호출에 성공했다는 사실만으로 HTML이 의미상 유효해졌다고 결론 내리면 안 된다. HTML 파서는 일부 잘못된 중첩을 복구하지만 createElement와 append가 그 파싱 과정을 매번 다시 실행하는 것은 아니다. [DOM의 삽입 유효성 규칙](https://dom.spec.whatwg.org/#concept-node-ensure-pre-insertion-validity)과 [HTML 콘텐츠 모델](https://html.spec.whatwg.org/multipage/dom.html#content-models)을 서로 다른 검증으로 읽는다.

### 문서에 연결되었다는 것과 객체가 존재한다는 것

createElement가 반환한 요소는 아직 문서에 없어도 프로퍼티와 자식 노드를 가질 수 있다. 문서에서 제거해도 JavaScript 변수가 그 객체를 계속 참조할 수 있다. isConnected는 어떤 문서에 연결되어 있는지 판단하는 데 쓰지만 객체의 존재·메모리 해제 여부·현재 화면 노출 여부까지 모두 의미하지 않는다.

숨겨진 컨테이너 아래의 요소는 문서에 연결되어 있어도 보이지 않을 수 있다. 반대로 분리된 노드를 변수로 가지고 있다고 화면에 남아 있는 것은 아니다. DOM 구조, 객체 참조, CSS 표시를 각각 확인해야 잘못된 “삭제가 안 된다” 진단을 피할 수 있다.

---

## 14. 여러 실험에서 사용할 작은 DOM 실험실

### 공통 실행 환경

다음 HTML을 파일로 저장해 브라우저에서 연다. 뒤의 JavaScript 실험은 이 문서의 콘솔에서 각각 실행한다. reset에 전달하는 문자열은 아래에 직접 작성한 신뢰된 예제뿐이며 사용자 입력을 HTML로 넣는 패턴이 아니다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>DOM 실험실</title></head>
<body>
  <h1>DOM 실험실</h1>
  <div id="lab"></div>
  <pre id="output"></pre>
  <script>
    const lab = document.querySelector('#lab');
    const output = document.querySelector('#output');
    function reset(markup) {
      lab.innerHTML = markup;
      output.textContent = '';
    }
    function report(value) {
      output.textContent += `${JSON.stringify(value)}\n`;
    }
  </script>
</body>
</html>
```

각 코드 조각은 블록으로 감싸 같은 이름의 const를 다른 실험에서 다시 선언할 수 있게 했다. report는 결과를 현재 문서에 문자열로 표시한다. 실험용 출력과 실제 서비스의 접근성 안내 요구는 다르므로 매번 모든 내부 값을 live 영역에 읽게 하는 방식으로 확장하지 않는다.

### 실험 1. 공백도 노드라는 사실

```javascript
{
  reset('<p>앞 <strong>중요</strong> 뒤</p><!-- 메모 -->');
  const paragraph = lab.querySelector('p');
  report([...paragraph.childNodes].map((node) => [node.nodeType, node.nodeName]));
  report(paragraph.children.length);
  report(lab.childNodes.length);
}
```

p의 자식은 Text, STRONG 요소, Text의 세 노드다. children은 STRONG 하나만 센다. lab에는 p와 Comment 두 노드가 있다. 공백을 화면이 합쳐 보여도 객체 트리의 노드가 자동으로 사라진 것은 아니다. Element만 다룰 코드에서 firstChild를 쓰면 Text를 받아 classList 같은 기능을 찾지 못할 수 있다.

### 실험 2. 정적 목록도 같은 객체를 보관한다

```javascript
{
  reset('<ul><li>A</li><li>B</li></ul>');
  const list = lab.querySelector('ul');
  const snapshot = list.querySelectorAll('li');
  const live = list.children;
  snapshot[0].textContent = '수정한 A';
  const added = document.createElement('li');
  added.textContent = 'C';
  list.append(added);
  report([snapshot.length, live.length]);
  report(snapshot[0].textContent);
  snapshot[0].remove();
  report([snapshot.length, live.length, snapshot[0].isConnected]);
}
```

추가 뒤 길이는 2와 3이다. snapshot의 첫 요소 문구는 수정한 A로 바뀌어 있다. 정적이라는 것은 목록의 구성이지 요소 객체 내용의 불변성이 아니다. 제거 뒤 snapshot 길이는 여전히 2, live는 2이며 snapshot이 보관한 첫 객체는 연결이 끊긴 상태다. [NodeList의 갱신 모델](https://dom.spec.whatwg.org/#old-style-collections)을 실제 참조 관계로 확인하는 예제다.

---

## 15. 이동·복제·제거에서 정체성을 유지하기

### 실험 3. append는 복제가 아니라 이동이다

```javascript
{
  reset('<section class="left"><button type="button">이동할 버튼</button></section><section class="right"></section>');
  const left = lab.querySelector('.left');
  const right = lab.querySelector('.right');
  const button = left.querySelector('button');
  button.dataset.noteId = 'same-object';
  right.append(button);
  report([left.children.length, right.children.length]);
  report(right.firstElementChild === button);
  report(button.dataset.noteId);
}
```

왼쪽은 0, 오른쪽은 1이며 객체 비교는 true다. 데이터 속성도 같은 객체에 남는다. “새 위치에 똑같은 HTML이 보인다”는 시각 결과만으로 재생성과 이동을 구별할 수는 없다. 객체 참조·연결된 상태·포커스 등 보존 요구가 있으면 어떤 API를 사용했는지 확인한다.

### 실험 4. cloneNode와 이벤트 리스너

```javascript
{
  reset('<button type="button" id="original">원본</button>');
  const original = lab.querySelector('button');
  let count = 0;
  original.addEventListener('click', () => { count += 1; });
  const copy = original.cloneNode(true);
  copy.removeAttribute('id');
  copy.textContent = '복제';
  lab.append(copy);
  original.click();
  copy.click();
  report(count);
  report(original === copy);
}
```

count는 1, 객체 비교는 false다. addEventListener로 등록한 리스너는 노드의 모든 실행 상태를 복제하는 대상이 아니다. id 속성은 복제될 수 있어 예제에서 제거했다. cloneNode가 복사하는 마크업과 복사하지 않는 리스너의 범위를 [MDN cloneNode 문서](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode)에서 확인할 수 있다.

click 호출은 이 실험에서 리스너 연결을 관찰하는 수단이다. 실제 키보드·포인터 조작의 모든 동작이나 사용자 활성화 조건을 재현하는 시험으로 사용하지 않는다. 접근성 검증에는 실제 조작도 필요하다.

### 실험 5. 순환 삽입과 잘못된 HTML은 다른 결과다

```javascript
{
  reset('<ul><li>A</li></ul>');
  const list = lab.querySelector('ul');
  const item = list.firstElementChild;
  try {
    item.append(list);
  } catch (error) {
    report(error.name);
  }
  const wrapper = document.createElement('div');
  wrapper.textContent = '직접 넣은 div';
  list.append(wrapper);
  report(list.lastElementChild.tagName);
}
```

첫 작업은 조상을 자손 안으로 넣어 순환을 만들려 하므로 구조 오류가 발생한다. 둘째는 div가 ul의 직접 자식이 되어 HTML 콘텐츠 모델에 어긋나지만 DOM 삽입 자체는 가능하다. 파서의 자동 수정이 여기서 개입한다고 기대하지 말고 HTML 검사기로 별도 확인한다.

---

## 16. 속성·프로퍼티·현재 상태를 독립적으로 관찰하기

### 실험 6. 문자열 false와 boolean 상태

```javascript
{
  reset('<input name="topic" value="HTML">');
  const input = lab.querySelector('input');
  input.setAttribute('disabled', 'false');
  report([input.hasAttribute('disabled'), input.disabled]);
  input.disabled = false;
  report([input.hasAttribute('disabled'), input.disabled]);
}
```

첫 결과는 true·true이고 둘째는 false·false다. boolean 속성은 존재 자체가 의미를 가지므로 문자열 false가 비활성화를 해제하지 않는다. 반면 일부 다른 속성은 열거 값·문자열·숫자 등 다른 변환 규칙을 사용한다. 모든 HTML 속성을 같은 boolean 규칙으로 다루지 않는다.

### 실험 7. 초기값·현재 값·폼 reset

```javascript
{
  reset('<form><input name="topic" value="HTML"></form>');
  const form = lab.querySelector('form');
  const input = form.elements.namedItem('topic');
  input.value = 'DOM';
  input.setAttribute('value', 'Events');
  report([input.getAttribute('value'), input.defaultValue, input.value]);
  form.reset();
  report([input.defaultValue, input.value]);
}
```

사용자가 편집한 상태처럼 현재 value를 먼저 바꾸면 이후 value 속성·defaultValue를 바꾸는 일과 현재 값 변경이 구별된다. reset 전에는 기본값 Events와 현재 값 DOM이 다를 수 있고, reset 뒤 현재 값이 기본값 Events로 돌아간다. 이는 input의 상태 규칙이며 모든 요소의 속성·프로퍼티에 동일하게 적용되는 일반 캐시 규칙이 아니다. [HTML input의 value·dirty value flag](https://html.spec.whatwg.org/multipage/input.html#the-input-element)를 기준으로 이해한다.

### dataset과 classList의 역할

dataset은 data-* 속성을 편리한 프로퍼티 이름으로 읽고 쓰는 경로이며 대체로 문자열을 다룬다. classList는 클래스 토큰을 다루므로 className 문자열 전체를 조립하다 다른 클래스를 잃는 문제를 줄일 수 있다. 클래스 변경이 접근성 상태를 자동으로 바꾸지는 않으므로 펼침·선택의 의미 상태가 필요하면 관련 속성도 일치시킨다.

---

## 17. 문자열 표현과 트리 변경을 혼동하지 않기

### 실험 8. textContent가 하위 요소를 대체한다

```javascript
{
  reset('<p>앞 <strong>중요</strong> 뒤</p>');
  const paragraph = lab.querySelector('p');
  const strong = paragraph.querySelector('strong');
  paragraph.textContent = '<strong>새 내용</strong>';
  report(paragraph.children.length);
  report(paragraph.textContent);
  report(strong.isConnected);
}
```

새 strong 요소가 생기지 않고 태그 모양 문자열이 텍스트로 표시된다. 기존 strong은 트리에서 빠지지만 변수로는 계속 가리킬 수 있다. textContent를 안전한 텍스트 출력에 사용하는 이유와 기존 자식 전체를 교체할 수 있다는 영향은 동시에 이해해야 한다.

### innerHTML의 직렬화 결과와 원본 HTML

innerHTML을 읽으면 현재 트리의 직렬화 결과를 얻는다. 브라우저가 파싱 중에 삽입한 노드와 스크립트가 바꾼 내용이 들어갈 수 있어 원래 받은 응답과 다르다. 문자열 비교로 원본을 복원하거나 변경 여부를 판단하려면 그 차이를 먼저 정의해야 한다.

### 읽기 API도 비용과 의미가 다르다

textContent는 트리의 텍스트를 다루고 innerText는 표현과 숨김·줄바꿈의 영향을 받을 수 있다. getBoundingClientRect는 시각 사각형을 관찰하며 레이아웃 상태를 계산할 필요가 생길 수 있다. 원하는 정보가 텍스트인지·구조인지·시각 크기인지 구별하여 API를 고른다.

---

## 18. 변경을 실제 사용 가능성과 연결하여 검증하기

### 최소 변경과 전체 교체의 선택

목록 전체를 replaceChildren로 새로 만드는 구현은 단순할 수 있지만, 입력 중인 노드·포커스·외부 코드가 보관한 참조가 사라질 수 있다. 데이터 정체성이 유지되는 항목은 기존 노드를 갱신하는 방식이 맞을 수도 있다. 반대로 복잡하게 부분 갱신하다 상태를 놓치는 경우도 있어 정확성과 유지보수 비용을 함께 비교한다.

### 성능은 변경 뒤 어떤 읽기를 하는지도 본다

스타일을 바꾸고 크기를 읽는 일을 반복하면 필요한 레이아웃 계산을 여러 번 유발할 수 있다. 같은 요구에서 측정과 쓰기를 모을 수 있는지 확인하되, 단순히 fragment를 사용했다는 이유만으로 빨라졌다고 결론 내리지 않는다. Performance 기록에서 실제 변경 구간·레이아웃·페인트 시간을 같은 조건으로 비교한다.

### 삭제 후 포커스·이름·상태를 확인한다

버튼으로 항목을 삭제했다면 사용자가 다음에 어디서 작업할지 정한다. 새로운 컨트롤을 추가했다면 이름과 설명이 연결되는지 확인한다. 중요한 결과를 알릴 필요가 있다면 적절한 상태 안내를 사용한다. DOM 갱신이 시각적으로 맞아도 키보드·보조 기술에서 완료되지 않은 기능일 수 있다.

---

## 19. 실무와 디버깅

선택 결과가 null이면 선택자, 실행 시점, 현재 문서를 확인한다. Elements와 `parentNode`, `children`, `isConnected`를 함께 보면 실제 위치를 추적할 수 있다. live 컬렉션을 앞에서부터 제거하며 순회하면 인덱스가 바뀌어 건너뛸 수 있으므로 복사본 순회 등 의도적인 방법을 택한다.

접근성은 DOM 조작 후에도 유지해야 한다. 새 컨트롤에 이름을 붙이고, 오류·완료 안내가 필요하면 적절한 상태 영역과 포커스 흐름을 설계한다. 성능이 느리다면 DOM 변경 횟수만 세기보다 변경과 기하 측정을 번갈아 수행해 레이아웃 비용을 반복 유발하는지 측정한다. DocumentFragment는 여러 노드를 구성할 수 있는 도구이지 모든 상황에서 더 빠른 것을 보장하는 도구는 아니다.

\[미리보기: React의 DOM 관리\]

React는 화면의 상태에 따라 DOM 변경을 관리한다. React가 관리하는 영역을 임의의 DOM 코드로 바꾸면 두 관리 방식이 충돌할 수 있으며, 기본 관계는 뒤의 Component / Props / State에서 연결한다.

---

## 20. 핵심 정리

<callout icon="✅" color="green_bg">
DOM은 HTML 문자열이 아니라 객체의 트리다. Element는 Node의 일부이고, 조회 결과의 갱신 방식과 현재 프로퍼티를 구별해야 한다. 안전하게 노드를 바꾸되 실제 화면·포커스·성능까지 확인한다.
</callout>

---

## 출처

- [WHATWG — DOM: Nodes](https://dom.spec.whatwg.org/#nodes)
- [WHATWG — DOM: Collections](https://dom.spec.whatwg.org/#old-style-collections)
- [MDN — innerHTML 보안과 동작](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Pre-insertion Validity](https://dom.spec.whatwg.org/#concept-node-ensure-pre-insertion-validity): 순환·노드 종류·문서 구조에 대한 삽입 제약.
- [WHATWG — HTML Content Models](https://html.spec.whatwg.org/multipage/dom.html#content-models): DOM 삽입 성공과 HTML 유효성이 다른 이유.
- [WHATWG — Collections](https://dom.spec.whatwg.org/#old-style-collections): 정적·live 목록의 의미와 관찰 범위.
- [MDN — cloneNode](https://developer.mozilla.org/en-US/docs/Web/API/Node/cloneNode): 복제되는 트리와 addEventListener 리스너·중복 id 주의점.
- [WHATWG — input](https://html.spec.whatwg.org/multipage/input.html#the-input-element): 기본값과 현재 값·dirty 상태의 규칙.
- [MDN — textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent): 하위 노드 대체와 텍스트 읽기·쓰기의 의미.

---

## 21. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 들여쓰기와 주석이 있는 요소에서 childNodes 수와 children 수가 다르다. 이유와 확인 방법을 설명하라.
2. querySelectorAll로 저장한 목록 뒤에 항목을 추가했다. 저장된 목록과 현재 children의 길이가 다른 이유는 무엇인가?
3. 동일한 노드를 두 번 append하면 두 개가 생기는가? 객체와 트리 위치를 구별하여 설명하라.
4. 사용자 이름을 innerHTML로 넣는 방법과 textContent로 넣는 방법의 동작·보안 차이를 설명하라.
