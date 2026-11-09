---
track: C
notion: https://app.notion.com/p/40c178c4afac8165bcbad222e6bf155d
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track C · Network / HTTP · Same-Origin Policy / CORS

브라우저가 다른 출처의 데이터를 읽는 것을 제한하는 이유와 서버가 공유를 허용하는 과정을 다룬다. fetch·XHR의 CORS 요청을 중심으로 설명한다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 출처와 사이트의 차이를 예시로 설명한다.
- 요청 전송·응답 수신·스크립트의 응답 읽기를 구분한다.
- 단순 요청과 프리플라이트가 필요한 요청의 흐름을 복원한다.
- 쿠키를 포함한 CORS 요청과 관련 헤더를 분석한다.
- CORS 오류를 요청·서버·프록시·브라우저의 책임으로 나누어 조사한다.

## 2. 이 주제를 왜 배우는가

사용자가 은행 사이트에 로그인한 채 공격자의 사이트를 열었다고 하자. 공격자 페이지의 스크립트가 은행에 요청을 보내 응답을 자유롭게 읽을 수 있다면, 사용자의 브라우저가 가지고 있는 로그인 권한으로 개인정보가 유출될 수 있다.

**동일 출처 정책(Same-Origin Policy, SOP)** 은 서로 다른 출처의 문서·스크립트 사이에 접근 경계를 두는 브라우저 보안 모델이다. **CORS(Cross-Origin Resource Sharing)** 는 서버가 특정 다른 출처의 스크립트에 응답을 공유하도록 허용하는 HTTP 기반 절차다. 프론트엔드와 API 호스트가 다르면 이 경계를 명시적으로 다뤄야 한다.

## 3. 출처는 스킴·호스트·포트의 조합이다

`https://app.example.com`과 비교해 보자. 기본 HTTPS 포트는 443으로 정규화해 비교한다.

| URL | 같은 출처인가 | 이유 |
| --- | --- | --- |
| `https://app.example.com/users` | 예 | 경로는 출처 구성 요소가 아님 |
| `https://app.example.com:443/` | 예 | 생략된 기본 포트와 같음 |
| `http://app.example.com/` | 아니오 | 스킴이 다름 |
| `https://api.example.com/` | 아니오 | 호스트가 다름 |
| `https://app.example.com:8443/` | 아니오 | 포트가 다름 |

쿠키의 SameSite가 사용하는 **사이트**는 일반적으로 스킴과 등록 가능한 도메인 기준이다. `https://app.example.com`과 `https://api.example.com`은 같은 사이트일 수 있지만 다른 출처다. 그래서 SameSite 조건에 맞는 쿠키라도 fetch의 자격 전송 설정과 CORS 응답 허용을 따로 확인해야 한다.

## 4. SOP가 모든 네트워크 요청을 금지하는 것은 아니다

다른 출처로 이동하거나 폼을 보내고 일부 리소스를 삽입하는 일은 가능하다. 대신 스크립트가 다른 출처의 민감한 응답 내용을 마음대로 읽는 것에는 제한이 있다. 삽입 가능한 리소스도 종류와 다른 보안 정책에 따라 제한이 달라진다.

따라서 **요청이 서버에 도착했는가**, **브라우저가 응답을 받았는가**, **스크립트가 응답을 읽을 수 있는가**를 나누어야 한다. Network 패널에 200이 보여도 자바스크립트에는 CORS 실패로 응답이 노출되지 않을 수 있다.

CORS는 보통 브라우저가 집행한다. 일반적인 서버 간 HTTP 클라이언트나 curl은 같은 SOP 제약을 받지 않는다. curl에서 성공한다는 것은 서버 통신 가능성의 근거이며 브라우저의 공유 정책까지 검증한 것은 아니다. [MDN Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) 참조.

## 5. 프리플라이트 없는 요청의 흐름

CORS 조건에서 GET·HEAD 또는 제한된 POST와 허용 목록의 헤더·값을 사용하는 요청은 흔히 **단순 요청(simple request)** 이라고 부른다. Content-Type도 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain` 같은 허용 조건이 있다. 메서드 이름 하나만으로 결정하지 않는다.

```text
브라우저 -- Origin을 포함한 실제 요청 --> API
브라우저 <-- 응답과 CORS 허용 헤더 ------ API
브라우저: 요청 출처와 허용 정보를 비교
  -> 허용이면 스크립트에 응답 공유
  -> 불허이면 스크립트의 응답 읽기 실패
```

이 흐름에서는 허용하지 않는 응답이라도 실제 요청은 이미 서버에서 실행되었을 수 있다. CORS 설정만으로 상태 변경 요청 자체나 CSRF를 모두 차단했다고 생각하면 안 된다.

## 6. 프리플라이트가 필요한 요청

PATCH·DELETE 같은 메서드, `Content-Type: application/json`, Authorization 같은 허용 목록 밖의 헤더는 일반적인 CORS 요청에서 **프리플라이트(preflight)** 를 요구할 수 있다. 브라우저가 먼저 OPTIONS로 실제 요청의 메서드·헤더를 허용하는지 확인한다.

```http
OPTIONS /profile HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Access-Control-Request-Method: PATCH
Access-Control-Request-Headers: content-type
```

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: PATCH
Access-Control-Allow-Headers: Content-Type
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 600
Vary: Origin

```

`Origin`은 실행 문서의 출처다. `Access-Control-Request-*`는 앞으로 보내려는 메서드와 추가 헤더 이름을 알린다. `Access-Control-Allow-*`는 서버의 허용 판단이다. OPTIONS 자체가 204라고 충분한 것은 아니며 필요한 허용 필드가 조건에 맞아야 한다.

일반적인 프리플라이트에는 사용자 쿠키 같은 자격 증명을 보내지 않는다. OPTIONS를 무조건 로그인 필요로 막으면 실제 요청을 시작하지 못할 수 있다. 허용 정책은 확인하되 실제 PATCH 등 본 요청에서 인증과 권한 검사를 수행한다. [Fetch CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol) 참조.

## 7. 실제 요청과 응답에도 허용 헤더가 필요하다

앞의 확인이 성공하면 브라우저는 실제 요청을 보낸다. 아래는 `api.example.com`의 유효한 쿠키가 있고 전송 조건이 맞는 경우의 예시다.

```http
PATCH /profile HTTP/1.1
Host: api.example.com
Origin: https://app.example.com
Content-Type: application/json
Content-Length: 14
Cookie: session=demo_random_id

{"name":"pen"}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 11
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: X-Request-ID
X-Request-ID: demo-123
Vary: Origin

{"ok":true}
```

프리플라이트에서 허용해도 실제 응답의 허용 헤더가 빠지면 스크립트의 응답 읽기가 실패한다. 오류 응답에서도 정책에 맞는 CORS 헤더가 필요하다. 그렇지 않으면 서버는 401·500을 보냈는데 클라이언트는 구체적인 상태를 읽지 못할 수 있다.

`Access-Control-Expose-Headers`는 기본으로 공개되는 범위 밖의 응답 헤더를 스크립트가 읽도록 알린다. `Allow-Headers`는 보내는 요청 헤더, `Expose-Headers`는 읽을 응답 헤더에 대한 정보다. CORS 허용이 브라우저의 모든 금지 헤더를 자유롭게 만들거나 Set-Cookie를 읽게 하는 것은 아니다.

## 8. 쿠키와 credentials, 와일드카드의 조건

브라우저 fetch의 자격 전송 기본값은 `same-origin`이다. 다른 출처 API에 쿠키를 포함하려면 보통 다음처럼 명시한다.

```javascript
fetch("https://api.example.com/profile", {
  method: "PATCH",
  credentials: "include",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ name: "pen" }),
});
```

`include`가 쿠키 정책을 무시하는 것은 아니다. Domain·Path·Secure·SameSite, 브라우저의 제3자 쿠키 제한 등도 만족해야 한다. 같은 사이트의 다른 출처와 사이트 자체가 다른 요청을 구분한다.

자격 증명을 포함하는 요청의 응답을 공유하려면 서버가 정확한 허용 출처와 `Access-Control-Allow-Credentials: true`를 제공해야 한다. 이 경우 `Access-Control-Allow-Origin: *`를 사용하면 안 된다. 이 헤더에 여러 출처를 쉼표로 나열하는 것도 올바른 허용 방식이 아니다.

서버가 Origin에 따라 응답을 달리하면 캐시가 출처별 응답을 구분하도록 `Vary: Origin` 등을 고려한다. Origin을 아무 검증 없이 그대로 반사하고 자격 공유를 허용하면 공격자 출처에도 민감한 응답을 공개할 수 있다.

## 9. 관련 계층과 책임 구분

| 참여자 | 책임 |
| --- | --- |
| 브라우저 | 출처 계산, 필요 시 프리플라이트, 응답 노출 여부 결정 |
| API 서버 | 허용 출처·메서드·헤더 정책 응답, 본 요청 인증·권한 검사 |
| 프록시·CDN | OPTIONS 전달·응답, 헤더 보존, 캐시 구분 |
| TCP·TLS·IP | 통신 전달과 구간 보호, CORS 허용 의미를 대신 판단하지 않음 |

Origin은 인증 자격이 아니다. 브라우저는 이를 관리하지만 브라우저 밖의 클라이언트는 임의 헤더를 보낼 수 있다. 허용한 출처라는 이유만으로 모든 사용자의 데이터 접근을 승인하면 안 된다.

## 10. Network 패널로 오류를 좁히는 순서

1. 페이지 출처와 API 출처를 스킴·호스트·포트로 비교한다.
2. Console의 CORS 설명과 Network의 OPTIONS 유무를 함께 본다.
3. OPTIONS만 있고 본 요청이 없으면 허용 메서드·헤더·출처·리다이렉트를 확인한다.
4. 본 요청까지 있으면 실제 응답의 CORS 헤더와 오류 응답 경로를 확인한다.
5. 쿠키 문제라면 credentials와 쿠키 저장·전송 차단 사유를 별도로 본다.
6. 프록시가 붙인 중복 허용 헤더나 캐시된 다른 출처의 응답도 확인한다.

프리플라이트 결과는 일정 시간 캐시될 수 있으므로 매번 OPTIONS가 보이지 않을 수 있다. 반대로 OPTIONS가 성공해도 실제 요청의 인증이 성공했다는 뜻은 아니다. 네트워크 연결 오류나 혼합 콘텐츠 차단을 모두 CORS 설정 문제로 취급하지 않는다.

## 11. 흔한 오해·보안·성능·운영 관점

**`mode: "no-cors"`는 해결책이 아니다.** 메서드·헤더 사용이 제한되고 응답이 불투명해져 스크립트가 본문·상태를 자유롭게 읽지 못한다. JSON API를 읽기 위한 권한을 우회하는 설정이 아니다.

**CORS 허용은 CSRF 방어 전체가 아니다.** 프리플라이트 없는 요청은 이미 실행될 수 있고 쿠키는 조건에 따라 자동 전달된다. 서버의 인증·권한 검사와 위조 요청 방어를 함께 유지한다.

프리플라이트는 추가 왕복을 만들므로 요청 구성과 `Access-Control-Max-Age`가 지연에 영향을 준다. 그러나 성능을 위해 필요한 보안 정책을 제거하거나 모든 출처를 허용하지 않는다. 브라우저별 캐시 상한과 정책도 실제 측정에 영향을 줄 수 있다.

## 12. 핵심 정리

<callout icon="✅" color="green_bg">
SOP는 브라우저의 출처 경계이고 CORS는 서버가 그 경계를 넘어 응답을 공유하는 절차다. 요청 실행과 스크립트의 응답 읽기는 구분해야 한다.
</callout>

- 출처는 스킴·호스트·포트이며 SameSite의 사이트와 다르다.
- 프리플라이트는 본 요청의 전송 허용을 먼저 확인한다.
- 본 응답에도 올바른 허용 헤더가 필요하다.
- 쿠키 전송, 응답 공유, 사용자 권한 검사는 별개의 조건이다.

## 출처

- [MDN — Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy): 출처 비교와 접근 경계.
- [Fetch Standard — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol): 프리플라이트·자격·응답 공유의 규칙.
- [MDN — CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS): 요청 유형과 CORS 헤더.
- [MDN — Using Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch): credentials와 불투명 응답.
- [MDN — Site](https://developer.mozilla.org/en-US/docs/Glossary/Site): 사이트와 출처의 차이.

## 13. 이해도 확인 질문

1. `https://app.example.com`과 `https://api.example.com`은 같은 사이트일 수 있는데도 CORS 설정이 필요한 이유는 무엇인가?
2. Network에 POST 200이 보이지만 자바스크립트는 CORS 오류를 받았다. 서버에서 작업이 실행됐을 가능성과 응답 읽기 권한을 구분해 보라.
3. OPTIONS만 보이고 PATCH가 전송되지 않았다. 어떤 요청·응답 헤더와 중간 장비를 확인하겠는가?
4. credentials가 include인 요청에 `Access-Control-Allow-Origin: *`와 Allow-Credentials가 함께 왔다. 브라우저가 응답을 공유할 수 있는가?
5. 쿠키가 필요한 API를 안전하게 공개하려면 CORS 외에 어떤 인증·권한·위조 요청 방어 조건을 확인해야 하는가?
