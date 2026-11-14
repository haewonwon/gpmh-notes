---
track: B
notion: https://app.notion.com/p/415178c4afac81e7a6a7c82871a4a880
---

# IntersectionObserver / MutationObserver / Web Worker 개념


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · IntersectionObserver / MutationObserver / Web Worker 개념

기하 교차·DOM 변경·별도 계산을 각각 이해하고 목적에 맞는 감지·실행 경로를 고른다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- IntersectionObserver의 기준·경계·비율·Entry와 관찰 종료를 설명할 수 있다.
- MutationObserver의 변경 종류·범위·기록 전달·반복 위험을 구별할 수 있다.
- Worker의 DOM 제약·메시지 복제·응답 계약·종료를 설명할 수 있다.
- 관찰 콜백과 별도 실행 환경을 구별하여 성능 기대를 판단할 수 있다.
- 지연 로딩·외부 DOM 변경·큰 계산에서 실제 비용과 사용자 피드백을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

“특정 영역이 화면 근처에 왔는가”, “외부 코드가 DOM을 바꾸었는가”, “계산 때문에 클릭이 늦어지는가”는 서로 다른 문제다. 관찰자와 Worker는 모두 콜백·비동기 흐름과 연결되지만 역할과 실행 위치가 다르다. 도구를 섞어 이해하면 Worker 안에서 DOM을 바꾸려 하거나 관찰 콜백에 긴 계산을 넣게 된다.

---

## 3. 관련 계층과 역할 구분

브라우저는 교차·DOM 변경을 관찰하여 메인 문서의 콜백에 기록을 전달한다. Worker는 문서 DOM이 없는 별도 실행 환경에서 메시지로 소통한다. 메인 문서 코드는 최종 UI·포커스·안내를 반영한다.

비동기 통지는 실행 위치 분리와 같은 말이 아니다. 관찰로 반복 확인 비용을 줄이는 일과 Worker로 계산 위치를 바꾸는 일을 각각 측정한다.

---

## 4. 세 도구의 역할을 비교한다

| 도구 | 알거나 처리하려는 것 | 콜백·코드가 있는 곳 |
| --- | --- | --- |
| IntersectionObserver | 대상과 기준 영역의 교차 상태 | 관찰하는 문서의 메인 스레드 콜백 |
| MutationObserver | 노드·속성·텍스트 등 설정한 DOM 변경 | 메인 스레드의 마이크로태스크 전달 |
| Dedicated Web Worker | 문서 DOM과 분리한 계산 | 별도 Worker 실행 환경 |

관찰자 콜백은 자동으로 Worker로 이동하지 않는다. 비동기 전달과 별도 스레드 실행은 같은 말이 아니다.

---

## 5. 관찰과 별도 계산을 선택하는 기준

관찰자는 브라우저에게 조건을 등록하고 변화의 정보를 전달받는 방식이다. 직접 반복 측정하는 코드보다 관심 대상을 분명하게 표현할 수 있지만 콜백 안의 일이 자동으로 가벼워지는 것은 아니다. Worker는 변화 관찰이 아니라 계산의 실행 환경을 분리한다.

DOM 변화, 화면 교차, 요소 크기 변화는 서로 다른 사실이다. 글꼴이나 viewport가 바뀌어 크기가 달라져도 DOM 속성이 바뀌지 않을 수 있고, 스크롤로 화면에 들어와도 DOM 자식 추가가 없을 수 있다. 감지하고 싶은 사실부터 구체적으로 적는다.

---

## 6. IntersectionObserver: 위치를 계속 직접 재지 않는다

스크롤할 때마다 모든 대상의 사각형을 읽는 대신 브라우저에 관찰을 요청한다. `root`는 기준 영역이며 기본값 null은 일반적인 문서 예제에서 viewport를 기준으로 한다. `rootMargin`은 기준 경계를 확장·축소하고 `threshold`는 교차 비율의 경계를 정한다. 관찰을 시작할 때 초기 상태를 포함한 통지가 올 수 있으므로 실제 교차 여부를 확인한다.

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>관찰자 실험</title>
  <style>.spacer { min-height: 120vh; }</style>
</head>
<body>
  <h1>두 가지 변화 관찰</h1>
  <p id="status" role="status"></p>
  <div class="spacer">아래로 스크롤하세요.</div>
  <section id="target"><h2>다음 학습</h2></section>
  <button id="change" type="button">상태 변경</button>
  <script>
    const target = document.querySelector('#target');
    const status = document.querySelector('#status');

    const intersection = new IntersectionObserver((entries) => {
      for (const entry of entries) {
        if (!entry.isIntersecting) continue;
        status.textContent = '다음 학습 영역이 기준 범위에 들어왔습니다.';
        intersection.unobserve(entry.target);
      }
    }, { rootMargin: '0px 0px 100px 0px', threshold: 0 });
    intersection.observe(target);

    const mutations = new MutationObserver((records) => {
      console.log(records.map((record) => record.attributeName));
    });
    mutations.observe(target, { attributes: true, attributeFilter: ['data-ready'] });
    document.querySelector('#change').addEventListener('click', () => {
      target.dataset.ready = 'true';
    });
    window.addEventListener('pagehide', () => {
      intersection.disconnect();
      mutations.disconnect();
    });
    window.addEventListener('pageshow', (event) => {
      if (!event.persisted) return;
      intersection.observe(target);
      mutations.observe(target, { attributes: true, attributeFilter: ['data-ready'] });
    });
  </script>
</body>
</html>
```

스크롤하여 기준 범위에 들어오면 한 번 안내하고 해당 대상의 관찰을 해제한다. 아래쪽 rootMargin을 늘렸으므로 실제 viewport 안에 보이기 전에도 통지될 수 있다. 교차 정보는 기본적으로 기하 관계다. 다른 요소에 가려졌는지, 사용자가 실제로 내용을 인지했는지까지 보장하는 증거로 사용하지 않는다. [Intersection Observer 표준](https://www.w3.org/TR/intersection-observer/)에서 관찰 조건을 확인한다.

---

## 7. IntersectionObserver의 설정과 통지를 해석한다

### root·rootMargin·threshold

root는 기준이 되는 스크롤 컨테이너 등을 지정하고 null이면 보통 viewport를 기준으로 한다. rootMargin으로 감지 경계를 넓히면 이미지·데이터 준비를 조금 일찍 시작하는 식의 설계가 가능하다. 실제 가시 영역에 들어온 때와 미리 감지한 때를 구별한다.

threshold는 교차 비율의 통지 경계를 나타낸다. `[0, 0.5, 1]`이라면 그 경계의 변화가 통지 조건에 관여한다. “50% 옵션을 주면 정확히 절반인 순간마다 모든 스크롤 픽셀에서 호출된다”는 뜻이 아니다. 브라우저는 변화 정보를 비동기로 전달하며 한 번의 콜백에 여러 대상·기록이 올 수 있다.

### Entry의 주요 정보

| 정보 | 의미 |
| --- | --- |
| target | 관찰 대상 |
| isIntersecting | 대상과 기준 영역의 교차 여부 |
| intersectionRatio | 대상 면적에 대한 교차 면적 비율 |
| boundingClientRect | 대상의 사각형 정보 |
| intersectionRect | 교차 영역 정보 |
| time | 기록과 연결된 시각 정보 |

경계 접촉이나 면적이 없는 대상 같은 경우를 고려하면 비율과 교차 여부를 무조건 같은 조건으로 대체하지 않는다. 기본 관찰은 픽셀 수준의 가림이나 사용자의 주의까지 판정하는 장치가 아니다.

### 여러 대상을 한 관찰자로 관리하기

같은 root·옵션을 가진 대상은 하나의 observer로 여러 개 observe할 수 있다. 완료된 대상만 unobserve하고 전체 기능이 끝나면 disconnect한다. 옵션을 바꾸는 요구가 생기면 새 관찰자의 생성·기존 관찰 해제를 설계한다. 무한 스크롤에는 중복 요청 방지와 사용자가 다시 시도할 수 있는 대체 경로도 필요하다.

---

## 8. MutationObserver: DOM 변경을 모아서 받는다

같은 예제에서 버튼은 data-ready 속성을 바꾼다. MutationObserver는 설정에 해당하는 변경 레코드를 나중에 전달한다. `attributes`, `childList`, `characterData`, `subtree` 등으로 무엇을 관찰할지 제한한다. 크기나 화면의 픽셀 변경을 관찰하는 API는 아니다.

변경 직후 같은 호출 스택 안에서 콜백이 즉시 실행되는 것으로 생각하지 않는다. DOM 변경은 이미 반영되었어도 레코드 전달은 마이크로태스크 시점에 이루어진다. 자세한 이벤트 루프 순서는 후속 JS Runtime / Browser Scheduling에서 다룬다.

관찰 콜백이 다시 같은 대상을 바꾸면 또 통지되어 반복될 수 있다. 변화를 구별하고 불필요한 재변경을 피한다. 외부 위젯이 바꾸는 영역을 감지하는 데 유용하지만, 스스로 소유한 애플리케이션 상태를 무조건 DOM에서 역추적하는 방식은 관리 비용을 늘릴 수 있다.

---

## 9. MutationObserver의 기록·범위·반복 방지

### 무엇을 관찰하는지 좁힌다

```javascript
const target = document.querySelector('#target');
const observer = new MutationObserver((records) => {
  for (const record of records) {
    if (record.type === 'attributes') {
      console.log(record.attributeName, record.oldValue);
    }
  }
});
if (target) {
  observer.observe(target, {
    attributes: true,
    attributeFilter: ['data-ready'],
    attributeOldValue: true,
  });
}
```

위 코드는 target 요소가 있을 때 하나의 속성만 관찰한다. oldValue가 필요하면 해당 옵션을 요청해야 한다. childList는 자식 추가·제거, characterData는 Text 같은 데이터 노드의 내용 변경을 관찰한다. subtree는 하위 영역으로 범위를 확장하지만 모든 문서·별도 경계까지 전역 관찰하는 뜻은 아니다.

textContent 대입으로 자식 Text가 교체되는 경우와 기존 `Text.data`가 바뀌는 경우는 기록 종류가 다를 수 있다. “화면 글자가 바뀌었다”는 인상만으로 characterData 하나면 충분하다고 판단하지 않는다.

### 콜백에서 동일한 변경을 다시 만들지 않는다

관찰 결과에 반응하여 또 같은 속성을 쓰면 새 기록이 생겨 반복될 수 있다. 정말 값이 달라져야 하는지 확인하고, 자신이 만든 변화와 외부 변화를 구별하는 규칙을 둔다. disconnect로 관찰을 잠시 끊는 방법은 놓칠 기록이 없는지도 검토해야 한다.

takeRecords는 전달 대기 중인 기록을 가져올 때 사용할 수 있다. disconnect와 기록 처리의 순서에 따라 관심 있던 마지막 변화를 놓칠 수 있으므로 종료 시 요구도 정한다. React처럼 상태를 소유하는 코드가 있다면 우선 그 상태 흐름으로 해결할 수 있는지 확인하고 DOM을 무조건 역추적하지 않는다.

---

## 10. Web Worker: DOM과 분리하여 계산하고 결과를 전달한다

아래 두 파일을 같은 디렉터리의 로컬 HTTP 서버로 제공한다. 메인 문서에는 실행 버튼과 결과 영역이 있고 Worker는 작은 합계 계산을 처리한다. 파일 URL 실행은 환경 제약이 있으므로 HTTP 환경을 사용한다.

```html
<!-- worker-demo.html -->
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>Worker 실험</title></head>
<body>
  <button type="button" id="run">합계 계산</button>
  <p id="result" role="status"></p>
  <script>
    const result = document.querySelector('#result');
    const worker = new Worker('./sum-worker.js');
    worker.addEventListener('message', (event) => {
      result.textContent = `합계: ${event.data}`;
    });
    worker.addEventListener('error', () => {
      result.textContent = '계산 중 오류가 발생했습니다.';
    });
    document.querySelector('#run').addEventListener('click', () => {
      result.textContent = '계산 중';
      worker.postMessage([10, 20, 30]);
    });
    window.addEventListener('pagehide', (event) => {
      if (!event.persisted) worker.terminate();
    });
  </script>
</body>
</html>
```

```javascript
// sum-worker.js
self.addEventListener('message', (event) => {
  const total = event.data.reduce((sum, value) => sum + value, 0);
  self.postMessage(total);
});
```

Worker에는 document가 없으므로 결과 DOM은 메인 문서에서 갱신한다. 메시지는 기본적으로 구조화 복제 규칙에 따라 전달되며 일반 객체가 동일한 참조로 공유되는 것은 아니다. 큰 데이터에서는 복제 비용과 transfer 가능한 데이터의 소유권 이전도 검토한다. Worker도 시작·메모리·통신 비용이 있으므로 이 예제처럼 작은 합계에는 실질적인 성능 이익을 기대하지 않는다.

UI 컴포넌트 단위로 Worker를 생성했다면 그 컴포넌트 수명이 끝날 때 terminate하는 등 소유 범위를 정한다. 여러 요청이 겹치면 응답 식별자를 두어 오래된 결과를 구별한다. Service Worker는 네트워크·수명 모델이 다른 도구이므로 이 페이지의 Dedicated Worker와 혼동하지 않는다.

---

## 11. Worker의 메시지 계약과 수명

### 함수 호출과 메시지 전달은 다르다

postMessage는 다른 실행 환경으로 데이터를 전달하는 계약이다. 일반 객체는 구조화 복제로 전달되어 같은 객체 참조를 그대로 공유하는 것이 아니다. 함수나 DOM 노드처럼 일반적으로 복제할 수 없는 값도 있다. 큰 데이터의 복제 비용이 계산 이익보다 클 수도 있다.

ArrayBuffer 같은 전송 가능한 자원은 transfer 목록으로 소유권을 이전할 수 있다. 원래 쪽의 buffer가 분리되는 등 사용 조건이 바뀌므로 “복사 없이 양쪽에서 마음대로 쓴다”로 이해하지 않는다. 공유 메모리는 별도 보안·동기화 요구가 있는 후속 개념이다.

### 요청 식별자로 오래된 결과를 구별한다

```javascript
let currentJob = 0;
function requestSum(worker, numbers) {
  const id = ++currentJob;
  worker.postMessage({ id, numbers });
}
function acceptSum(event) {
  if (event.data.id !== currentJob) return;
  console.log('현재 작업 결과:', event.data.total);
}
```

이 코드는 뒤의 Worker 예제를 확장할 때 사용하는 메인 쪽 조각이다. Worker도 `{ id, total }` 형태로 응답해야 계약이 맞는다. 화면에서 최신 요청만 유효하다면 식별자를 비교하여 이전 결과를 버릴 수 있다. 수신 데이터의 타입과 예상 필드도 검증한다.

### 취소·종료·오류가 별개의 요구다

terminate는 Worker 실행을 종료하는 강한 수단이고, 작업만 취소하려면 메시지 프로토콜과 계산 중단 지점을 별도로 설계할 수 있다. 종료된 Worker는 새로 만들어야 다시 사용할 수 있다. error 이벤트, 메시지 데이터 문제, 실행 파일 로딩 실패와 계산 결과의 업무 오류를 구별한다.

Dedicated Worker는 생성한 쪽의 작업을 위한 별도 환경이다. module Worker를 쓰면 import 등 모듈 로딩 요구가 추가된다. Worker에서도 CPU·메모리는 소비하므로 개수를 무제한 늘리는 것이 전체 성능 개선은 아니다.

---

## 12. 관찰 대상과 실행 장소를 선택하는 판단 표

### 같은 화면 변화도 원인은 다를 수 있다

스크롤로 요소가 viewport에 들어오는 일, 자식 노드가 추가되는 일, 긴 계산이 입력 처리를 늦추는 일은 다른 현상이다. IntersectionObserver는 기하학적 교차를, MutationObserver는 등록한 종류의 DOM 변경을, Worker는 별도 실행 환경의 계산을 다룬다. 화면이 바뀌었다는 인상 하나로 같은 도구를 고르지 않는다.

### 관찰자는 느린 콜백을 자동으로 분리하지 않는다

브라우저가 변화를 감지하고 나중에 콜백을 전달해도 그 콜백은 문서의 메인 실행 환경에서 일을 한다. 그 안에서 긴 계산이나 수많은 DOM 변경을 수행하면 입력·화면 갱신이 늦어질 수 있다. 브라우저가 맡는 관찰 비용과 개발자가 작성한 반응 코드의 비용을 나누어 측정한다.

### 도구가 보장하지 않는 의미를 덧붙이지 않는다

교차는 사용자의 실제 주의나 픽셀 가림 전체를 증명하지 않고, DOM 변경은 시각 크기 변화 전체를 통지하지 않으며, Worker는 계산량·메모리 비용을 없애지 않는다. 올바른 판단에는 각 도구의 보장 범위와 남는 책임이 필요하다.

---

## 13. IntersectionObserver의 경계·비율·초기 통지

### root의 목적과 rootMargin의 준비 구간

root는 기준 영역을 지정한다. rootMargin을 넓히면 실제 viewport에 보이기 전에 준비를 시작하는 구간을 만들 수 있다. 따라서 rootMargin 200px으로 받은 진입 통지를 “사용자가 이미 내용을 보았다”는 지표로 사용하지 않는다. 성능을 위한 선행 준비와 실제 노출의 의미를 구별한다.

### threshold는 매 픽셀의 스크롤 콜백이 아니다

threshold 목록은 교차 비율의 통지 경계와 관련된다. 브라우저는 초기 상태와 경계 변화에 따른 기록을 전달할 수 있고, 한 번에 여러 Entry가 올 수 있다. 코드를 entries[0]만 가정하여 나머지 대상을 무시하거나, observe 직후 첫 콜백은 무조건 진입이라고 처리하지 않는다.

### 실험 1. 초기 상태·진입·비율 변화를 로그로 보기

```html
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8"><title>교차 비율 관찰</title>
  <style>
    #viewport { height: 220px; overflow: auto; border: 2px solid; }
    .space { height: 260px; }
    #target { height: 120px; background: #cde5ff; }
  </style>
</head>
<body>
  <h1>스크롤 영역 안의 교차</h1>
  <div id="viewport"><div class="space"></div><div id="target">대상</div><div class="space"></div></div>
  <pre id="output"></pre>
  <script>
    const viewport = document.querySelector('#viewport');
    const target = document.querySelector('#target');
    const output = document.querySelector('#output');
    const observer = new IntersectionObserver((entries) => {
      for (const entry of entries) {
        output.textContent += JSON.stringify({
          intersecting: entry.isIntersecting,
          ratio: Number(entry.intersectionRatio.toFixed(2)),
        }) + '\n';
      }
    }, { root: viewport, threshold: [0, .5, 1] });
    observer.observe(target);
  </script>
</body>
</html>
```

처음에는 대상이 스크롤 영역 아래에 있어 false·0인 상태를 관찰할 수 있다. 스크롤하면 일부 진입, 절반 경계, 전체 교차와 같은 변화가 기록된다. 빠르게 움직이면 관찰된 상태 사이를 건너뛰어 모든 중간 비율이 출력되는 것은 아니다. 콜백 횟수를 스크롤 거리의 정확한 측정으로 사용하지 않는다.

### isIntersecting과 ratio의 반례

기준과 대상의 경계가 닿는 경우에는 교차 여부와 양의 면적 비율이 단순한 같은 조건이 아닐 수 있다. 면적이 0인 대상에도 비율 계산의 별도 조건이 있다. [Intersection Observer 처리 모델](https://www.w3.org/TR/intersection-observer/#update-intersection-observations-algo)을 근거로, 필요한 조건이 “기하 진입”인지 “충분한 면적의 교차”인지 정한다.

### 관찰 종료와 반복 요청 방지

한 번만 로드할 대상은 완료 뒤 unobserve할 수 있고 기능 전체가 끝나면 disconnect한다. 무한 목록의 sentinel이 계속 보이는 상태라면 이전 요청이 끝나기 전에 중복 요청을 시작하지 않는 정책이 필요하다. 관찰자가 있다는 이유로 사용자에게 다음 페이지를 불러올 버튼이나 오류 재시도 경로가 불필요해지는 것은 아니다.

---

## 14. MutationObserver는 현재 DOM과 변경 기록을 다르게 제공한다

### oldValue와 현재 값

변경 기록은 관심 있는 변화의 종류·대상·이전 값 등을 제공한다. 모든 기록에 그 순간의 newValue 스냅샷이 포함되는 일반 변경 로그는 아니다. 여러 변경 뒤 콜백에서 target의 현재 속성을 읽으면 최종 값이 보일 수 있다. 과거 기록과 현재 객체의 상태를 구별해야 한다.

### 실험 2. 한 속성을 두 번 바꾸기

```javascript
{
  const target = document.createElement('div');
  const observer = new MutationObserver((records) => {
    console.log(records.map((record) => ({
      old: record.oldValue,
      current: record.target.getAttribute('data-step'),
    })));
    observer.disconnect();
  });
  observer.observe(target, { attributes: true, attributeOldValue: true });
  target.setAttribute('data-step', 'a');
  target.setAttribute('data-step', 'b');
  console.log('호출 스택의 동기 변경 완료');
}
```

동기 로그 뒤에 관찰 콜백을 받는다. 이전 값은 null과 a로 다르지만, 두 기록에서 현재 속성을 읽으면 모두 b일 수 있다. 관찰 대상이 문서에 연결되어 있지 않아도 해당 노드의 DOM 변경을 관찰할 수 있다. 화면에 보이는지를 관찰하는 API가 아니라는 사실도 함께 확인한다.

### 실험 3. 글자 변경과 자식 교체

```javascript
{
  const paragraph = document.createElement('p');
  paragraph.textContent = '처음';
  const observer = new MutationObserver((records) => {
    console.log(records.map((record) => record.type));
    observer.disconnect();
  });
  observer.observe(paragraph, {
    subtree: true, childList: true, characterData: true, characterDataOldValue: true,
  });
  paragraph.firstChild.data = '같은 Text 변경';
  paragraph.textContent = '새 자식으로 교체';
}
```

첫 작업은 기존 Text의 characterData 변경, 둘째는 p의 자식 구성 교체를 관찰한다. 화면상으로는 둘 다 글자 변경처럼 보일 수 있지만 기록 종류가 다르다. 어떤 코드가 대상에 변화를 주는지에 맞게 관찰 옵션을 정한다. [MutationRecord](https://developer.mozilla.org/en-US/docs/Web/API/MutationRecord)의 필드와 기록 종류가 근거다.

### 실험 4. takeRecords와 종료

```javascript
{
  const target = document.createElement('div');
  const observer = new MutationObserver(() => console.log('비동기 전달'));
  observer.observe(target, { attributes: true });
  target.setAttribute('data-ready', 'true');
  const pending = observer.takeRecords();
  console.log(pending.length);
  observer.disconnect();
}
```

대기 기록을 직접 가져온 길이는 1이다. takeRecords는 콜백을 직접 실행하는 API가 아니라 기록 큐를 가져오는 API다. 종료 직전의 변경이 중요하면 필요한 기록을 처리한 뒤 관찰을 해제한다. [DOM MutationObserver 규칙](https://dom.spec.whatwg.org/#interface-mutationobserver)을 기준으로 종료와 기록 처리의 순서를 설계한다.

---

## 15. Worker의 별도 환경과 메시지 계약

### 같은 객체 참조를 함수 인자처럼 넘기는 것이 아니다

postMessage의 데이터는 기본적으로 구조화 복제 규칙을 따른다. 일반 객체를 보냈다고 같은 객체 참조를 양쪽이 직접 수정하는 것은 아니다. 함수·DOM 노드 등은 일반적인 복제 대상이 아니며, 일부 자원은 transfer로 소유권을 이전할 수 있다. 데이터의 크기와 전달 빈도도 성능의 일부다.

### 실험 5. 계산과 DOM 반영을 분리하기

아래 worker.js와 index.html을 같은 폴더의 로컬 HTTP 서버로 제공한다. 파일을 직접 여는 환경의 제약과 혼동하지 않는다.

```javascript
// worker.js
self.addEventListener('message', (event) => {
  const { id, limit } = event.data;
  if (!Number.isSafeInteger(limit) || limit < 1 || limit > 50000000) {
    self.postMessage({ id, error: '계산 범위를 확인하세요.' });
    return;
  }
  let total = 0;
  for (let number = 1; number <= limit; number += 1) total += number;
  self.postMessage({ id, total, hasDocument: typeof document !== 'undefined' });
});
```

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>Worker 계산</title></head>
<body>
  <label for="limit">1부터 더할 수</label>
  <input id="limit" type="number" min="1" max="50000000" value="10000000">
  <button id="start" type="button">계산</button>
  <button id="cancel" type="button">계산 중단</button>
  <p id="result" role="status">대기</p>
  <script>
    const input = document.querySelector('#limit');
    const result = document.querySelector('#result');
    let worker = null;
    let currentId = 0;
    function stop() {
      currentId += 1;
      worker?.terminate();
      worker = null;
      result.textContent = '중단';
    }
    function getWorker() {
      if (worker) return worker;
      const created = new Worker('./worker.js');
      created.addEventListener('message', (event) => {
        if (worker !== created || event.data.id !== currentId) return;
        result.textContent = event.data.error ??
          `합계 ${event.data.total}, Worker에 document 존재: ${event.data.hasDocument}`;
      });
      created.addEventListener('error', () => {
        if (worker !== created) return;
        created.terminate();
        worker = null;
        result.textContent = 'Worker 실행에 실패했습니다.';
      });
      worker = created;
      return created;
    }
    document.querySelector('#start').addEventListener('click', () => {
      const limit = input.valueAsNumber;
      if (!Number.isSafeInteger(limit) || limit < 1 || limit > 50000000) {
        result.textContent = '1부터 50000000 사이의 정수를 입력하세요.';
        return;
      }
      const id = ++currentId;
      result.textContent = '계산 중';
      getWorker().postMessage({ id, limit });
    });
    document.querySelector('#cancel').addEventListener('click', stop);
    window.addEventListener('pagehide', stop);
  </script>
</body>
</html>
```

### 결과와 적용 범위

Worker에서 document 존재는 false다. 계산 결과를 메시지로 받아 메인 문서에서 textContent를 바꾼다. 메시지의 id와 Worker 인스턴스 둘 다 확인하여 이전 작업이나 종료한 인스턴스의 결과를 현재 화면에 쓰지 않는다. 취소는 해당 Worker의 대기 작업까지 포함하여 실행 환경을 끝내며 다음 계산에서는 새 인스턴스를 만든다.

합계는 수식으로 더 빠르게 계산할 수도 있지만 이 예제는 계산 위치를 관찰하려고 반복문을 사용했다. 실제 병목을 해결할 때는 알고리즘 개선과 Worker 이전 비용을 함께 비교한다. 작은 계산에 Worker를 만들면 시작·전달 비용이 이익보다 클 수 있다.

### terminate와 협력적 취소

terminate는 실행을 끝내는 수단이다. 현재 계산만 중단하는 프로토콜을 메시지로 만들 수도 있지만 Worker가 긴 동기 루프 안에 있으면 새 취소 메시지를 처리할 기회를 얻지 못할 수 있다. 작업 분할·중단 지점 또는 Worker 종료 중 요구에 맞는 방법을 고른다. [Worker.terminate](https://developer.mozilla.org/en-US/docs/Web/API/Worker/terminate)의 즉시 종료 계약과 구별한다.

---

## 16. 세 도구를 실제 비용과 사용자 흐름으로 검증하기

### 관찰 범위를 줄이고 자기 변경의 반복을 막는다

문서 전체 subtree의 모든 속성을 관찰하는 설정은 목적보다 넓을 수 있다. 대상·필터·변경 종류를 제한하고, 콜백에서 같은 속성을 다시 쓰며 반복 통지를 만들지 않는지 확인한다. 자체 state를 이미 소유한 앱에서는 먼저 그 상태 흐름으로 해결할 수 있는지 검토한다.

### Worker 이전 전후의 전체 시간을 비교한다

계산만이 아니라 시작·복제·메시지·최종 DOM 반영 시간을 포함한다. 메인 스레드의 입력 반응이 개선되었는지, 메모리와 전체 CPU 사용이 늘었는지도 본다. 별도 실행이 항상 전체 작업을 더 빨리 끝내는 것은 아니다.

### 대기·완료·실패·중단을 사용자가 구별한다

관찰 기반 지연 로딩에는 다시 시도할 경로를, 계산에는 중단과 오류 안내를 제공한다. 관찰 이벤트의 모든 내부 기록을 보조 기술에 연속으로 읽게 하기보다 사용자의 작업과 연결된 상태만 알린다. 시각적 변화와 접근 가능한 피드백을 함께 검증한다.

---

## 17. 실무 적용과 검증

관찰 범위를 작게 하고 필요가 끝나면 unobserve·disconnect한다. 콜백에서는 최소 작업만 수행하고 Performance로 긴 작업을 확인한다. Worker 도입 전후에는 계산 시간뿐 아니라 시작·데이터 전달·최종 DOM 갱신까지 포함해 비교한다. 지연 로딩을 쓰더라도 사용자가 스크롤하지 못하면 내용을 얻을 수 없는 구조를 피하고, 로딩·완료·실패를 적절하게 안내한다.

---

## 18. 핵심 정리

<callout icon="✅" color="green_bg">
IntersectionObserver는 교차 상태, MutationObserver는 DOM 변경, Worker는 별도 환경의 계산을 다룬다. 무엇을 감지·실행하는지와 어디에서 실행되는지를 먼저 구별한 뒤 비용과 종료를 설계한다.
</callout>

---

## 출처

- [W3C — Intersection Observer](https://www.w3.org/TR/intersection-observer/)
- [WHATWG — MutationObserver](https://dom.spec.whatwg.org/#mutationobserver)
- [WHATWG — Web Workers](https://html.spec.whatwg.org/multipage/workers.html)
- [MDN — Structured clone algorithm](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)

### 이번 설명의 근거와 적용 범위

- [W3C — Intersection Processing Model](https://www.w3.org/TR/intersection-observer/#update-intersection-observations-algo): 초기 상태·경계·비율·교차 판정.
- [WHATWG — MutationObserver](https://dom.spec.whatwg.org/#interface-mutationobserver): 기록 큐·통지·takeRecords·disconnect의 규칙.
- [MDN — MutationRecord](https://developer.mozilla.org/en-US/docs/Web/API/MutationRecord): 변경 종류와 oldValue 등 기록 필드.
- [WHATWG — Web Workers](https://html.spec.whatwg.org/multipage/workers.html): 별도 전역 환경·메시지·Worker 수명.
- [MDN — Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers): 문서 DOM 제한과 데이터 전달.
- [MDN — terminate](https://developer.mozilla.org/en-US/docs/Web/API/Worker/terminate): 실행 환경 종료의 범위.

---

## 19. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 화면 근처 진입, 외부 위젯의 자식 추가, 큰 계산 지연에 각각 어떤 도구를 고르겠는가?
2. MutationObserver 콜백에 긴 계산을 넣으면 UI가 자동으로 보호되는가? 실행 위치를 근거로 설명하라.
3. IntersectionObserver가 교차를 알렸다는 이유만으로 사용자가 광고를 실제로 봤다고 단정할 수 없는 이유는 무엇인가?
4. Worker에서 document를 사용할 수 없는 이유와 결과를 화면에 반영하는 경로를 설명하라.
