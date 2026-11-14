---
track: B
notion: https://app.notion.com/p/415178c4afac814f88fdcbbc9bbfeb16
---

# Web API란 무엇인가


<callout icon="📍" color="gray_bg">
**학습 범위**: Track B · 1–3주 · Web Platform · Web API란 무엇인가

언어와 실행 환경의 경계를 이해하고 각 API의 입력·결과·보안·수명 계약을 읽는 기준을 익힌다.
</callout>

<table-of-contents />

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- JavaScript 내장 기능·Web API·라이브러리 API·HTTP 엔드포인트를 구별할 수 있다.
- 동기·비동기 결과 전달과 작업 실행 위치를 구별할 수 있다.
- 문서와 Worker 등 제공 환경의 차이를 설명할 수 있다.
- 기능 존재와 실제 호출 허용 여부를 따로 처리할 수 있다.
- 실패한 계층과 정리·취소 요구를 API 문서에서 확인할 수 있다.

---

## 2. 이 주제를 왜 배우는가

JavaScript로 호출할 수 있다고 모두 JavaScript 언어 자체의 기능은 아니다. 배열을 정렬하는 기능과 화면 요소를 만들거나 네트워크 요청을 보내는 기능의 제공 주체는 다르다. 이 경계를 알면 “브라우저에서 되는데 다른 환경에서 안 된다”는 문제를 설명할 수 있다.

---

## 3. 관련 계층과 역할 구분

JavaScript 엔진은 언어를 실행하고 브라우저 호스트 환경은 문서·네트워크·저장소 같은 외부 기능을 노출한다. 웹 표준은 인터페이스의 동작 계약을 정하며 브라우저별 구현·권한 정책이 실제 제공 조건에 영향을 준다.

React 같은 라이브러리와 서버 API는 이 플랫폼 위에서 별도의 계약을 만든다. 같은 함수 호출 문법을 쓰더라도 기능의 제공 주체와 실패 책임을 구별해야 한다.

---

## 4. API와 실행 환경의 관계

API(Application Programming Interface)는 어떤 기능을 사용하는 규칙과 접점이다. 함수, 객체, 프로퍼티, 이벤트 등으로 노출될 수 있다. Web API는 웹 플랫폼이 제공하는 문서 조작, 통신, 저장소, 시간·작업 예약, 장치 접근 등의 기능을 말한다. 여기서 API는 서버의 HTTP 엔드포인트만을 뜻하지 않는다. [MDN Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)는 기능군별 인터페이스를 정리한다.

| 예 | 제공 주체·역할 |
| --- | --- |
| `Array`, `Map`, `Promise`, `JSON` | JavaScript 언어의 표준 내장 기능 |
| `document`, `Element`, DOM 이벤트 | 문서 환경의 Web API |
| `fetch`, `URL`, `localStorage` | 통신·주소·저장을 위한 플랫폼 API |
| React 컴포넌트·Hook | 애플리케이션에서 사용하는 라이브러리 기능 |

여러 환경이 같은 이름의 API를 구현할 수도 있다. 예를 들어 fetch와 URL은 일부 서버 JavaScript 환경에도 제공되지만, 그 환경에 document나 브라우저 저장소까지 있다는 뜻은 아니다. Worker에도 일부 Web API가 있지만 문서 DOM은 없다.

---

## 5. 언어·호스트 환경·라이브러리·서버 API의 차이

### JavaScript 엔진이 코드를 실행한다

엔진은 JavaScript 문법과 값·함수·표준 내장 객체를 처리한다. 배열에 map을 호출하거나 Promise를 만드는 것은 문서가 없는 환경에서도 가능한 언어 기능이다. DOM을 제공하는 것은 브라우저의 문서 환경이므로 엔진만 있다고 document를 사용할 수 있는 것은 아니다.

### 호스트 환경이 외부 기능을 연결한다

브라우저는 파일·네트워크·문서·권한처럼 언어 자체가 직접 소유하지 않는 기능을 인터페이스로 노출한다. window가 있는 문서와 WorkerGlobalScope가 있는 Worker는 사용할 수 있는 기능이 다르다. globalThis는 전역 객체에 접근하는 공통 경로지만 모든 전역 객체의 프로퍼티가 같다는 뜻은 아니다.

### 라이브러리는 플랫폼 기능 위에 별도의 계약을 제공한다

React의 useState나 어떤 통신 라이브러리의 함수는 라이브러리의 API다. 내부에서 Web API를 사용하더라도 그 함수의 캐시·오류·수명 규칙은 해당 라이브러리 문서를 확인해야 한다. 반대로 fetch는 React를 설치하지 않아도 제공될 수 있는 플랫폼 API다.

### HTTP 엔드포인트와 브라우저 fetch는 같은 API가 아니다

서버의 `/api/notes`는 네트워크를 통해 접근할 데이터·기능의 계약이다. fetch는 그 계약에 맞춰 요청할 수 있는 브라우저 측 인터페이스다. 서버가 무엇을 응답하는지와 브라우저가 응답을 읽도록 허용하는지도 구별한다. CORS 오류는 문법 오류나 React state 오류와 같은 단계가 아니다.

---

## 6. API 문서를 읽는 순서

| 확인할 정보 | 질문 |
| --- | --- |
| 노출 환경 | Window·Worker·서버 중 어디에 있는가? |
| 입력과 반환값 | 어떤 값이 필요하고 무엇을 돌려주는가? |
| 결과 전달 방식 | 즉시 반환·Promise·이벤트·콜백 중 무엇인가? |
| 실패 조건 | 예외·Promise 거부·실패 이벤트를 어떻게 받는가? |
| 권한과 보안 | HTTPS·사용자 활성화·권한·origin 제한이 있는가? |
| 수명과 정리 | 취소·해제·연결 종료가 필요한가? |
| 호환성 | 지원 브라우저와 버전에서 어떤 제한이 있는가? |

이 정보는 함수 이름 하나를 외우는 것보다 실제 구현에 직접 연결된다. 메서드가 존재하더라도 권한이 거부되거나 입력이 잘못되면 호출은 실패한다. 반환된 Promise의 완료가 실제 화면 표시나 다른 시스템의 최종 처리 완료 중 어디까지 의미하는지도 확인한다.

---

## 7. 기능을 호출하고 결과를 받는 흐름

동기 API는 호출 중 결과를 반환한다. DOM 요소를 만드는 일이나 localStorage 접근이 예다. 비동기 API는 요청을 시작하고 결과를 나중에 Promise나 이벤트·콜백으로 전달할 수 있다. 비동기라는 말은 자동으로 모든 작업이 별도 스레드에서 실행되어 비용이 사라진다는 뜻이 아니다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>환경의 API</title></head>
<body>
  <h1>주소 해석</h1>
  <p id="result"></p>
  <script>
    const names = ['HTML', 'DOM'];
    const text = names.join(' / '); // JavaScript 내장 배열 기능
    const output = document.querySelector('#result'); // DOM API
    const address = new URL('/notes?q=web', 'https://example.com');

    output.textContent = `${text}: ${address.pathname}`;
    console.log(address.searchParams.get('q'));
    console.log('fetch 지원:', typeof globalThis.fetch === 'function');
  </script>
</body>
</html>
```

브라우저에서 열면 문단에 `HTML / DOM: /notes`가 표시되고 콘솔에 `web`이 나온다. URL 객체 생성은 문자열을 해석할 뿐 그 주소로 요청하거나 이동하지 않는다. `textContent` 지정은 DOM을 변경한다. 동일한 JavaScript 문법으로 사용하지만 각 API가 하는 일은 구별된다.

---

## 8. 동기·비동기·별도 실행을 구별한다

동기 호출은 현재 호출 흐름에서 결과를 얻는다. 비동기 호출은 결과를 나중에 전달하는 계약이다. 별도 스레드나 실행 환경은 작업이 어디에서 수행되는지의 문제다. 이 세 축을 섞으면 async 함수를 만들기만 해도 큰 계산이 메인 스레드에서 사라진다고 오해하기 쉽다.

Promise로 감쌌더라도 안에서 곧바로 실행하는 긴 계산은 여전히 현재 실행 환경의 시간을 사용할 수 있다. Worker는 별도 실행 환경을 제공하지만 시작·메시지 전달 비용이 있다. 세부 스케줄링은 7–9주 JS Runtime / Browser Scheduling에서 다루고, 이 페이지에서는 반환 방식만으로 작업 위치를 단정하지 않는 기준을 익힌다.

### 비동기 결과가 올 때 화면이 아직 있는가

요청을 시작한 화면이 사라진 뒤 응답이 도착할 수 있다. 관찰자는 필요 없는 대상의 변경을 계속 알릴 수 있다. API를 사용할 때 성공 콜백만 작성하지 말고 취소·해제·오래된 결과 무시 같은 수명 조건을 확인한다. 이후 fetch·관찰자·Worker에서 각각 구체화한다.

---

## 9. 제공되는 기능과 허용되는 기능은 다르다

API가 존재해도 보안 컨텍스트(보통 HTTPS), 사용자 권한, 사용자 활성화, iframe 정책 등에 따라 호출이 실패할 수 있다. 현재 문서의 origin도 중요하다. 웹의 origin은 일반적으로 스킴·호스트·포트의 조합으로, 문서와 데이터 접근의 기본 경계가 된다. `https://example.com`과 `http://example.com`은 같은 origin이 아니다.

따라서 기능 존재 여부를 확인하는 feature detection과 실제 호출의 오류 처리를 함께 사용한다. 브라우저 이름만으로 지원을 추정하거나, 권한을 이미 받았다고 가정하지 않는다. 특정 API가 문서와 Worker 중 어디에 노출되는지도 문서에서 확인한다.

---

## 10. 기능 감지와 실패 처리는 함께 필요하다

```javascript
function canUseClipboard() {
  return typeof navigator !== 'undefined'
    && typeof navigator.clipboard?.writeText === 'function';
}
console.log('인터페이스 존재:', canUseClipboard());
console.log('보안 컨텍스트:', globalThis.isSecureContext);
```

이 코드는 존재 여부를 관찰할 뿐 클립보드에 쓰거나 권한을 요청하지 않는다. true라고 실제 쓰기 권한이나 사용자 활성화 조건까지 충족한 것은 아니다. 구현에서는 명확한 사용자 동작 시점에 호출하고 실제 실패를 처리해야 한다.

API가 없을 때 경고만 출력하고 기능 전체를 멈출지, 수동 복사 등 대안을 제공할지는 제품의 요구에 따라 결정한다. 권한이 없는 사용자를 오류 화면에 가두기보다 현재 가능한 작업과 재시도 조건을 알려준다.

### 디버깅할 계층을 좁히기

ReferenceError로 이름 자체가 없다면 노출 환경·로드 시점을 확인한다. Promise가 거부되면 오류 이름·입력·권한·보안 조건을 확인한다. 응답은 왔는데 화면만 틀리면 데이터 해석과 UI 반영을 조사한다. 모든 문제를 “브라우저 호환성”으로 뭉뚱그리지 않고 관찰한 실패 지점을 구별한다.

---

## 11. Web API의 계약을 인터페이스·상태·환경으로 나누기

### API는 함수 이름 목록보다 넓은 계약이다

메서드가 무엇을 받는지뿐 아니라 어느 객체에서 호출하는지, 어떤 상태에서 사용할 수 있는지, 결과를 언제 전달하는지가 API의 일부다. 같은 이름이 있어도 환경마다 노출 범위·권한·수명 조건이 다를 수 있다. 함수가 존재한다는 관찰 하나를 “기능을 반드시 성공적으로 사용할 수 있다”로 바꾸지 않는다.

DOM의 document.createElement, 네트워크의 fetch, 저장소의 localStorage는 전부 JavaScript 문법으로 호출하지만 외부 세계와 맺는 계약이 다르다. JavaScript 언어의 배열·문자열과 호스트가 제공하는 문서·네트워크·권한을 구별하면 문제를 조사할 문서도 명확해진다.

### 인터페이스와 인스턴스

인터페이스는 제공할 프로퍼티·메서드·이벤트의 형태와 동작을 설명한다. 실제 문서에서 사용하는 document나 요소 객체는 그 계약을 구현한 객체다. 표준에는 JavaScript 표면과 연결하기 위한 [Web IDL](https://webidl.spec.whatwg.org/) 정의가 사용된다. IDL의 문법을 모두 배우는 것이 목표는 아니며, 노출 환경·인수·반환 타입·예외 같은 정보를 읽는 접점이라는 점을 이해한다.

### 객체를 생성하는 API와 전역 객체를 사용하는 API

URL·AbortController처럼 new로 객체를 만들기도 하고, document·navigator처럼 환경이 제공한 객체에 접근하기도 한다. 모든 Web API를 new로 만들 수 있거나 모든 메서드를 window에 직접 붙은 함수처럼 호출할 수 있는 것은 아니다. 문서의 생성 방식과 사용 예제를 함께 확인한다.

---

## 12. 언어 기능과 환경 기능을 직접 비교하기

### 실험 1. 같은 문법과 다른 제공 주체

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>환경 기능 관찰</title></head>
<body>
  <h1>사용 가능한 기능</h1>
  <pre id="output"></pre>
  <script>
    const data = {
      array: typeof Array,
      promise: typeof Promise,
      global: typeof globalThis,
      document: typeof globalThis.document,
      fetch: typeof globalThis.fetch,
      worker: typeof globalThis.Worker,
      secureContext: globalThis.isSecureContext,
    };
    document.querySelector('#output').textContent = JSON.stringify(data, null, 2);
  </script>
</body>
</html>
```

Array와 Promise는 언어 기능이다. document·fetch·Worker·isSecureContext는 호스트의 제공 조건과 관련된다. 같은 코드를 문서가 없는 환경에 복사하면 출력 단계의 document 접근이 실패할 수 있다. 그렇다고 JavaScript 언어가 다른 언어로 바뀐 것은 아니다. 어떤 전역이 제공되는지와 프로그램이 어떤 환경을 전제로 하는지 구별한다.

이 출력은 지원 현황을 관찰하는 작은 도구일 뿐 권한·모든 메서드·전체 브라우저 버전의 호환성을 증명하지 않는다. 실제 기능을 고른 뒤 해당 API의 지원 범위와 실패 조건을 추가 확인해야 한다.

### 실험 2. 메서드를 떼어 호출할 때의 수신 객체

브라우저 콘솔에서 실행한다.

```javascript
{
  const create = document.createElement;
  try {
    create('p');
  } catch (error) {
    console.log(error.name);
  }
  const paragraph = create.call(document, 'p');
  console.log(paragraph.tagName);
}
```

첫 호출은 Document를 수신 객체로 사용하지 않아 TypeError가 발생할 수 있고, 둘째는 document를 명시하여 P를 만든다. 오류 메시지의 정확한 문구는 브라우저마다 다를 수 있다. 중요한 것은 메서드를 변수로 저장했다는 사실이 원래의 this 관계를 자동 보존하지 않는다는 점이다. [Web IDL의 연산 바인딩](https://webidl.spec.whatwg.org/#es-operations)이 호스트 인터페이스의 호출 검사를 설명한다.

모든 함수를 무조건 bind해야 한다는 결론은 아니다. 해당 메서드가 어떤 수신 객체 계약을 가지는지 확인한다. 이 문제와 API 미지원, 보안 권한 거부는 서로 다른 오류 원인이다.

### 실험 3. JavaScript 값이 인터페이스 값으로 변환되는 경우

```javascript
{
  const input = document.createElement('input');
  input.disabled = 'false';
  console.log(input.disabled, input.hasAttribute('disabled'));
  input.disabled = false;
  console.log(input.disabled, input.hasAttribute('disabled'));
}
```

문자열 false는 비어 있지 않은 문자열이므로 boolean 인터페이스 값으로 변환될 때 true가 된다. 실제 boolean false를 전달해야 해제된다. API가 값을 받는 방식과 문자열의 인간 언어적 의미는 다를 수 있다. 입력 타입·형식과 반환값을 문서에서 확인해야 하는 이유다.

---

## 13. 동기·Promise·이벤트로 전달되는 실패를 구별하기

### 동기 예외는 현재 호출 안에서 발생한다

```javascript
try {
  new URL('주소가 아닌 문자열');
} catch (error) {
  console.log('동기 실패:', error.name);
}
```

기준 주소 없이 유효하지 않은 절대 주소를 만들면 URL 생성 호출에서 예외가 난다. 이런 API에는 해당 호출을 감싸는 try·catch가 의미 있다. 실패가 어떤 시점에 전달되는지 모른 채 모든 API에 동일한 오류 처리 코드를 붙이지 않는다.

### Promise 실패는 비동기 결과 경로에서 처리한다

```javascript
async function inspectJson() {
  const response = new Response('JSON이 아닌 문자열', {
    headers: { 'Content-Type': 'application/json' },
  });
  try {
    await response.json();
  } catch (error) {
    console.log('본문 해석 실패:', error.name);
  }
}
inspectJson();
```

이 코드는 네트워크 요청 없이 Response 본문 읽기의 실패를 재현한다. Content-Type을 JSON으로 적는 것이 본문을 올바른 JSON으로 고쳐 주지는 않는다. json이 반환한 Promise의 실패를 처리해야 하며, 호출만 하고 결과를 버리면 실패를 놓칠 수 있다.

### 이벤트로 실패를 통지하는 API도 있다

이미지·Worker·여러 연결 API 등은 로딩이나 실행의 실패를 이벤트로 통지할 수 있다. 실제 문서에서 addEventListener로 해당 실패 이벤트를 관찰해야 하는지, 생성자 호출에서 즉시 예외도 생길 수 있는지 확인한다. “비동기 API는 전부 Promise”라고 일반화하지 않는다.

같은 기능 안에서도 동기 입력 검사, 나중의 요청 실패, 업무 결과의 실패가 공존할 수 있다. 성공한 호출과 성공한 작업의 범위를 구별하여 사용자에게 안내한다.

---

## 14. origin·보안 컨텍스트·권한을 서로 다른 경계로 읽기

### 실험 4. 주소 문자열의 origin

```javascript
{
  const addresses = [
    'https://example.com/a',
    'https://example.com:443/b',
    'http://example.com/a',
    'https://example.com:8443/a',
  ];
  console.table(addresses.map((value) => ({ value, origin: new URL(value).origin })));
}
```

첫째와 둘째는 HTTPS의 기본 포트가 정규화되어 같은 origin으로 표현된다. HTTP로 바뀌거나 다른 유효 포트를 쓰면 다르다. 경로가 다르다는 사실만으로 다른 origin이 되는 것은 아니다. [HTML origin 모델](https://html.spec.whatwg.org/multipage/browsers.html#origin)은 문서 접근 경계의 기준이다.

### opaque origin의 문자열을 같은 권한으로 오해하지 않는다

일부 주소·문서는 일반적인 스킴·호스트·포트 튜플 대신 opaque origin을 사용한다. 그 직렬화가 null로 나타난다고 두 문서가 동일한 권한을 공유한다는 뜻은 아니다. 모든 URL의 origin 문자열 비교를 보안 판정의 만능 함수로 사용하지 않는다. 파일을 직접 연 환경과 HTTP로 제공한 환경의 동작을 같은 배포 조건으로 간주하지 않는 이유이기도 하다.

### 보안 컨텍스트는 origin 동일성 검사와 다르다

일부 강력한 API는 보안 컨텍스트에서만 노출된다. 보통 HTTPS가 중요하지만 개발용 loopback 주소 같은 잠재적으로 신뢰할 수 있는 조건도 있다. isSecureContext로 현재 환경을 관찰하고 API 문서의 요구를 확인한다. 서로 다른 HTTPS 사이트가 모두 보안 컨텍스트일 수 있어도 같은 origin인 것은 아니다. [MDN Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts)를 이 구분의 근거로 사용한다.

### 권한·사용자 활성화·정책은 추가 조건이다

API가 있고 보안 컨텍스트여도 사용자가 권한을 거절하거나 iframe의 정책이 사용을 제한할 수 있다. 어떤 API는 클릭 같은 사용자 활성화와 호출 시점도 요구한다. 권한을 미리 물어두면 모든 후속 호출이 언제나 성공한다는 모델로 설계하지 않는다. 실제 호출 결과를 처리해야 한다.

---

## 15. 존재 확인·실제 호출·대체 흐름을 연결하기

### 실험 5. 명시적인 사용자 동작에서 클립보드 사용

아래 예제는 사용자가 버튼을 눌렀을 때 예제 문자열을 클립보드에 쓴다. 지원 조건을 갖춘 HTTP 개발 환경 또는 HTTPS 문서에서 실행하고, 호출이 실패하면 사용자가 직접 복사할 수 있는 텍스트를 남긴다.

```html
<!doctype html>
<html lang="ko">
<head><meta charset="utf-8"><title>클립보드 기능과 대안</title></head>
<body>
  <label for="text">복사할 내용</label>
  <textarea id="text" readonly>웹 플랫폼 학습 노트</textarea>
  <button id="copy" type="button">내용 복사</button>
  <p id="status" role="status"></p>
  <script>
    const text = document.querySelector('#text');
    const status = document.querySelector('#status');
    document.querySelector('#copy').addEventListener('click', async () => {
      if (typeof navigator.clipboard?.writeText !== 'function') {
        status.textContent = '텍스트를 선택하여 직접 복사하세요.';
        text.focus();
        text.select();
        return;
      }
      try {
        await navigator.clipboard.writeText(text.value);
        status.textContent = '복사했습니다.';
      } catch (error) {
        status.textContent = '자동 복사를 사용할 수 없습니다. 직접 복사하세요.';
        text.focus();
        text.select();
        console.error(error);
      }
    });
  </script>
</body>
</html>
```

존재 확인은 미지원 경로를 구별하고 try·catch는 실제 허용·실행 실패를 처리한다. 둘은 대체 관계가 아니다. readonly 텍스트를 남겨 자동 기능을 사용할 수 없는 환경에서도 목적을 달성할 수 있게 했다. [Clipboard.writeText](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText)의 보안·Promise 계약을 따른 예제다.

### 권한 조회도 모든 API에 동일하지 않다

Permissions API가 있어도 모든 권한 이름을 모든 브라우저가 지원하는 것은 아니다. query의 결과가 현재 상태를 알려줄 수 있지만 실제 기능의 호출 조건까지 모두 대신하지는 않는다. 권한 조회 자체의 실패와 기능 사용 실패를 구별하고 필요하지 않은 권한을 앱 초기 로드부터 요구하지 않는다.

---

## 16. 자원과 등록의 수명을 API 계약에 포함하기

### 객체 URL은 문자열이지만 관리할 자원과 연결된다

```javascript
{
  const blob = new Blob(['예제 메모'], { type: 'text/plain;charset=utf-8' });
  const address = URL.createObjectURL(blob);
  console.log(address.startsWith('blob:'));
  URL.revokeObjectURL(address);
}
```

이 코드는 문서 환경에서 예제 Blob을 가리키는 객체 URL을 만들고 해제한다. 임의의 원격 파일을 다운로드하지 않는다. 문자열 참조를 잊는 것과 해당 URL이 보관하는 자원을 해제하는 일은 다를 수 있어 실제 소비가 끝난 시점을 정해야 한다. 너무 일찍 해제하면 소비하지 못하고 너무 늦으면 자원 수명이 늘어날 수 있다. [MDN 객체 URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL_static)의 생성·해제 계약을 확인한다.

### 취소·정리 메서드의 목적이 다르다

타이머의 clear, 리스너 제거, 관찰자의 disconnect, 요청의 abort, Worker의 terminate는 모두 정리와 관련되지만 결과가 같지는 않다. 어떤 것은 다음 콜백만 막고, 어떤 것은 연결된 작업을 중단하며, 어떤 것은 실행 환경 자체를 끝낸다. 이미 완료한 외부 작업의 되돌리기까지 보장한다고 일반화하지 않는다.

### 비동기 함수라는 표면만으로 비용을 판단하지 않는다

async 함수 안에서 즉시 긴 계산을 하면 첫 await에 도달하기 전까지 현재 실행 흐름의 시간을 사용한다. Promise를 돌려준다는 사실은 별도 스레드의 증거가 아니다. 처리 위치·콜백 전달·정리 방식을 해당 API별로 확인하며, 스케줄링 세부는 7–9주 JS Runtime / Browser Scheduling에서 확장한다.

---

## 17. 표준·구현·제품 요구의 증거를 구분하기

### 표준은 계약, 호환성 문서는 구현 범위다

WHATWG·W3C 표준은 인터페이스와 처리 절차를 규정한다. MDN의 지원 정보나 브라우저 벤더 문서는 특정 구현의 제공 범위·제약을 확인하는 데 쓰인다. 표준 초안에 있다고 모든 안정 버전에서 사용할 수 있는 것은 아니므로 새로운 기능은 대상 환경과 상태를 명시한다.

### 콘솔의 존재 확인과 실제 사용자 시험

typeof 검사는 객체 존재를 알려준다. 실제 클릭, 권한 거절, 보안 연결, iframe, 화면 이탈, 재시도 시험은 다른 조건을 검증한다. 지원되는 환경에서 한 번 성공한 결과를 모든 환경의 보장으로 확대하지 않는다. 실패했을 때도 사용자에게 가능한 다음 행동을 제공해야 기능이 완성된다.

### 성능 측정은 같은 작업을 비교한다

API 호출을 줄였더라도 실제 요청 크기·DOM 변경·계산 비용이 늘면 사용자는 더 느리게 느낄 수 있다. 호출 횟수·전달 데이터·메모리 수명·사용자 응답 시간을 함께 기록한다. 비동기 표면, 라이브러리 사용 여부, 코드 줄 수만으로 빠르다고 결론 내리지 않는다.

---

## 18. 실무 적용과 디버깅

원하는 기능의 제공 환경, 입력·반환 타입, 동기·비동기 여부, 오류 조건, 권한 요구, 대상 브라우저 지원을 순서대로 확인한다. Promise가 반환되면 결과뿐 아니라 실패도 처리한다. 보안 제한을 브라우저 오류로 취급하여 무작정 우회하는 대신 허용된 데이터 흐름을 설계한다.

사용자의 위치나 장치처럼 권한이 필요한 기능은 목적이 드러나는 시점에 요청하고 거부되었을 때의 사용 흐름을 준비한다. 성능은 API 호출 횟수와 실제로 발생시키는 작업에 달려 있다. 동기 저장소나 DOM 측정을 무거운 루프 안에서 반복하는지 확인한다.

---

## 19. 핵심 정리

<callout icon="✅" color="green_bg">
JavaScript는 코드를 표현하는 언어이고 Web API는 실행 환경의 기능에 접근하는 접점이다. 제공 여부와 호출 허용 여부를 구별하고, 각 기능의 결과 전달·실패·비용을 확인해야 한다. 이후 fetch·저장소·관찰자·Worker는 이 분류의 구체적인 사례다.
</callout>

---

## 출처

- [MDN — Web APIs](https://developer.mozilla.org/en-US/docs/Web/API)
- [MDN — Web API 개요](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Client-side_APIs/Introduction)
- [WHATWG — URL API](https://url.spec.whatwg.org/#api)
- [MDN — Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts)

### 이번 설명의 근거와 적용 범위

- [WHATWG — Web IDL](https://webidl.spec.whatwg.org/): 인터페이스·호출·JavaScript 바인딩의 계약.
- [WHATWG — Origins](https://html.spec.whatwg.org/multipage/browsers.html#origin): 튜플·opaque origin과 문서 경계.
- [MDN — Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Secure_Contexts): 보안 컨텍스트와 잠재적으로 신뢰할 수 있는 환경.
- [MDN — Clipboard.writeText](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText): 실제 클립보드 호출의 보안·오류 조건.
- [MDN — Permissions.query](https://developer.mozilla.org/en-US/docs/Web/API/Permissions/query): 권한 조회와 지원 이름의 제약.
- [MDN — createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL_static): 객체 URL 자원의 수명 관리.

---

## 20. 이해도 확인 질문

<callout icon="📝" color="purple_bg">
코드와 상황의 결과를 먼저 예측하고, 근거와 검증 방법을 함께 설명한다. 답과 피드백은 대화에서만 다룬다.
</callout>

1. `Array.map`은 동작하지만 `document`가 없는 실행 환경이 있다. 무엇이 다른 것인가?
2. new URL을 호출했는데 Network에 요청이 없다. 왜 정상적인가?
3. API가 존재하는데 호출이 거부된다. 지원 여부 외에 어떤 조건을 확인해야 하는가?
4. 비동기 API를 호출하면 긴 계산도 화면을 절대 막지 않는다는 주장에 어떤 문제가 있는가?
