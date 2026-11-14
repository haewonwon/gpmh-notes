---
track: B
notion: https://app.notion.com/p/415178c4afac81368230f7c036e79156
---

# script / async / defer / module


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · script / async / defer / module

HTML 파싱·외부 파일 준비·코드 실행·DOM과 데이터 준비를 구별하여 로딩 방식을 선택한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- classic·async·defer·module의 준비·실행·순서 차이를 설명할 수 있다.
- DOM·코드·데이터 의존성을 구별하여 초기화 흐름을 설계할 수 있다.
- 파서 삽입과 동적 script의 실행 규칙을 구별할 수 있다.
- module의 스코프·import·URL·MIME 요구를 설명할 수 있다.
- DOMContentLoaded·readyState와 앱 준비의 차이를 실제 로딩 조건에서 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

같은 JavaScript도 언제 실행되는지에 따라 요소를 찾지 못하거나 의존 함수가 아직 정의되지 않을 수 있다. 로딩 속성을 고르는 일은 파일을 빨리 받는 문제와 실행 의존성을 함께 다루는 일이다. HTML 파싱에서 미리 본 실행 시점을 이 페이지에서 연결한다.

---

## 3. 관련 계층과 역할 구분

HTML 파서는 script 요소를 발견하고 네트워크는 파일과 의존 모듈을 준비한다. JavaScript 엔진은 코드를 평가하며 DOM API는 현재 만들어진 문서에 접근한다. 렌더링과 앱 데이터 준비는 별도의 조건을 가진다.

빌드 도구가 번들·경로를 바꿀 수 있어도 브라우저가 최종 응답을 읽고 실행한다는 경계는 남는다. 예제의 범위가 파서 삽입인지 동적 삽입인지 명시하여 규칙을 적용한다.

---

## 4. 다운로드와 실행은 다르다

브라우저는 스크립트를 네트워크로 준비하고 JavaScript 엔진으로 실행한다. 다운로드가 HTML 파싱과 겹쳐도 실제 메인 스레드 실행은 파싱과 다른 메인 스레드 작업의 진행에 영향을 줄 수 있다. “비동기 다운로드”를 “실행도 화면과 완전히 독립적”이라고 이해하지 않는다.

다음 비교는 **일반 HTML 문서에서 파서가 삽입한 스크립트**를 기준으로 한다. 동적으로 생성한 script, import 관계, 실패·차단 조건 등은 별도 규칙이 있다. [HTML script 표준](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element)이 기준이다.

| 형태 | 준비와 실행 | 문서 순서 | DOMContentLoaded와의 관계 |
| --- | --- | --- | --- |
| 외부 classic 기본 | 일반적으로 파싱을 기다리게 하고 준비 후 실행 | 파서가 만난 순서 | 실행·이후 파싱이 끝나야 진행 |
| 외부 classic `defer` | 파싱과 다운로드를 겹치고 파싱 후 실행 | defer끼리 문서 순서 | 일반적으로 실행을 기다림 |
| 외부 classic `async` | 파싱과 다운로드를 겹치고 준비되면 실행 | 다운로드·준비 순서에 영향 | async 완료를 기다리는 기준이 아님 |
| `type="module"` | 의존 모듈을 준비하고 기본적으로 파싱 후 평가 | 모듈 의존 관계 포함 | 기본 지연 처리에 관여, 비동기 평가 주의 |
| `type="module" async` | 모듈 그래프 준비 후 가능한 시점에 평가 | 다른 script와 순서 보장 없음 | async로 분류 |

인라인 classic script에는 외부 다운로드가 없고 파서가 만났을 때 실행된다. 여기에 async·defer를 붙여 외부 classic처럼 만들 수는 없다. module은 인라인에도 지연 처리 의미가 있다. module에 defer를 추가해도 기본 module 동작을 더 지연시키는 별도 기능이 되지 않는다.

---

## 5. 파서·네트워크·실행의 시간을 나누어 읽기

HTML 파서는 앞에서부터 토큰을 처리해 문서를 만든다. 네트워크는 외부 파일을 준비하고, JavaScript 엔진은 준비된 코드를 실행한다. 다운로드를 겹치는 것과 실제 실행을 다른 스레드로 옮기는 것은 다르다. async·defer라는 속성 이름만으로 긴 JavaScript 실행이 UI를 막지 않는다고 판단하지 않는다.

브라우저의 preload scanner 같은 최적화는 뒤쪽 리소스 요청을 앞당길 수 있다. 그렇다고 실제 DOM 파서가 그 위치까지 이미 완성된 것은 아니다. Network의 요청 시작 순서와 콘솔의 실행 순서·DOM 준비 시점을 각각 관찰해야 한다.

### 의존성을 세 종류로 구별한다

1. DOM 의존성: 실행할 때 대상 요소가 이미 존재해야 한다.
2. 코드 의존성: 먼저 실행한 코드의 함수·전역 값 또는 import가 필요하다.
3. 데이터 의존성: fetch 등 별도 비동기 초기화의 결과가 필요하다.

defer는 앞의 일반적인 DOM·defer 문서 순서 요구에 도움이 되지만 별도 요청의 업무 데이터까지 준비해 주지는 않는다. module은 import로 코드 의존성을 표현하지만 모든 비동기 데이터가 자동으로 동기화되는 것은 아니다.

---

## 6. 기본 classic이 DOM을 못 찾는 이유

```html
<head>
  <script>
    console.log(document.querySelector('#message')); // 뒤의 요소가 아직 없음
  </script>
</head>
<body><p id="message">문서 내용</p></body>
```

파서는 뒤쪽 body까지 트리를 만들기 전에 스크립트를 실행한다. 따라서 올바른 선택자여도 결과가 null일 수 있다. 스크립트를 body 끝으로 옮기면 앞서 파싱된 요소를 다룰 수 있지만, 외부 파일을 일찍 발견하면서 DOM 파싱 뒤 실행하려는 목적에는 defer가 적합할 수 있다.

---

## 7. 예제: 의존 순서가 있는 외부 파일

아래 세 파일을 같은 디렉터리의 로컬 HTTP 서버로 제공한다.

```html
<!-- index.html -->
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>스크립트 실행 순서</title>
  <script defer src="./label.js"></script>
  <script defer src="./app.js"></script>
</head>
<body><p id="message">준비 중</p></body>
</html>
```

```javascript
// label.js
window.makeLabel = (topic) => `학습: ${topic}`;
console.log('label 실행');
```

```javascript
// app.js
document.querySelector('#message').textContent = window.makeLabel('Web Platform');
console.log('app 실행');
```

두 파일은 파싱과 겹쳐 다운로드할 수 있지만 실행은 label → app 순서이며 이때 본문 파싱도 끝나 있다. 따라서 문단은 “학습: Web Platform”으로 바뀐다. 두 태그를 async로 바꾸면 app이 먼저 실행되어 함수가 없거나 아직 DOM이 준비되지 않을 수 있다. 캐시 때문에 우연히 성공하는 결과를 순서 보장으로 간주하지 않는다.

서로 독립적인 분석 스크립트는 async를 검토할 수 있지만, DOM이나 다른 스크립트에 의존하면 그 의존성을 명시적으로 처리해야 한다.

---

## 8. 동적으로 삽입한 script의 실행 규칙

### 파서가 발견한 태그와 생성 API로 만든 태그를 구별한다

```javascript
const script = document.createElement('script');
script.src = './feature.js';
script.addEventListener('load', () => console.log('파일 실행 후 load'));
script.addEventListener('error', () => console.log('파일 준비 실패'));
document.head.append(script);
```

feature.js가 존재하는 HTTP 환경의 문서에서 실행하는 조각이다. 동적 외부 classic script는 일반적으로 준비된 순서대로 실행할 수 있다. HTML에 적힌 defer 태그 두 개의 순서 규칙을 그대로 가져오지 않는다.

### 순서를 요구하면 명시적으로 관리한다

```javascript
for (const source of ['./label.js', './app.js']) {
  const script = document.createElement('script');
  script.async = false;
  script.src = source;
  document.head.append(script);
}
```

이 예제는 앞의 classic 파일 둘이 실제로 존재하고 document에 필요한 요소도 이미 준비된 때의 조각이다. async false를 삽입 전에 설정하여 동적 classic의 순서 의도를 표현한다. 파일 실패 시 의존 기능을 어떻게 중단·안내할지도 구현해야 한다. 모듈 의존성이면 import로 관계를 표현하는 것이 더 직접적일 수 있다.

### innerHTML로 script 문자열을 넣는 것은 별도 경로다

innerHTML로 마크업을 넣는 것과 script 요소를 생성하여 문서에 연결하는 것은 같은 실행 절차가 아니다. innerHTML로 넣은 script가 실행되지 않는 경우라도 다른 악성 마크업 위험은 남는다. 파싱 경로와 보안은 HTML·DOM 페이지의 구분을 유지한다.

---

## 9. module은 로딩 속성뿐 아니라 실행 모델도 바꾼다

```javascript
// label-module.js
export const makeLabel = (topic) => `학습: ${topic}`;
```

```javascript
// app-module.js
import { makeLabel } from './label-module.js';
document.querySelector('#message').textContent = makeLabel('Module');
```

HTML에서 앞의 두 classic 태그를 `<script type="module" src="./app-module.js"></script>` 하나로 바꾸면 의존 관계를 import로 표현한다. 모듈은 자체 스코프와 strict mode를 가지며, 최상위 선언이 자동으로 window 프로퍼티가 되는 classic 방식과 다르다. 정상적인 JavaScript MIME 타입과 모듈의 CORS 요구를 확인하고 file URL보다 HTTP 서버로 실험한다.

모듈 의존성의 상세 평가 순서와 top-level await 내부 동작은 이후 런타임 학습 범위다. 여기서는 비동기 평가가 있으면 DOMContentLoaded가 모든 후속 비동기 초기화 완료를 보장하지 않는다는 점만 구별한다.

---

## 10. module에서 함께 바뀌는 스코프·의존성·로딩

### import가 전역 변수 공유를 대신한다

classic 파일의 최상위 선언이 window와 어떻게 연결되는지도 선언 방식에 따라 다르다. module은 자체 스코프와 명시적인 export·import로 의존성을 표현한다. 전역에 함수를 넣던 코드를 type만 바꾸면 참조 위치가 맞지 않을 수 있다.

### URL과 응답 형식도 실행 계약이다

브라우저 모듈은 주소로 의존 모듈을 찾고 JavaScript에 맞는 MIME 타입 등을 검사한다. 빌드 도구가 변환해 주는 패키지 이름 import와 브라우저가 직접 해석할 수 있는 경로를 구별한다. 개발 서버에서는 되지만 정적 배포에서 HTML 오류 페이지를 JS 주소로 반환하면 MIME·파싱 오류가 날 수 있다.

같은 실행 환경에서 동일하게 해석된 모듈은 재사용될 수 있다. 하지만 다른 문서·Worker까지 하나의 전역 인스턴스가 공유된다는 뜻은 아니다. URL·실행 환경·번들 처리에 따라 달라지는 범위를 확인한다.

---

## 11. DOMContentLoaded는 모든 준비의 끝이 아니다

DOMContentLoaded는 문서 파싱과 관련 지연 스크립트 처리의 기준이다. 모든 이미지 다운로드나 async 스크립트, 별도로 시작한 fetch·Promise 작업의 완료를 뜻하지 않는다. 지연 처리 스크립트와 스타일시트 사이의 대기로 발생 시점이 영향을 받을 수도 있다.

늦게 불러온 코드에서 이벤트 리스너만 붙이면 이미 지나간 DOMContentLoaded를 놓칠 수 있다. 이때는 `document.readyState`를 확인하여 아직 loading이면 리스너를 등록하고, 아니면 필요한 초기화를 실행하는 패턴을 검토한다. 화면의 실제 사용 가능 시점은 앱의 데이터·초기화 조건까지 포함해 따로 정의해야 한다.

---

## 12. DOM 준비와 애플리케이션 준비를 따로 기다리기

```javascript
function initializeDocument() {
  const output = document.querySelector('#message');
  if (output) output.textContent = '문서 준비 완료';
}
if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', initializeDocument, { once: true });
} else {
  initializeDocument();
}
```

이 패턴은 늦게 로드된 코드가 DOMContentLoaded를 놓치지 않고 DOM 관련 초기화를 시작하도록 한다. readyState가 loading이 아니라는 이유만으로 다른 모든 지연 스크립트의 전역 초기화나 API 요청까지 완료되었다고 간주하지 않는다. 코드·업무 데이터의 준비는 별도의 Promise·초기화 계약으로 연결한다.

### 실패 상황에서의 검증 순서

파일이 200인지뿐 아니라 응답 본문이 실제 JavaScript인지 확인한다. 다음으로 콘솔 오류의 최초 위치, 선행 코드의 실패 여부, DOM 대상의 존재 시점, 초기화 중복을 조사한다. 캐시가 켜진 빠른 환경에서 우연히 맞는 실행 순서를 보장으로 오해하지 않도록 제한된 네트워크에서도 시험한다.

오류나 지연이 있을 때 사용자에게 기본 콘텐츠·대기·재시도 경로를 제공한다. 속성 선택으로 파일 준비 시간을 겹친 뒤에도 긴 평가·큰 번들의 비용이 남는다면 실제 Performance 구간을 측정하여 다음 개선을 정한다.

---

## 13. 스크립트 로딩을 부분 순서로 예측하기

### 다운로드 시작·준비 완료·실행은 별개다

태그를 먼저 만난 것이 파일의 준비 완료나 실행 완료를 뜻하지 않는다. defer는 일반적인 파서 삽입 경로에서 문서 순서·파싱 뒤 실행과 연결되고, async는 다른 script와 문서 순서를 보장하지 않는다. [HTML script 처리 모델](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element)을 각 사건의 조건으로 읽는다.

### 한 가지 총순서 대신 보장되는 선후 관계

async 파일과 본문 파싱의 상대 시점은 네트워크·캐시·현재 파싱 상태에 따라 달라질 수 있다. 반면 같은 문서에서 파서가 삽입한 defer A·B의 실행 순서는 의존할 수 있는 조건이다. 한 번 찍힌 로그의 순서를 모든 실행의 보장으로 기록하지 않는다. 이 구분은 로컬에서만 우연히 성공하는 의존 코드를 찾는 기준이다.

### DOM·코드·데이터의 준비를 따로 적는다

요소를 찾으려면 DOM이 필요하고, 선행 함수를 호출하려면 해당 코드가 실행되어야 하며, 검색 결과를 표시하려면 데이터가 필요하다. defer가 DOM과 코드 순서 문제에 도움이 되어도 별도의 fetch 결과까지 준비해 주지는 않는다. 초기화가 기다려야 할 대상을 구체적으로 나눈다.

---

## 14. 순서와 DOM 접근을 기록하는 실행 실험

### 실험 1. 공통 기록 함수와 세 외부 파일

아래 파일들을 같은 디렉터리의 로컬 HTTP 서버로 제공한다. index.html의 첫 인라인 코드는 뒤의 파일들이 사용할 기록 함수를 준비한다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>스크립트 시간표</title>
  <script>
    window.studyTimeline = [];
    window.recordStudy = (name) => {
      const row = `${name}: ${document.readyState}, target=${Boolean(document.querySelector('#target'))}`;
      studyTimeline.push(row);
      console.log(row);
      const output = document.querySelector('#output');
      if (output) output.textContent = studyTimeline.join('\n');
    };
    document.addEventListener('DOMContentLoaded', () => recordStudy('DOMContentLoaded'));
    window.addEventListener('load', () => recordStudy('load'));
  </script>
  <script async src="./independent.js"></script>
  <script defer src="./foundation.js"></script>
  <script defer src="./application.js"></script>
</head>
<body>
  <h1 id="target">실행 순서</h1>
  <pre id="output"></pre>
  <script>recordStudy('body inline');</script>
</body>
</html>
```

```javascript
// independent.js
recordStudy('async independent');
```

```javascript
// foundation.js
window.makeStudyLabel = (topic) => `학습: ${topic}`;
recordStudy('defer foundation');
```

```javascript
// application.js
document.querySelector('#target').textContent = makeStudyLabel('Web Platform');
recordStudy('defer application');
```

body inline은 그 위치까지 파싱한 상태에서 실행하여 target이 존재한다. foundation은 application보다 먼저 실행되고 두 defer 코드가 실행될 때 본문 파싱은 끝나 있다. DOMContentLoaded는 이 일반 defer 실행들을 지난 뒤에 관찰한다. async independent가 body inline의 앞뒤 어디에 나오는지는 준비 상태에 따라 달라질 수 있다.

기본 load는 관련 자원 준비와 연결되지만, 각 스크립트가 별도로 시작한 모든 Promise·업무 데이터 초기화까지 기다리는 계약은 아니다. 기록 함수는 현재 문서의 조건을 관찰하는 도구이며 다른 자원을 추가하면 선후 관계를 다시 검토해야 한다.

### 실험 2. 의존 파일 둘을 async로 바꾸기

foundation과 application의 defer를 async로 바꾼다. application이 먼저 준비되면 필요한 함수가 없을 수 있다. 빠른 로컬 환경에서 성공했다고 순서가 보장된 것은 아니며, 네트워크 제한·캐시를 바꿔도 모든 실행에서 실패가 반드시 나타나는 것은 아니다. 여기서 증명해야 할 것은 실패 빈도가 아니라 의존 순서가 계약으로 표현되었는지다.

### 실험 3. 기본 classic으로 바꾸기

두 속성을 제거하면 일반적인 파서 삽입 classic 경로에서 준비·실행을 기다린다. application은 뒤쪽 target을 만들기 전에 실행될 수 있다. 함수 의존성은 문서 순서로 충족되어도 DOM 의존성은 충족되지 않는 것이다. 태그를 body 끝에 옮기는 선택과 defer로 일찍 준비하는 선택을 각각의 요구에 맞게 비교한다.

---

## 15. module의 의존 그래프·스코프·응답 형식

### 실험 4. 전역 연결을 import로 바꾸기

앞의 foundation·application 태그를 아래 module 태그로 바꾸고 두 파일을 준비한다.

```html
<script type="module" src="./module-app.js"></script>
```

```javascript
// module-foundation.js
export function makeStudyLabel(topic) { return `학습: ${topic}`; }
recordStudy('module foundation');
```

```javascript
// module-app.js
import { makeStudyLabel } from './module-foundation.js';
document.querySelector('#target').textContent = makeStudyLabel('Module');
recordStudy('module application');
```

의존 관계가 import로 표현된다. module 파일의 선언은 자동으로 window.makeStudyLabel이 되는 공유 전역 계약이 아니다. 자체 스코프·strict mode 등도 바뀌므로 type 속성만 바꾸고 기존 전역 연결이 그대로 유지될 것으로 기대하지 않는다. 기본 module의 파싱 뒤 처리와 async module·비동기 평가의 조건도 구별한다.

### classic의 전역도 선언 방식에 따라 다르다

classic의 var·함수 선언과 let·const는 전역 객체와의 관계가 다를 수 있다. 따라서 “classic의 모든 선언은 window에 있다”는 설명도 정확하지 않다. 공유 목적의 API는 명시적 import 또는 의도한 window 프로퍼티로 표현하고, 어느 실행 환경에서 어떤 값을 공유하려는지 정한다.

### URL과 MIME을 실제 응답에서 확인한다

브라우저 모듈은 의존성을 URL로 준비하고 올바른 JavaScript MIME 등 로딩 요구를 검사한다. JS 주소에서 오류 HTML을 반환한다면 상태 코드만 봐서는 원인을 놓칠 수 있다. 빌드 도구가 처리하는 패키지 이름과 브라우저가 직접 해석하는 경로·import map 조건도 구별한다. [MDN Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)의 브라우저 요구를 기준으로 한다.

### 모듈 재사용의 범위

같은 실행 환경에서 같은 주소로 해석된 모듈은 모듈 그래프 안에서 재사용될 수 있다. 다른 문서·Worker·다른 주소까지 하나의 전역 인스턴스를 자동 공유한다는 뜻은 아니다. 같은 파일 이름이라는 인상보다 해석된 URL·실행 환경·빌드 결과를 확인한다.

---

## 16. 동적 script의 순서와 실패를 직접 표현하기

### 실험 5. 선행 파일이 성공한 뒤 다음 파일 준비

target과 기록 함수가 이미 있는 별도의 실험 문서에서 실행한다. 앞의 정적 태그와 중복 실행하지 않는다.

```javascript
function loadClassic(source) {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = source;
    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`준비 실패: ${source}`));
    document.head.append(script);
  });
}
async function initializeStudy() {
  try {
    await loadClassic('./foundation.js');
    await loadClassic('./application.js');
    console.log('순서가 필요한 파일 처리 완료');
  } catch (error) {
    console.error(error);
  }
}
initializeStudy();
```

첫 파일의 load를 기다린 뒤 둘째를 요청하므로 실패 시 둘째를 시작하지 않는다. 동적으로 두 태그를 async false로 추가하여 다운로드를 겹치되 실행 순서를 관리하는 방식과는 준비 시간의 겹침이 다르다. 의존성과 네트워크 준비 효율을 함께 고려한다.

### load가 앱의 모든 비동기 처리를 기다리지는 않는다

파일 실행이 다른 fetch·타이머·Promise를 시작했다면 그 업무 완료를 별도 계약으로 기다려야 한다. script load 성공과 앱 내부 초기화 성공을 같은 신호로 사용하지 않는다. 파일 로딩 실패와 실행 중 예외도 개발자 도구에서 구별한다.

### 태그 제거와 실행 효과의 정리는 별개다

동적 script를 여러 번 추가하면 리스너·타이머·위젯이 중복 생성될 수 있다. 실행 뒤 태그를 제거해도 그 코드가 등록한 모든 작업이 자동으로 해제되는 것은 아니다. 로더의 중복 초기화 정책, 재시도, 기능 수명과 정리를 함께 정한다.

---

## 17. DOMContentLoaded가 앱 준비 완료가 아닌 반례

### readyState를 어떤 기준으로 사용할까

loading이면 파싱 중이고 그 밖의 상태는 DOM 초기화를 시작할 단서다. 그러나 interactive라는 이유만으로 모든 defer·module 코드나 업무 데이터가 준비되었다고 보장하지 않는다. 다른 코드가 필요하면 명시적인 의존·완료 신호로 연결한다.

### 실험 6. module의 await 이후 코드

```javascript
// async-module.js: type="module"로 연결한다.
recordStudy('module 시작');
await new Promise((resolve) => setTimeout(resolve, 1000));
recordStudy('대기 이후 module');
```

이 코드의 await 이후 실행이 DOMContentLoaded 뒤에 올 수 있다. [DOMContentLoaded 공식 문서](https://developer.mozilla.org/en-US/docs/Web/API/Document/DOMContentLoaded_event)는 동적 import·async·주입 스크립트·await 이후 코드에서 준비 이벤트가 이미 지났을 수 있는 조건을 설명한다. 모든 module 내부 비동기 작업을 해당 이벤트가 끝까지 기다린다는 모델은 잘못이다.

여기서는 DOM 준비와 업무 준비를 구별하는 반례로만 사용한다. top-level await의 상세 모듈 평가 절차는 후속 런타임 학습에서 확장한다.

### 스타일시트가 간접적으로 시점에 영향을 주는 경우

스크립트 실행은 관련 실행 차단 스타일시트가 준비되기를 기다릴 수 있다. DOMContentLoaded가 모든 CSS 다운로드를 직접 기다리는 단일 규칙은 아니더라도, 지연 스크립트의 대기 때문에 발생 시점이 영향을 받을 수 있다. 모든 스타일시트·async 자원에 같은 대기 규칙을 적용하지 않는다.

---

## 18. 실행 오류·성능·사용 가능성을 함께 확인하기

### 준비 단계별 기록

Network에서는 요청 주소·응답 본문·MIME·실패를, 콘솔에서는 실행 순서와 최초 오류를, DOM에서는 대상 생성 시점을 확인한다. 함수가 없으면 스코프·선행 실행·내부 실패를, 요소가 없으면 파싱 시점을, 화면만 늦으면 별도 데이터와 긴 평가 구간을 조사한다.

### 순서 개선이 계산 비용을 없애지는 않는다

async·defer·module로 준비 시간을 겹쳐도 긴 JavaScript 평가가 입력을 늦출 수 있다. 같은 콘텐츠·기기·네트워크에서 다운로드와 메인 스레드 실행을 나누어 측정한다. 코드 분할 등 실제 성능 전략은 후속 Web Performance에서 확장한다.

### 실패와 지연도 UI 상태다

초기 HTML에 제목·안내·기본 탐색을 제공하고 대기·실패·재시도 경로를 정한다. 캐시된 빠른 환경뿐 아니라 느린 준비·잘못된 MIME·실패 응답에서도 키보드 사용과 의미 구조가 유지되는지 확인한다. 로딩 속성 선택은 이런 요구를 만족시키는 수단이다.

---

## 19. 실무 적용과 검증

Network에서 다운로드 순서를, 콘솔에서 실행 순서를 확인한다. 네트워크 제한·캐시 비활성화·새로고침을 바꾸며 우연한 순서 의존을 찾는다. 메인 스레드의 긴 실행은 async·defer만으로 해결되지 않으므로 Performance에서 평가 비용을 확인한다.

초기 HTML에도 제목·안내·기본 탐색을 제공하면 스크립트가 늦거나 실패해도 사용자가 상태를 이해할 수 있다. 초기화 중인 컨트롤의 비활성화와 완료·실패 안내도 로딩 설계의 일부다.

---

## 20. 핵심 정리

<callout icon="✅" color="green_bg">
async와 defer는 다운로드뿐 아니라 실행 시점·순서가 다르다. module은 의존 관계와 스코프까지 바꾼다. 필요한 DOM·다른 코드·비동기 데이터 중 무엇에 의존하는지 먼저 파악하고 준비 조건을 설계한다.
</callout>

---

## 출처

- [WHATWG — The script element](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element)
- [MDN — script](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script)
- [MDN — DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/API/Document/DOMContentLoaded_event)
- [MDN — JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Script Element](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element): classic·module·async·defer·parser-inserted 처리.
- [MDN — script](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/script): 속성·인라인·외부 로딩의 적용 조건.
- [MDN — DOMContentLoaded](https://developer.mozilla.org/en-US/docs/Web/API/Document/DOMContentLoaded_event): 문서 준비와 await 뒤 실행·지연 스크립트의 범위.
- [MDN — Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules): import·스코프·모듈의 URL·MIME 요구.

---

## 21. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. head의 classic script가 뒤쪽 요소를 못 찾는다. 파싱·실행 순서로 원인과 두 가지 해결 방법을 설명하라.
2. 라이브러리와 의존 앱 파일에 모두 async를 붙이면 어떤 경쟁이 생길 수 있는가?
3. module로 바꾼 뒤 window에서 최상위 변수를 찾지 못한다. 로딩 속도 외에 무엇이 바뀌었는가?
4. DOMContentLoaded가 발생했으므로 API 데이터·이미지·모든 비동기 초기화도 완료되었다고 판단할 수 있는가?
