---
track: B
notion: https://app.notion.com/p/415178c4afac8150b694e5fe08897c7c
---

# fetch / URL / Location / History


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · fetch / URL / Location / History

주소 구성·요청과 응답·문서 이동·기록 복원을 각각 배우고 공유 가능한 화면 상태로 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- URL의 origin·path·query·fragment와 상대 주소를 설명할 수 있다.
- 요청 메서드·헤더·본문과 응답 상태·본문 해석을 구별할 수 있다.
- 실패·취소·오래된 응답·CORS의 문제 계층을 판단할 수 있다.
- Location의 이동과 History의 기록 갱신·복원을 목적에 맞게 고를 수 있다.
- 초기 진입·제출·뒤로 가기·새로고침에서 같은 화면 상태를 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

검색어가 담긴 주소를 만들기, 검색 결과를 받기, 주소 표시줄을 갱신하기는 연결되어 있지만 각각 다른 기능이다. URL을 바꿨다고 데이터가 자동으로 갱신되거나 fetch를 호출했다고 페이지가 이동하지는 않는다. 이 경계를 이해해야 새로고침·공유·뒤로 가기가 일관적인 화면을 만들 수 있다.

---

## 3. 관련 계층과 역할 구분

URL API는 문자열 주소를 구조화한다. Fetch는 요청·응답과 브라우저의 접근 정책을 다룬다. Location과 History는 현재 탐색과 세션 기록을 관리한다. DOM·React 등의 UI 코드는 결과를 화면에 표현하고 서버는 경로·요청 형식에 맞는 응답을 제공한다.

하나의 버튼이 이 여러 계층을 연결할 수 있지만 각 API가 다른 계층까지 자동으로 수행하는 것은 아니다. 서버 구현과 상세 HTTP 이론은 다른 트랙의 범위다.

---

## 4. 네 API의 역할

| API | 맡은 일 | 자동으로 하지 않는 일 |
| --- | --- | --- |
| `URL`, `URLSearchParams` | 주소 해석·구성과 쿼리 인코딩 | 요청·페이지 이동 |
| `fetch` | 요청과 응답 처리 | 화면 렌더링·주소 표시줄 변경 |
| `location` | 현재 주소 조회와 문서 이동 | 애플리케이션 상태 설계 |
| `history` | 세션 기록 이동·현재 문서의 기록 갱신 | URL에 맞는 화면 구현 |

`new URL('/search', location.href)`는 상대 주소를 기준 주소에 대해 해석한다. `searchParams.set('q', value)`는 쿼리 값을 인코딩하므로 문자열 연결로 `&`, 공백, 한글 등을 잘못 처리하는 일을 줄인다. 주소 객체 수정만으로 실제 location이 바뀌지는 않는다.

---

## 5. URL: 주소의 구성 요소와 인코딩

### origin·path·query·fragment를 구별한다

```javascript
const url = new URL('https://example.com:8443/notes/web?q=DOM&tag=a&tag=b#examples');
console.log(url.origin); // https://example.com:8443
console.log(url.pathname); // /notes/web
console.log(url.searchParams.get('q')); // DOM
console.log(url.searchParams.getAll('tag')); // ['a', 'b']
console.log(url.hash); // #examples
```

origin은 일반적으로 스킴·호스트·포트의 조합이다. path는 자원 경로, query는 추가 매개변수, fragment는 문서 내부 위치 등을 표현한다. HTTP 요청에 fragment가 그대로 서버의 경로·쿼리처럼 전송되는 것은 아니다. URL에 들어 있다는 이유만으로 모든 부분이 서버가 처리할 입력이라고 가정하지 않는다.

같은 호스트라도 스킴이나 유효 포트가 다르면 origin이 달라질 수 있다. 브라우저의 읽기 허용과 History 주소 제한을 판단할 때 이 경계를 먼저 확인한다. Cookie의 사이트·도메인 범위와도 동일한 분류가 아니다.

### 상대 주소에는 기준이 필요하다

```javascript
const base = 'https://example.com/study/index.html';
console.log(new URL('notes', base).href); // https://example.com/study/notes
console.log(new URL('/notes', base).href); // https://example.com/notes
```

앞의 슬래시 하나에 따라 현재 디렉터리 기준과 origin 루트 기준이 달라진다. 현재 문서의 base 요소도 상대 주소 해석에 관여할 수 있으므로 location.href와 문서의 baseURI를 목적에 맞게 선택한다. 배포 경로가 바뀌었을 때 요청이 잘못되는 문제를 문자열 연결로만 추적하지 않는다.

### 쿼리 값은 구조화하여 넣는다

```javascript
const url = new URL('/search', 'https://example.com');
url.searchParams.set('q', 'HTML & DOM');
url.searchParams.append('tag', 'browser');
url.searchParams.append('tag', 'accessibility');
console.log(url.href);
```

set은 해당 키의 값을 설정하고 append는 같은 이름의 항목을 추가할 수 있다. get은 첫 값, getAll은 모든 값을 읽는다. 이미 인코딩한 문자열을 다시 set하면 의도와 달리 이중 인코딩될 수 있으므로 보통 원래 값을 API에 전달한다.

---

## 6. Request와 Response: 통신 계약을 구성한다

### 메서드·헤더·본문은 역할이 다르다

```javascript
async function saveNote(note) {
  const response = await fetch('/api/notes', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(note),
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  if (response.status === 204) return null;
  return response.json();
}
```

이 예제는 해당 경로를 처리하는 서버가 있어야 한다. JSON.stringify는 JavaScript 데이터를 요청 본문의 문자열로 바꾸고 Content-Type은 그 형식을 서버에 알린다. Response의 json은 응답 내용을 해석할 뿐 요청의 JSON 변환을 대신하지 않는다. 204처럼 본문이 없는 응답에 무조건 json을 호출하면 해석 단계에서 실패할 수 있다.

FormData를 body로 보낼 때는 브라우저가 multipart 경계 정보를 포함한 헤더를 만들도록 일반적으로 Content-Type을 직접 고정하지 않는다. 요청 형식은 서버 계약과 맞춰야 하며 fetch가 모든 객체를 자동으로 원하는 전송 형식으로 바꾸는 것은 아니다.

### 응답 본문은 스트림을 소비한다

Response 본문은 기본적으로 한 번 읽는 흐름이다. json을 읽고 다시 text를 읽으면 이미 소비된 본문 문제가 생길 수 있다. 여러 소비가 정말 필요하면 읽기 전에 clone 등 해당 API의 규칙을 검토한다. 대량 응답을 복제·전부 보관하면 메모리 비용도 늘어난다.

---

## 7. fetch의 성공을 여러 단계로 나눈다

fetch는 보통 응답 상태와 헤더를 사용할 수 있을 때 Response를 전달한다. 본문 전체를 JSON으로 읽는 `response.json()`은 다시 비동기 작업이다. 네트워크 실패와 HTTP 실패를 구별해야 한다. 404·500 응답도 대체로 fetch의 Promise를 이행하므로 `response.ok`나 status를 확인하고 애플리케이션의 실패로 처리한다. JSON이 잘못되어 본문 해석 단계에서 실패하는 경우도 있다. [Fetch 표준](https://fetch.spec.whatwg.org/)이 요청·응답과 보안 처리를 정의한다.

```javascript
async function loadNotes(query, signal) {
  const url = new URL('/api/notes', location.origin);
  url.searchParams.set('q', query);
  const response = await fetch(url, { signal });
  if (!response.ok) {
    throw new Error(`응답 상태: ${response.status}`);
  }
  return response.json();
}
```

이 함수는 `/api/notes`가 JSON을 반환하는 서버를 전제로 하는 통신 패턴이다. 서버가 없는 정적 HTML에 붙이면 API 데이터가 생기지 않는다. 응답 본문은 기본적으로 한 번 소비하므로 같은 Response에서 json과 text를 연속으로 읽지 않는다. 이전 요청을 중단하려면 AbortController의 signal을 전달할 수 있지만, 중단이 이미 서버에서 처리한 작업까지 되돌리지는 않는다.

다른 origin의 응답을 읽으려면 CORS 규칙과 서버의 허용이 필요하다. `mode: 'no-cors'`는 응답 내용을 자유롭게 읽는 우회 방법이 아니다. 쿠키 등 자격 증명 전송 여부도 credentials 설정과 서버·쿠키 정책을 함께 확인해야 한다.

---

## 8. 실패·취소·경쟁 상태를 따로 설계한다

| 상황 | 관찰 위치 | UI 설계 |
| --- | --- | --- |
| 네트워크·정책 실패 | fetch 거부 | 연결·허용 조건에 맞는 재시도 안내 |
| HTTP 4xx·5xx | Response 상태 | 실패 상태·서버 계약에 맞는 메시지 |
| 잘못된 JSON | 본문 해석 실패 | 형식·스키마 오류와 데이터 상태 구별 |
| 요청 취소 | AbortSignal 관련 실패 | 사용자가 취소한 흐름을 일반 오류와 구별 |
| 이전 응답이 늦게 도착 | 요청 식별·현재 상태 | 오래된 결과가 새 화면을 덮지 않게 처리 |

### AbortController의 역할

```javascript
const controller = new AbortController();
const pending = fetch('/api/notes', { signal: controller.signal });
controller.abort();
pending.catch((error) => console.log(error.name));
```

이 조각은 시작한 요청에 중단 신호를 보내는 실험이다. 취소는 통신과 본문 처리 등 관련 흐름을 중단하는 수단이지 서버의 이미 완료된 쓰기를 롤백하는 트랜잭션이 아니다. 자동 재시도까지 구현할 때는 요청이 중복 실행되어도 되는지 별도로 판단한다.

### CORS는 응답 읽기 권한과 연결된다

다른 origin으로 요청할 수 있는 조건과 JavaScript가 응답 내용을 읽을 수 있는 조건을 구별한다. 요청 형식에 따라 사전 요청(preflight)이 필요할 수 있고 서버는 올바른 허용 응답을 제공해야 한다. credentials include를 사용해도 서버·쿠키·브라우저 정책을 무시할 수 없다. 상세한 네트워크·보안 규칙은 후속 Network / Security에서 확장한다.

---

## 9. 예제: 서버 없이 요청과 기록 변화를 구별하기

아래 파일을 로컬 HTTP 서버로 제공하여 연다. `data:` URL의 JSON을 fetch하므로 별도의 API 서버는 필요 없다. 실제 검색 서버의 지연·CORS·오류까지 재현하는 예제는 아니다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>주소와 데이터</title></head>
<body>
  <h1>노트 검색</h1>
  <form id="search">
    <label for="query">검색어</label>
    <input id="query" name="q">
    <button type="submit">검색</button>
  </form>
  <p id="status" role="status"></p>
  <ul id="results"></ul>
  <script>
    const form = document.querySelector('#search');
    const input = document.querySelector('#query');
    const list = document.querySelector('#results');
    const status = document.querySelector('#status');
    let requestVersion = 0;

    async function renderFromAddress() {
      const version = ++requestVersion;
      const url = new URL(location.href);
      const query = url.searchParams.get('q') ?? '';
      input.value = query;
      list.replaceChildren();
      status.textContent = '불러오는 중';
      try {
        const response = await fetch('data:application/json,%5B%22HTML%22,%22DOM%22%5D');
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const notes = await response.json();
        if (version !== requestVersion) return;
        const matches = notes.filter((name) => name.toLowerCase().includes(query.toLowerCase()));
        for (const name of matches) {
          const item = document.createElement('li');
          item.textContent = name;
          list.append(item);
        }
        status.textContent = `${matches.length}개 결과`;
      } catch (error) {
        if (version !== requestVersion) return;
        status.textContent = '결과를 불러오지 못했습니다.';
        console.error(error);
      }
    }

    form.addEventListener('submit', (event) => {
      event.preventDefault();
      const url = new URL(location.href);
      url.searchParams.set('q', input.value);
      history.pushState(null, '', url);
      renderFromAddress();
    });
    window.addEventListener('popstate', renderFromAddress);
    renderFromAddress();
  </script>
</body>
</html>
```

초기 진입에서는 현재 주소의 q를 읽어 입력과 목록을 만든다. 제출하면 pushState로 기록을 추가한 뒤 렌더링 함수를 직접 호출한다. pushState 자체가 popstate를 발생시키지 않기 때문이다. 뒤로 가기로 해당 기록을 이동하면 popstate 리스너가 주소에서 상태를 다시 읽는다.

`requestVersion`은 예전 비동기 결과가 나중 요청의 화면을 덮지 않게 한다. 요청을 시작한 순서와 끝나는 순서는 같다고 보장할 수 없다. 실제 서버를 사용할 때도 이 경쟁 상황을 설계해야 한다.

---

## 10. Location과 History의 선택

`location.assign(url)`은 일반적인 문서 이동을 요청하고, `location.replace(url)`은 현재 기록을 교체하는 이동을 요청한다. fragment만 달라지는 이동처럼 같은 문서 안에서 처리되는 경우도 있다. `history.pushState`와 `replaceState`는 현재 문서의 기록·주소를 갱신하며 그 자체로 새 문서를 fetch하지 않는다. 새 주소에는 같은 origin 등 제한이 있다.

공유 가능한 상태는 쿼리에, 일시적인 보조 상태는 history.state 등을 검토하되 큰 데이터와 민감 정보를 무분별하게 저장하지 않는다. 새 주소를 직접 열거나 새로고침해도 서버가 올바른 문서를 제공하는지 확인한다. 클라이언트 기록만 바꾸어서는 서버 라우팅이 구성되지 않는다.

---

## 11. History: 화면 상태를 기록과 함께 복원하기

### push와 replace를 사용자 작업의 의미로 고른다

새로운 검색 실행처럼 뒤로 가기로 이전 상태를 다시 보고 싶으면 pushState를 검토한다. 최초 URL의 기본값을 정리하거나 하나의 작업 중 임시 상태를 갱신한다면 replaceState가 적합할 수 있다. 키 입력마다 기록을 하나씩 추가하면 뒤로 가기가 입력 기록 전체를 거슬러 가는 불편을 줄 수 있다.

pushState·replaceState는 새 문서를 받지 않고 현재 문서의 기록을 관리한다. location.assign·replace는 이동을 요청한다. 두 계열의 replace가 모두 이름이 같다고 같은 처리 과정인 것은 아니다.

### URL을 상태의 기준으로 사용할 때

초기 진입, 새로고침, 뒤로 가기, 앞으로 가기에서 모두 같은 URL 해석 함수를 사용하면 화면의 재현성을 높일 수 있다. q 같은 공유 가능한 필터를 주소에 두고, 매번 주소를 읽어 입력·결과를 맞춘다. 민감 데이터나 큰 응답 전체를 URL에 넣는 것은 별도 노출·크기 문제를 만든다.

history.state에는 직렬화 가능한 보조 데이터를 둘 수 있지만 영구 저장소처럼 의존하지 않는다. 기록 이동으로 복원된 화면이 데이터 만료나 실패를 어떻게 처리할지도 정한다. 브라우저의 스크롤 복원과 포커스 이동도 화면 이동 경험에 영향을 준다.

### 주소·데이터·화면 세 축을 함께 검사하기

주소만 바뀌고 화면이 그대로면 렌더링 연결을 확인한다. 데이터만 바뀌고 주소가 그대로면 공유·새로고침 요구를 확인한다. 뒤로 가기만 틀리면 기록 이동 이벤트와 초기화 경로를 확인한다. 서버에서 직접 연 주소만 실패하면 서버 문서 제공 경로를 확인한다. 라우터 도구를 도입해도 이 요구 자체가 사라지는 것은 아니다.

---

## 12. 네트워크 준비·응답 상태·본문 해석을 독립적으로 다루기

### fetch의 이행과 업무 성공은 다르다

fetch가 Response를 전달했다는 것은 서버의 업무 처리가 성공했다는 뜻이 아니다. 404·500도 응답이며, 응답 본문에 업무 오류가 들어 있을 수도 있다. 반대로 네트워크 실패나 브라우저 정책으로 응답을 사용할 수 없는 경우에는 Response를 읽는 단계에 도달하지 못할 수 있다. [Fetch 사용 문서](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)의 네트워크 실패와 HTTP 상태 구분을 기준으로 처리한다.

### Response를 만든 실험과 실제 네트워크 실험을 구별한다

new Response로 상태와 본문을 만든 예제는 본문 소비·상태 판정을 재현하기 좋다. 그러나 DNS·연결·CORS·쿠키·실제 서버 지연을 시험하는 것은 아니다. 어떤 계층을 검증한 코드인지 명시해야 결과를 다른 환경으로 과장하지 않는다.

### 실험 1. 상태와 JSON 실패는 다른 조건이다

```javascript
async function inspectResponses() {
  const cases = [
    new Response('{"message":"없음"}', { status: 404 }),
    new Response('올바르지 않은 JSON', { status: 200 }),
    new Response(null, { status: 204 }),
  ];
  for (const response of cases) {
    console.log('상태:', response.status, 'ok:', response.ok);
    if (!response.ok) { console.log('HTTP 실패로 분기'); continue; }
    if (response.status === 204) { console.log('본문 없는 성공'); continue; }
    try { console.log(await response.json()); }
    catch (error) { console.log('본문 해석 실패:', error.name); }
  }
}
inspectResponses();
```

404는 ok false로 분기한다. 200은 HTTP 상태 기준으로 성공 범위지만 JSON 해석에서 실패한다. 204는 본문 없는 성공으로 처리한다. 이 세 결과를 catch 하나에서 전부 “인터넷 연결 없음”으로 표시하면 사용자가 잘못된 대응을 하게 된다. 재시도 가능성과 오류 원인은 서버 계약과 상태에 맞게 결정한다.

---

## 13. 응답 본문을 소비한다는 의미

### bodyUsed와 여러 번 읽기

본문 읽기는 일반적인 객체 프로퍼티 조회처럼 몇 번이고 동일한 값을 꺼내는 작업이 아니다. Response는 본문 스트림과 연결되고, 한 번 소비한 뒤 다시 json·text를 호출하면 실패할 수 있다. JSON으로 변환한 결과를 여러 곳에서 사용하려면 보통 해석 결과를 보관한다.

### 실험 2. 한 번 읽기와 clone

```javascript
async function inspectBody() {
  const response = new Response('{"topic":"DOM"}');
  const copy = response.clone();
  console.log(response.bodyUsed);
  console.log(await response.json());
  console.log(response.bodyUsed);
  try { await response.text(); }
  catch (error) { console.log('다시 읽기 실패:', error.name); }
  console.log(await copy.text());
}
inspectBody();
```

처음 bodyUsed는 false, 읽은 뒤에는 true다. 원본을 다시 읽는 작업은 실패하지만 읽기 전에 만든 copy는 별도 소비 경로로 텍스트를 읽을 수 있다. clone은 아무 비용 없이 무제한 복사하는 보관 전략이 아니므로 큰 응답에서 불필요하게 남발하지 않는다. [Response.bodyUsed](https://developer.mozilla.org/en-US/docs/Web/API/Response/bodyUsed)와 본문 읽기 계약을 함께 확인한다.

### 헤더를 확인하는 것과 본문을 읽는 것

상태·헤더를 확인할 때 본문 JSON이 이미 준비되었다고 생각하지 않는다. json은 별도 비동기 해석 작업이고 그 과정에서도 중단·오류가 생길 수 있다. 사용자가 화면을 바꾼 경우에는 fetch가 끝난 시점뿐 아니라 본문을 다 읽은 뒤에도 결과가 여전히 유효한지 확인한다.

---

## 14. URL 구성과 History 상태를 재현 가능한 입력으로 만들기

### 실험 3. 쿼리의 반복 항목·공백·특수문자

```javascript
{
  const url = new URL('https://example.com/notes');
  url.searchParams.set('q', 'HTML & DOM');
  url.searchParams.append('tag', 'browser');
  url.searchParams.append('tag', 'accessibility');
  const restored = new URL(url.href);
  console.log(restored.searchParams.get('q'));
  console.log(restored.searchParams.getAll('tag'));
  console.log(restored.hash);
}
```

왕복 후 q는 원래 문자열, tag는 두 값의 목록이다. 내부 인코딩 문자열을 수동으로 합치는 대신 입력값을 API에 전달하고, 주소를 다시 읽었을 때 같은 의미를 얻는지 검사한다. set과 append의 차이를 놓치면 필터 여러 개를 하나로 덮거나 중복 기록을 만들 수 있다. [URLSearchParams 표준](https://url.spec.whatwg.org/#urlsearchparams)을 형식의 근거로 삼는다.

### URL은 외부에서 들어오는 입력이다

사용자가 쿼리를 직접 수정하거나 오래된 공유 주소를 열 수 있다. page·sort·filter 값에 기본값과 허용 범위를 두고, 주소에서 읽은 문자열을 그대로 HTML로 파싱하지 않는다. 주소가 유효한 URL이라는 것과 애플리케이션이 허용하는 필터라는 것은 다른 검증이다.

### History에 저장할 수 있는 상태에도 계약이 있다

pushState의 state는 직렬화 가능한 값을 기준으로 사용한다. 함수·DOM 노드처럼 그대로 보관할 수 없는 값을 넣으면 예외가 날 수 있다. 크기·빈도 정책도 브라우저 구현의 영향을 받으므로 큰 응답의 영구 캐시로 삼지 않는다. [pushState의 state·URL 조건](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState)을 확인한다.

### 실험 4. 주소 갱신이 요청을 자동 실행하지 않는다

로컬 HTTP 문서의 콘솔에서 실행한다. 쿼리만 바꾸므로 현재 파일 경로는 유지된다.

```javascript
{
  const url = new URL(location.href);
  url.searchParams.set('q', 'DOM');
  history.pushState({ selected: 'dom' }, '', url);
  console.log(location.search, history.state);
  try {
    history.pushState({ callback() {} }, '', location.href);
  } catch (error) {
    console.log('저장할 수 없는 상태:', error.name);
  }
}
```

첫 호출 뒤 주소와 state는 달라져도 검색 결과 DOM을 만드는 코드는 실행되지 않는다. 두 번째는 함수가 있는 state 때문에 실패할 수 있다. 실제 네트워크 요청·목록 반영·오류 안내는 별도 코드가 연결해야 한다. “주소 표시줄이 바뀌었다”와 “화면이 새 데이터로 바뀌었다”를 따로 관찰한다.

---

## 15. 실제 정적 JSON으로 주소·요청·화면을 연결하기

### 실행 조건과 데이터 파일

아래 두 파일을 같은 폴더에 저장하고 로컬 HTTP 서버로 제공한다. 예를 들어 그 폴더에서 `python3 -m http.server 8000`을 실행한 뒤 index.html을 연다. 이 명령은 실험 파일 제공을 위한 것이며 서버 API 구현을 학습하는 예제가 아니다.

```json
[
  { "id": "html", "title": "HTML Parsing", "tag": "browser" },
  { "id": "dom", "title": "DOM Manipulation", "tag": "browser" },
  { "id": "react", "title": "React Basics", "tag": "ui" }
]
```

위 내용을 notes.json으로 저장한다. 인덱스 파일은 다음과 같다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>주소 기반 검색</title></head>
<body>
  <h1>노트 검색</h1>
  <form id="search">
    <label for="query">검색어</label>
    <input id="query" name="q">
    <button type="submit">검색</button>
  </form>
  <p id="status" role="status"></p>
  <ul id="results"></ul>
  <script>
    const form = document.querySelector('#search');
    const input = document.querySelector('#query');
    const results = document.querySelector('#results');
    const status = document.querySelector('#status');
    let generation = 0;
    let current = null;

    async function renderAddress() {
      const version = ++generation;
      current?.abort();
      const controller = new AbortController();
      current = controller;
      const query = new URL(location.href).searchParams.get('q') ?? '';
      input.value = query;
      results.replaceChildren();
      status.textContent = '불러오는 중';
      try {
        const response = await fetch('./notes.json', { signal: controller.signal });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const rows = await response.json();
        if (!Array.isArray(rows)) throw new Error('목록 형식이 아닙니다.');
        if (version !== generation) return;
        const matches = rows.filter((row) =>
          row !== null && typeof row === 'object' &&
          typeof row.title === 'string' && row.title.toLowerCase().includes(query.toLowerCase()));
        for (const row of matches) {
          const item = document.createElement('li');
          item.textContent = row.title;
          results.append(item);
        }
        status.textContent = matches.length ? `${matches.length}개 결과` : '결과가 없습니다.';
      } catch (error) {
        if (version !== generation || controller.signal.aborted) return;
        status.textContent = '목록을 불러오지 못했습니다. 다시 검색하세요.';
        console.error(error);
      } finally {
        if (current === controller) current = null;
      }
    }
    form.addEventListener('submit', (event) => {
      event.preventDefault();
      const url = new URL(location.href);
      url.searchParams.set('q', input.value);
      history.pushState(null, '', url);
      renderAddress();
    });
    window.addEventListener('popstate', renderAddress);
    renderAddress();
  </script>
</body>
</html>
```

### 단계별 상태 변화

초기 진입은 현재 주소를 읽고 입력값을 복원한 뒤 요청한다. 제출은 URL을 만든 뒤 기록에 추가하고 렌더링 함수를 직접 호출한다. 뒤로 가기는 popstate를 통해 같은 함수를 호출한다. 데이터를 읽은 뒤에는 형식을 검사하고, 현재 요청 세대가 맞을 때만 DOM을 바꾼다. 응답이 빈 목록인지·읽기 실패인지도 다른 안내로 표시한다.

current는 이전 요청의 중단을, generation은 오래된 결과의 무효화를 담당한다. 중단만으로 이미 완료된 모든 비동기 후속 작업의 유효성 판정이 자동으로 해결되는 것은 아니다. finally에서도 자신이 현재 컨트롤러인지 확인하여 오래된 작업이 새 요청의 상태를 지우지 않게 했다.

### 성공뿐 아니라 실패를 재현하기

notes.json의 파일 이름을 바꾸면 HTTP 실패를, 내용을 잘못된 JSON으로 바꾸면 해석 실패를, 빈 배열로 바꾸면 빈 결과를 볼 수 있다. Network의 지연 조건을 바꾸고 검색·뒤로 가기를 빠르게 반복하여 최신 주소와 결과가 맞는지 확인한다. 이 예제는 매번 목록을 요청하므로 실제 제품에서는 캐시·데이터 수명 요구를 별도로 설계해야 한다.

---

## 16. Location·History·브라우저 정책의 적용 범위

### 문서 이동과 현재 문서 기록을 나눈다

location.assign·replace는 이동을 요청한다. fragment 변화처럼 같은 문서 안에서 처리되는 이동도 있다. pushState·replaceState는 현재 문서와 연결된 기록을 갱신하고 자체적으로 새 문서를 내려받지 않는다. 따라서 임의의 새 경로를 pushState로 표시한 뒤 새로고침하면 서버가 그 경로의 문서를 제공할 수 있는지 별도로 확인해야 한다.

### popstate는 모든 주소 변경의 통지가 아니다

pushState나 replaceState 호출 자체는 popstate를 발생시키는 일반적인 트리거가 아니다. 기록을 탐색할 때의 상태 복원과 앱이 직접 주소를 갱신할 때의 렌더링을 각각 연결한다. fragment 변화와 hashchange도 같은 API의 동의어가 아니므로 [popstate 문서](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event)의 발생 조건을 확인한다.

### CORS·자격 증명·서버 변경은 별도의 계약이다

다른 origin의 요청에서는 응답 읽기 허용, 사전 요청 조건, 쿠키 정책이 영향을 준다. credentials 설정은 쿠키를 무조건 강제로 보내는 명령이 아니며 no-cors는 응답 읽기 제한의 우회 수단이 아니다. 세부 네트워크·인증 구현은 후속 Network / Security에서 확장하고, 현재는 요청·응답·읽기 허용의 경계를 구별한다.

### 취소는 서버 작업의 롤백이 아니다

abort는 클라이언트의 관련 작업을 중단하는 도구다. 서버가 이미 저장한 주문이나 게시글을 되돌렸다는 증거가 아니다. 특히 쓰기 요청을 자동 재시도할 때는 중복 실행 요구와 서버 계약을 함께 설계해야 한다. 읽기 화면에서 오래된 결과를 버리는 문제와 서버의 업무 무결성은 서로 다른 책임이다.

---

## 17. 실무 적용과 디버깅

Network에서 요청 주소·상태·본문·CORS를, 주소 표시줄에서 기록 변화를, 콘솔에서 JSON 해석 실패를 확인한다. 로딩·실패·빈 결과를 구별해서 안내하고, 로딩 중에도 입력 포커스와 다시 시도할 경로를 보존한다. 검색어가 바뀔 때마다 보내는 요청 비용과 이전 결과의 경쟁도 확인한다.

\[미리보기: 라우터와 데이터 페칭 도구\]

라우터와 데이터 페칭 라이브러리는 주소·화면·비동기 상태를 연결하는 반복 작업을 추상화한다. 캐시·재검증과 프레임워크별 구현은 후속 커리큘럼에서 다룬다.

---

## 18. 핵심 정리

<callout icon="✅" color="green_bg">
URL은 주소를 구성하고, fetch는 응답을 얻고, Location은 이동을 요청하고, History는 기록을 관리한다. 이들을 연결하는 화면 갱신과 오류·경쟁 처리는 애플리케이션의 책임이다.
</callout>

---

## 출처

- [WHATWG — URL](https://url.spec.whatwg.org/)
- [WHATWG — Fetch](https://fetch.spec.whatwg.org/)
- [WHATWG — Location / History](https://html.spec.whatwg.org/multipage/nav-history-apis.html)
- [MDN — Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Fetch](https://fetch.spec.whatwg.org/): Request·Response·본문·자격 증명·CORS의 플랫폼 계약.
- [MDN — Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch): 상태 확인·본문 읽기·중단의 사용 조건.
- [MDN — Response.bodyUsed](https://developer.mozilla.org/en-US/docs/Web/API/Response/bodyUsed): 소비한 본문을 반복 조회할 수 없는 이유.
- [WHATWG — URLSearchParams](https://url.spec.whatwg.org/#urlsearchparams): 쿼리의 인코딩·반복 항목 처리.
- [MDN — History.pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState): 상태의 직렬화·주소·예외 조건.
- [MDN — popstate](https://developer.mozilla.org/en-US/docs/Web/API/Window/popstate_event): 기록 이동과 직접 갱신의 차이.

---

## 19. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 404가 왔는데 catch가 실행되지 않는다. 네트워크·HTTP·본문 해석 중 어느 단계를 구별해야 하는가?
2. pushState를 호출했지만 검색 목록이 바뀌지 않는다. 누가 무엇을 추가로 해야 하는가?
3. 빠르게 검색어를 바꿨더니 이전 검색 결과가 마지막에 표시된다. 요청 순서와 완료 순서를 어떻게 다루겠는가?
4. 클라이언트에서는 정상인 주소를 새로고침하면 서버 404가 난다. History API가 해결하지 않는 부분은 무엇인가?
