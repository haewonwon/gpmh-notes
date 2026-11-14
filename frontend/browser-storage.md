---
track: B
notion: https://app.notion.com/p/415178c4afac8102ae68c2244f637b27
---

# Cookie / LocalStorage / SessionStorage / IndexedDB


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · Cookie / LocalStorage / SessionStorage / IndexedDB

전송되는 쿠키, 문자열 저장소, 구조화된 데이터베이스의 차이를 수명·범위·비용과 연결한다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 쿠키 속성을 범위·수명·전송·스크립트 접근 조건으로 구별할 수 있다.
- localStorage와 sessionStorage의 공유 범위·현재 값·변경 통지를 설명할 수 있다.
- IndexedDB의 store·키·index·버전·transaction 완료를 구별할 수 있다.
- 저장 실패·버전 변경·동시 수정·삭제·복구 요구를 설계할 수 있다.
- 실제 요청 헤더와 저장소 패널에서 보안 조건·데이터·비용을 검증할 수 있다.

---

## 2. 이 주제를 왜 배우는가

브라우저 저장소는 모두 “새로고침해도 값을 남긴다”는 하나의 기능이 아니다. 로그인 요청에 함께 보내는 데이터와 이 탭의 임시 초안, 큰 오프라인 자료는 요구가 다르다. 무엇을 보존하고 누구와 공유해야 하는지를 먼저 정해야 적절한 수단을 선택할 수 있다.

---

## 3. 관련 계층과 역할 구분

브라우저는 저장 영역과 접근 API를 제공하고 origin·탐색 컨텍스트·사이트 정책 등의 경계를 적용한다. 쿠키는 서버 응답과 요청 헤더에도 연결되지만 Web Storage·IndexedDB는 데이터가 자동으로 서버에 첨부되는 저장소가 아니다.

애플리케이션은 데이터 형식·만료·동기화·오류·복구를 설계한다. React state 같은 메모리 상태와 브라우저 저장소는 별개이므로 저장한 뒤 화면 갱신 경로도 연결해야 한다.

---

## 4. 저장 수단을 비교한다

| 수단 | 주요 범위·수명 | 데이터·접근 방식 | HTTP에 자동 첨부 |
| --- | --- | --- | --- |
| Cookie | 도메인·경로·만료·보안·사이트 정책의 영향 | 작은 이름·값, 헤더 또는 관련 API | 조건에 맞는 요청에 가능 |
| localStorage | 일반적으로 origin별, 세션 이후에도 유지 가능 | 문자열 키·값, 동기 | 안 됨 |
| sessionStorage | origin과 최상위 탐색 컨텍스트별 페이지 세션 | 문자열 키·값, 동기 | 안 됨 |
| IndexedDB | 일반적으로 origin별, 영속 저장 가능 | 구조화된 데이터·인덱스·트랜잭션, 비동기 | 안 됨 |

저장소는 브라우저의 삭제·비공개 모드·용량·파티셔닝 정책에 영향을 받는다. “영속”은 사용자가 지우거나 브라우저가 회수할 수 없다는 뜻이 아니다. 특정 용량을 모든 브라우저에 동일하게 보장하지 않는다.

---

## 5. 쿠키는 요청과 연결되는 저장이다

서버는 Set-Cookie 응답 헤더로 쿠키를 설정할 수 있고, 브라우저는 범위와 정책이 맞는 요청의 Cookie 헤더에 값을 첨부할 수 있다. 모든 쿠키가 모든 요청에 붙지는 않는다. Domain·Path·Secure·SameSite·만료·자격 증명 정책 등이 관여한다. 쿠키는 단순히 origin 하나로만 범위가 정해지는 Web Storage와 다르다.

`HttpOnly`는 JavaScript의 document.cookie 접근을 제한한다. `Secure`는 보안 연결 조건과 관련되고, `SameSite`는 사이트 간 요청에서 전송 조건을 제한한다. SameSite의 사이트와 same-origin의 origin은 동일한 개념이 아니다. 만료·Max-Age가 없는 세션 쿠키도 브라우저의 세션 복원 기능 때문에 기대보다 오래 복원될 수 있다. [MDN 쿠키 안내](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)를 기준으로 속성의 목적을 구별한다.

쿠키는 자동 전송 특성이 있고 JavaScript 저장소는 스크립트 접근 특성이 있으므로 인증 데이터의 저장은 각각 CSRF·XSS 등 서로 다른 요구와 연결된다. 이 페이지에서는 이를 선택 기준으로만 확인하며 인증 구현은 후속 보안 학습 범위다.

---

## 6. Cookie의 속성을 전송 조건으로 읽기

### 서버 설정과 JavaScript 설정의 차이

```text
Set-Cookie: session=opaque-value; Path=/; Secure; HttpOnly; SameSite=Lax
```

이것은 서버 응답 헤더의 예다. JavaScript로 HttpOnly 쿠키를 읽거나 document.cookie로 같은 보호 속성을 설정하는 방식은 사용할 수 없다. 일반 쿠키는 document.cookie로 설정할 수 있지만 그것은 모든 쿠키를 한 번에 교체하는 일반 객체 저장과 다르다.

쿠키 값을 인증에 사용할 때 서버가 불투명한 식별자를 어떻게 검증·만료·폐기하는지는 별도 서버 설계다. 브라우저에 쿠키가 있다는 사실만으로 사용자가 인증된다는 뜻은 아니다.

### 범위·수명·보호 조건은 서로 다른 축이다

| 속성 | 다루는 조건 | 보장하지 않는 것 |
| --- | --- | --- |
| Domain | 어떤 호스트 범위에 전송 가능한가 | JavaScript 접근의 완전한 보안 격리 |
| Path | 어떤 요청 경로에 첨부하는가 | 디렉터리별 스크립트 보안 경계 |
| Expires·Max-Age | 언제 만료하는가 | 사용자의 삭제·세션 복원 정책 통제 |
| Secure | 보안 연결 조건 | JavaScript 읽기 차단 |
| HttpOnly | 스크립트 API 접근 제한 | 모든 XSS 영향이나 자동 전송 위험 제거 |
| SameSite | 사이트 간 요청에서 전송 조건 | 모든 인증·CSRF 요구의 단독 해결 |

Domain을 생략한 host-only 쿠키와 상위 도메인 범위에 설정한 쿠키는 전송 범위가 다르다. Expires와 Max-Age를 함께 지정할 때의 우선순위, 삭제 시 동일 범위 조건 등도 확인한다. 쿠키를 삭제하려면 이름만 아니라 경로·도메인 등 원래 범위와 맞는 설정이 중요하다.

브라우저는 제3자 쿠키와 파티셔닝에 추가 정책을 적용할 수 있다. 따라서 같은 코드가 탑레벨 문서에서는 동작하지만 임베드 환경에서는 제한될 수 있다. 현재 지원 대상의 정책과 실제 요청 헤더를 함께 확인한다.

---

## 7. localStorage와 sessionStorage의 범위·변경 이벤트

### 문자열로 저장한다는 뜻

```javascript
localStorage.setItem('theme', 'dark');
localStorage.setItem('page-size', String(20));
const size = Number(localStorage.getItem('page-size'));
console.log(size);
```

getItem은 저장값의 문자열 또는 null을 반환한다. 숫자·불리언이 필요하면 변환과 유효성 검사를 명시한다. JSON을 사용하면 여러 필드를 저장할 수 있지만 파싱 성공이 애플리케이션 스키마의 유효성까지 뜻하지는 않는다.

예를 들어 `JSON.parse` 결과가 객체인지, version이 맞는지, text가 문자열인지 확인해야 한다. 저장소를 사용자가 개발자 도구로 바꾸거나 이전 버전 코드가 다른 형식으로 남길 수 있기 때문이다.

### 같은 origin과 같은 탭은 서로 다른 조건이다

localStorage는 일반적으로 같은 origin 문서들이 공유하는 저장 영역이다. sessionStorage는 origin에 더해 최상위 탐색 컨텍스트의 페이지 세션을 구별한다. 같은 탭 안의 같은 origin 문서가 연결될 수 있지만 다른 탭의 현재 값과 계속 공유하는 모델은 아니다.

새로고침은 페이지 세션 종료와 같지 않다. 브라우저 종료·복원, 복제 탭, 비공개 모드는 실제 수명이 기대와 다를 수 있으므로 요구되는 환경에서 시험한다. file URL의 저장소 동작을 배포된 HTTP origin의 보장처럼 사용하지 않는다.

### storage 이벤트는 저장한 문서 자체의 상태 갱신을 대신하지 않는다

```javascript
window.addEventListener('storage', (event) => {
  if (event.storageArea === localStorage && event.key === 'theme') {
    console.log('다른 문서에서 바뀐 테마:', event.newValue);
  }
});
```

이 리스너는 조건에 맞는 다른 문서의 변경 통지를 다룬다. 저장한 문서의 UI는 자신의 상태 갱신 경로로 반영한다. 여러 탭이 동시에 읽고 계산한 뒤 쓰는 작업은 데이터베이스 트랜잭션과 같은 원자성을 제공하지 않으므로 충돌 정책이 필요할 수 있다.

---

## 8. 작은 UI 설정은 Web Storage로 저장한다

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>초안 저장</title></head>
<body>
  <h1>학습 초안</h1>
  <label for="draft">메모</label>
  <textarea id="draft"></textarea>
  <button type="button" id="save">이 탭에 저장</button>
  <p id="status" role="status"></p>
  <script>
    const input = document.querySelector('#draft');
    const status = document.querySelector('#status');
    const key = 'learning-draft:v1';
    try {
      const raw = sessionStorage.getItem(key);
      if (raw !== null) {
        const data = JSON.parse(raw);
        if (typeof data.text === 'string') input.value = data.text;
      }
    } catch (error) {
      status.textContent = '저장된 초안을 읽지 못했습니다.';
      console.error(error);
    }
    document.querySelector('#save').addEventListener('click', () => {
      try {
        sessionStorage.setItem(key, JSON.stringify({ text: input.value }));
        status.textContent = '이 탭의 초안을 저장했습니다.';
      } catch (error) {
        status.textContent = '저장하지 못했습니다. 내용을 별도로 복사하세요.';
        console.error(error);
      }
    });
  </script>
</body>
</html>
```

로컬 HTTP 서버로 제공하면 origin 기준으로 실험하기 쉽다. 저장 버튼을 누르고 같은 탭을 새로고침하면 초안이 복원된다. sessionStorage는 페이지 세션을 기준으로 하므로 같은 origin의 모든 탭이 하나의 현재 값을 공유하는 저장소는 아니다. opener가 있는 새 창은 초기 복사본을 받을 수 있으나 이후 같은 저장 영역으로 동기화되는 것은 아니다.

localStorage로 바꾸면 일반적으로 탭 간 같은 origin의 저장 영역을 공유하며 다시 방문할 때도 읽을 수 있다. 다른 문서에서 변경된 내용을 반영하려면 storage 이벤트 등을 처리해야 한다. 저장한 문서 자체에 동일한 storage 이벤트가 발생한다고 가정하지 않는다. [Web Storage 표준](https://html.spec.whatwg.org/multipage/webstorage.html)은 범위와 이벤트 규칙을 구별한다.

---

## 9. 데이터가 커지면 IndexedDB를 검토한다

IndexedDB는 데이터베이스 안의 object store에 키와 값을 저장하며 index로 조회 경로를 추가할 수 있다. 관련 변경은 transaction으로 묶는다. Web Storage처럼 문자열 하나를 매번 통째로 읽고 직렬화하는 대신 구조화된 레코드를 비동기로 다룰 수 있다.

예를 들어 많은 학습 노트를 오프라인에서 검색하려면 `notes` 저장소에 id를 키로 두고 updatedAt 같은 인덱스를 고려할 수 있다. 연결 → 버전 변경 시 저장소 생성·이전 → 트랜잭션 → 읽기·쓰기 → 완료·실패 처리의 흐름으로 이해한다. 하나의 요청 성공이 쓰기 트랜잭션의 최종 커밋 완료와 같지는 않다. 개별 요청 결과와 transaction의 complete·abort를 구별한다.

IndexedDB가 비동기라고 해서 대량 데이터 직렬화·복제·화면 처리가 무료가 되지는 않는다. 데이터 양과 조회 조건을 기준으로 선택하고, 작은 토글 설정 하나에 데이터베이스 구조를 과도하게 도입하지 않는다.

---

## 10. IndexedDB의 기본 구조와 트랜잭션

### 데이터베이스·object store·키·인덱스

데이터베이스는 여러 object store를 담는다. store는 레코드를 저장하고 키는 레코드를 구별한다. index는 title·updatedAt 같은 다른 필드로 접근하는 경로를 제공한다. 모든 데이터에 무조건 인덱스를 추가하면 쓰기·저장 비용도 늘어나므로 필요한 조회를 먼저 정한다.

```javascript
const request = indexedDB.open('study-notes', 1);
request.onupgradeneeded = () => {
  const db = request.result;
  const store = db.createObjectStore('notes', { keyPath: 'id' });
  store.createIndex('by-updated-at', 'updatedAt');
};
request.onerror = () => console.error(request.error);
request.onblocked = () => console.log('다른 탭의 연결 상태를 확인하세요.');
request.onsuccess = () => {
  const db = request.result;
  db.onversionchange = () => db.close();
  const transaction = db.transaction('notes', 'readwrite');
  const store = transaction.objectStore('notes');
  store.put({ id: 'html', title: 'HTML', updatedAt: Date.now() });
  transaction.oncomplete = () => {
    console.log('트랜잭션 완료');
    db.close();
  };
  transaction.onabort = () => {
    console.error(transaction.error);
    db.close();
  };
};
```

브라우저의 HTTP 문서 콘솔에서 실행할 수 있다. 처음 연 버전에서는 upgrade 이벤트에서 저장소와 인덱스를 만든다. 이미 같은 이름·버전의 다른 스키마가 있다면 깨끗한 실험 환경 또는 의도한 버전 마이그레이션을 사용한다. 버전을 올릴 때 열린 다른 탭의 연결로 업그레이드가 막힐 수 있어 versionchange·blocked 처리를 고려한다.

### 개별 요청과 커밋의 완료를 구별한다

put 요청이 성공한 뒤에도 같은 트랜잭션의 다른 요청이 실패하여 전체가 abort될 수 있다. 따라서 사용자에게 최종 저장 완료를 알릴 기준은 요구에 맞는 트랜잭션 완료다. 트랜잭션은 적용 대상 store와 읽기·쓰기 모드를 정하며 관련 변경을 묶는다.

IndexedDB 트랜잭션을 임의의 네트워크 await 동안 계속 열려 있는 범용 작업 문맥으로 취급하지 않는다. 활성 기간과 자동 커밋 규칙을 확인하고 필요한 데이터를 준비한 뒤 DB 작업을 요청한다. 데이터베이스 트랜잭션의 세부 이론은 별도 DB 학습으로 확장한다.

---

## 11. 저장 수단 선택을 요구사항으로 설명하기

작은 테마 설정은 문자열 저장으로 충분할 수 있고, 한 탭에서 진행하는 작성 단계는 sessionStorage의 범위를 검토한다. 큰 오프라인 레코드와 인덱스 조회는 IndexedDB가 적합할 수 있다. 서버 요청에 조건부로 함께 전달되어야 하는 식별 정보는 쿠키의 전송 모델과 관련된다.

데이터를 남기는 결정에는 삭제·내보내기·만료·백업·공유의 요구가 따라온다. 장기 보존해야 할 사용자 자료를 브라우저 로컬 저장만으로 보장하지 않고 복구 경로를 설계한다. 저장 실패가 발생해도 현재 편집 내용을 화면에서 잃지 않게 하는 것이 기능 요구사항이다.

---

## 12. Cookie의 범위는 origin과 같지 않다

### host·domain·path·site를 구별한다

일반적인 웹 origin은 스킴·호스트·포트를 묶지만 Cookie의 범위는 호스트·Domain·Path와 여러 전송 정책으로 정해진다. 따라서 Web Storage의 origin 격리와 쿠키의 전달 범위를 같은 표로 단순 치환하면 틀릴 수 있다. [HTTP 쿠키 표준](https://httpwg.org/specs/rfc6265.html)은 쿠키의 저장·선택·헤더 전달을 별도로 설명한다.

Domain을 생략한 host-only 쿠키는 설정한 호스트에 제한되고, 허용되는 Domain을 명시하면 그 도메인과 관련 하위 호스트에 전송될 수 있다. 임의의 다른 도메인 쿠키를 마음대로 설정할 수 있는 것은 아니다. Path는 요청 경로 선택 조건이지만 같은 출처의 악성 스크립트를 격리하는 디렉터리 보안 장벽은 아니다.

### 수명과 서버 인증 상태는 별개다

Expires·Max-Age는 브라우저가 값을 얼마나 유지할지와 관계한다. 서버가 해당 세션 식별자를 유효하게 받아들일지는 서버의 만료·폐기 정책이다. 브라우저 쿠키만 남아 있는데 서버에서는 이미 만료되었을 수도 있고, 클라이언트 삭제가 서버의 모든 세션 기록을 자동 폐기하는 것도 아니다. 이 페이지에서는 경계를 구별하고 실제 인증 설계는 후속 학습에서 다룬다.

### Secure·HttpOnly·SameSite가 각각 보호하는 것

Secure는 전송 연결 조건, HttpOnly는 JavaScript 접근 제한, SameSite는 사이트 간 전송 상황과 관련된다. 하나를 설정했다고 나머지 보호가 자동으로 생기지 않는다. HttpOnly 쿠키라도 브라우저가 조건에 맞는 요청에 첨부할 수 있어, XSS가 실행된 페이지의 모든 영향을 사라지게 만드는 속성은 아니다. SameSite의 세부·현행 제한은 [Set-Cookie 문서](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie)에서 속성별로 확인한다.

---

## 13. 쿠키를 설정·삭제하며 범위를 관찰하기

### 실험 1. 로컬 실험 문서에서 작은 쿠키 다루기

로컬 HTTP 서버로 제공하는 실험 문서의 콘솔에서 실행한다. 예제 이름의 쿠키만 만들고 삭제한다.

```javascript
{
  document.cookie = 'study-lab-theme=dark; Path=/; SameSite=Lax';
  const entry = document.cookie.split(';').map((item) => item.trim())
    .find((item) => item.startsWith('study-lab-theme='));
  console.log(entry);
  document.cookie = 'study-lab-theme=; Path=/; Max-Age=0; SameSite=Lax';
  console.log(document.cookie.split(';').map((item) => item.trim())
    .some((item) => item.startsWith('study-lab-theme=')));
}
```

허용된 일반적인 문서 환경에서는 설정한 항목을 읽고, 같은 범위로 만료시킨 뒤에는 해당 실험 항목이 사라진다. 정책이나 환경으로 쿠키가 제한된 경우에는 같은 관찰이 나오지 않을 수 있다. document.cookie에는 JavaScript에서 읽을 수 없는 HttpOnly 항목이 보이지 않는다. 이 문자열을 서버의 모든 Set-Cookie 속성을 보여주는 객체라고 생각하지 않는다.

### 삭제할 때 범위를 맞추는 이유

같은 이름이라도 경로·도메인 범위가 다른 쿠키가 있을 수 있다. Path가 다른 항목을 다른 설정으로 만료시키면 의도한 쿠키가 남아 “삭제가 안 된다”고 보일 수 있다. Application과 실제 요청 헤더에서 이름뿐 아니라 범위를 확인한다. 전체 쿠키를 무작정 지우는 코드를 기본 디버깅 방법으로 삼지 않는다.

### 실제 전송은 Network에서 확인한다

document.cookie에서 읽었다는 사실은 특정 요청에 첨부되었다는 증거가 아니다. 요청 주소·Secure·SameSite·credentials·브라우저 정책을 포함한 조건을 확인하고 Network의 Cookie·Set-Cookie를 본다. JavaScript가 일반 fetch에서 Set-Cookie 헤더를 자유롭게 읽을 수 있는 일반 응답 헤더라고 가정하지 않는다.

---

## 14. 문자열 저장의 형식·검증·버전을 설계하기

### 저장 성공과 유효한 데이터는 다르다

Web Storage는 문자열을 저장한다. JSON.parse가 성공했어도 기대한 객체·필드·버전이라는 보장은 없다. 이전 버전의 앱, 사용자의 개발자 도구 수정, 부분적인 기능 변경이 다른 형식을 남길 수 있다. 데이터의 유효성을 검사한 뒤 업무 값으로 사용하고, 읽기 실패와 저장 실패를 각각 처리한다.

### 실험 2. 버전이 있는 초안의 읽기와 쓰기

```javascript
const draftKey = 'study-lab:draft:v1';

function loadDraft() {
  try {
    const raw = localStorage.getItem(draftKey);
    if (raw === null) return { text: '', reason: 'missing' };
    const value = JSON.parse(raw);
    if (value === null || typeof value !== 'object' || value.version !== 1
        || typeof value.text !== 'string') {
      return { text: '', reason: 'invalid' };
    }
    return { text: value.text, reason: 'restored' };
  } catch (error) {
    return { text: '', reason: 'unavailable', error };
  }
}

function saveDraft(text) {
  try {
    localStorage.setItem(draftKey, JSON.stringify({ version: 1, text }));
    return { saved: true };
  } catch (error) {
    return { saved: false, error };
  }
}
```

두 함수는 HTTP 실험 문서에서 호출할 수 있다. 저장된 항목이 없는 경우와 형식이 잘못된 경우, 저장소 접근이 불가능한 경우를 reason으로 나누었다. UI는 복원 실패를 알리되 현재 편집 중인 내용을 파괴하지 않는 정책을 정할 수 있다. 실패를 빈 초안으로만 뭉개면 실제로 사용자가 작성한 자료가 있었는지 판단하기 어렵다.

### 직접 실패 조건을 바꿔 보기

```javascript
console.log(saveDraft('HTML 학습 중'));
console.log(loadDraft());
localStorage.setItem(draftKey, '{');
console.log(loadDraft().reason);
localStorage.setItem(draftKey, JSON.stringify({ version: 9, text: '다른 버전' }));
console.log(loadDraft().reason);
localStorage.removeItem(draftKey);
```

잘못된 JSON은 파싱 경로의 실패, 다른 버전은 앱 형식 검사에서의 invalid다. 이런 경우를 모두 정상적인 빈 상태로 표시할지, 복구·내보내기 안내를 제공할지는 자료의 중요성에 따라 결정한다. 예제는 테스트 키만 제거하며 같은 origin의 다른 앱 자료 전체를 clear하지 않는다.

---

## 15. 탭 사이 공유와 페이지 세션을 재현하기

### 실험 3. 두 탭으로 storage 이벤트 관찰하기

같은 HTTP 문서 두 탭을 연다. 첫 탭의 콘솔에 다음을 등록한다.

```javascript
window.addEventListener('storage', (event) => {
  if (event.key !== 'study-lab:theme') return;
  console.log({ old: event.oldValue, next: event.newValue, source: event.url });
});
```

둘째 탭에서 다음을 실행한다.

```javascript
localStorage.setItem('study-lab:theme', 'dark');
localStorage.setItem('study-lab:theme', 'light');
localStorage.removeItem('study-lab:theme');
```

첫 탭은 다른 문서의 변경을 관찰할 수 있다. 저장한 둘째 탭 자체의 리스너가 같은 변경을 받아 자기 UI를 갱신해 주는 모델은 아니다. 삭제 시 newValue는 null일 수 있다. [storage 이벤트의 전달 범위](https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event)를 UI 동기화 설계에 적용한다.

### sessionStorage는 다른 탭의 현재 값을 공유하지 않는다

같은 origin에서도 최상위 탐색 컨텍스트의 페이지 세션이 구별된다. opener가 있는 새 창이 초기 복사본을 받을 수 있다는 사실과 이후 같은 저장 영역을 계속 공유한다는 주장은 다르다. 새로고침·새 탭·복제·종료·세션 복원은 실제 요구 환경에서 각각 확인한다.

### 이벤트가 있어도 원자적 읽기·수정·쓰기는 아니다

두 탭이 같은 count를 읽고 각자 1을 더해 쓰면 두 증가 중 하나가 사라질 수 있다. storage 이벤트는 변경 통지이지 다중 문서의 트랜잭션 잠금이 아니다. 동시 편집이 필요한 데이터라면 충돌 해결·단일 소유자·서버 동기화·데이터베이스 트랜잭션 등 요구에 맞는 모델을 정해야 한다.

---

## 16. IndexedDB를 레코드와 트랜잭션으로 읽기

### store·keyPath·index의 관계

store는 레코드 집합, key는 그 안에서 레코드를 구별하는 값이다. keyPath는 저장 객체의 어느 필드에서 기본 키를 읽을지 지정할 수 있다. index는 다른 필드로 찾는 별도 접근 경로다. 저장된 객체가 어떤 형태인지와 어떤 조회가 필요한지를 먼저 정한 뒤 키·인덱스를 고른다.

put은 동일 키가 있으면 갱신할 수 있고 add는 중복 키가 허용되지 않는 추가를 표현한다. 두 메서드를 이름이 비슷한 저장 동작으로만 취급하면 중복 레코드 처리와 오류 조건이 달라진다. [Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB)를 생성·업그레이드·요청·트랜잭션의 순서로 읽는다.

### 실험 4. 요청 success 뒤에도 전체 저장은 abort될 수 있다

다음은 새로운 실험 데이터베이스에서 중복 키로 실패를 재현하고 끝에 정리하는 코드다. 로컬 HTTP 문서에서 실행한다.

```javascript
function inspectTransactionAbort() {
  const name = `study-lab-${Date.now()}-${Math.random()}`;
  const opening = indexedDB.open(name, 1);
  opening.onupgradeneeded = () => {
    opening.result.createObjectStore('notes', { keyPath: 'id' });
  };
  opening.onerror = () => console.error(opening.error);
  opening.onsuccess = () => {
    const db = opening.result;
    const transaction = db.transaction('notes', 'readwrite');
    const store = transaction.objectStore('notes');
    const first = store.add({ id: 'same', title: '첫 항목' });
    const second = store.add({ id: 'same', title: '중복 항목' });
    first.onsuccess = () => console.log('첫 요청 success');
    second.onerror = () => console.log('둘째 요청:', second.error.name);
    transaction.oncomplete = () => console.log('트랜잭션 complete');
    transaction.onabort = () => {
      console.log('트랜잭션 abort');
      const reading = db.transaction('notes', 'readonly');
      const all = reading.objectStore('notes').getAll();
      all.onsuccess = () => console.log('남은 레코드:', all.result.length);
      reading.oncomplete = () => {
        db.close();
        const deletion = indexedDB.deleteDatabase(name);
        deletion.onsuccess = () => console.log('실험 DB 정리');
      };
    };
  };
}
inspectTransactionAbort();
```

첫 요청은 success를 알릴 수 있지만 둘째 add의 중복 키 오류가 기본 처리로 트랜잭션을 abort시킨다. 새 실험 DB의 레코드는 0개로 확인된다. 따라서 첫 요청의 성공 로그만 보고 사용자에게 최종 저장 완료를 알리면 틀릴 수 있다. 이 예제는 요청 오류의 기본 abort 동작을 사용하며, 오류 이벤트를 취소하여 다르게 처리하는 설계까지 같은 결과라고 일반화하지 않는다.

### 트랜잭션의 활성 기간

트랜잭션은 임의의 네트워크 작업을 await하는 동안 무한히 열어두는 범용 실행 문맥이 아니다. 요청을 넣을 수 있는 활성 상태와 자동 커밋 조건을 따른다. 필요한 외부 데이터를 준비한 뒤 관련 DB 요청을 묶고, 최종 complete·abort를 확인한다. [IDBTransaction 문서](https://developer.mozilla.org/en-US/docs/Web/API/IDBTransaction)가 수명과 자동 완료 조건을 설명한다.

### 스키마 변경과 열린 연결

버전을 올려 저장소·인덱스를 변경할 때 다른 탭의 기존 연결이 업그레이드를 막을 수 있다. 기존 연결의 versionchange와 새 요청의 blocked를 처리할 정책이 필요하다. 사용자가 편집 중인 화면을 무조건 강제 새로고침하기보다 저장·종료·재시도의 안내를 설계한다.

---

## 17. 저장 수단 선택의 마지막 조건: 삭제·복구·보안·비용

### 영속성은 절대 보존이 아니다

브라우저 데이터 삭제, 비공개 모드, 용량 회수, 정책 변경으로 데이터가 사라지거나 접근이 제한될 수 있다. 저장소마다 정확한 한도를 고정된 숫자로 외우기보다 대상 환경과 [저장량·회수 정책 문서](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)를 확인한다. 사용자의 중요한 자료에는 별도의 백업·내보내기·동기화가 필요할 수 있다.

### 빠른 API 선택보다 저장 빈도와 데이터 양

작은 설정 하나에는 단순한 문자열 저장이 충분할 수 있다. 매 입력마다 거대한 JSON 전체를 직렬화해 동기 저장하면 반응성이 나빠질 수 있다. IndexedDB도 복제·인덱스·큰 결과 배열의 비용이 있어 무조건 무료가 아니다. 저장 시간·입력 지연·읽는 데이터 양을 같은 작업으로 측정한다.

### 민감 데이터는 저장 위치 이름만으로 보호되지 않는다

localStorage·IndexedDB는 동일한 origin에서 실행되는 허용된 코드가 접근할 수 있다. 쿠키도 속성별로 서로 다른 보호를 제공한다. 토큰·개인정보의 저장은 인증·XSS·CSRF·만료·폐기 요구와 연결되며, 이 페이지의 저장 API 비교만으로 완성하지 않는다. 보안 구현은 해당 후속 학습에서 확장한다.

---

## 18. 실패·보안·수명을 설계한다

저장 접근은 정책 또는 용량 때문에 예외가 날 수 있고 JSON은 과거 버전·잘못된 값 때문에 파싱이나 검증에 실패할 수 있다. 저장하지 못해도 사용자가 작업을 잃지 않도록 안내하고, 데이터 버전과 초기화·삭제 경로를 둔다. 브라우저 저장소를 중요한 자료의 유일한 백업으로 삼지 않는다.

localStorage에 넣었다고 데이터가 암호화되거나 XSS로부터 보호되는 것은 아니다. 동일 origin의 실행 가능한 스크립트가 접근할 수 있으므로 비밀 정보는 요구에 맞는 별도 보안 설계가 필요하다. 쿠키의 Path도 스크립트 보안 격리 수단으로 간주하지 않는다.

Application/Storage 패널에서 저장 범위·값·쿠키 속성을 확인하고, 새로고침·새 탭·종료·삭제·저장 실패를 각각 실험한다. 동기 저장소를 입력마다 큰 데이터로 반복 갱신하면 메인 스레드 비용이 생기므로 크기와 빈도를 측정한다.

---

## 19. 핵심 정리

<callout icon="✅" color="green_bg">
쿠키는 요청 전송 조건, Web Storage는 작은 문자열 데이터와 범위, IndexedDB는 비동기 구조화 데이터와 트랜잭션이 핵심이다. 선택은 API 이름보다 수명·공유·데이터 양·보안·실패 요구로 결정한다.
</callout>

---

## 출처

- [MDN — HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)
- [WHATWG — Web Storage](https://html.spec.whatwg.org/multipage/webstorage.html)
- [W3C — Indexed Database API](https://www.w3.org/TR/IndexedDB/)
- [MDN — Storage quotas and eviction](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)

### 이번 설명의 근거와 적용 범위

- [IETF HTTP Working Group — RFC 6265](https://httpwg.org/specs/rfc6265.html): 쿠키의 domain·path·저장·전송 모델. SameSite 등의 현행 속성은 별도 현행 문서와 함께 확인한다.
- [MDN — Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie): Secure·HttpOnly·SameSite·만료와 브라우저 제약.
- [WHATWG — Web Storage](https://html.spec.whatwg.org/multipage/webstorage.html): localStorage·sessionStorage의 범위와 동기 API.
- [MDN — storage 이벤트](https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event): 다른 문서로의 변경 통지.
- [MDN — Using IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB): 스키마·store·index·연결·요청 처리.
- [MDN — IDBTransaction](https://developer.mozilla.org/en-US/docs/Web/API/IDBTransaction): 요청 성공과 최종 완료·abort의 차이.
- [MDN — Storage Quotas and Eviction](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria): 용량·회수·영속 저장의 실제 범위.

---

## 20. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. 이 탭의 임시 초안, 여러 방문에 남길 테마, 큰 오프라인 자료에 각각 어떤 저장 수단을 고르겠는가?
2. localStorage의 값을 바꿨는데 서버 요청의 Cookie 헤더에 없다. 왜 정상인가?
3. IndexedDB 쓰기 요청의 success만 보고 저장 완료를 알리면 어떤 완료 단계를 놓치는가?
4. 저장소에 데이터가 항상 남고 접근도 항상 성공한다고 가정한 코드를 어떻게 보완하겠는가?
