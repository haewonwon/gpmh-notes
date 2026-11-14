---
track: A
notion: https://app.notion.com/p/415178c4afac814da827eff4e8b7f998
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · Module · ESM / CommonJS 개념

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 모듈의 역할과 공개 인터페이스를 설명한다.
- ESM의 named/default export와 import를 구분한다.
- CommonJS의 `require`·`module.exports`를 읽고 잘못된 export를 고친다.
- 실행 환경·확장자·모듈 형식에 따른 오류를 진단한다.

## 2. 이 주제를 왜 배우는가

모든 코드를 하나의 파일이나 전역 공간에 두면 같은 이름이 충돌하고, 다른 기능의 내부 변수에 의존하기 쉽다. **모듈(module)** 은 코드의 경계를 만들고 필요한 기능만 외부에 공개하는 단위다. 파일 분할 자체보다 어떤 값을 공개하고 어느 모듈에 의존하는지가 중요하다.

**ESM(ECMAScript Modules)** 은 언어 표준의 모듈 체계다. **CommonJS(CJS)** 는 Node.js 생태계에서 널리 사용하는 별도의 체계다. 둘 다 JavaScript를 실행하지만 로딩·내보내기·해석 규칙이 다르다.

### 모듈 경계는 의존 방향을 만든다

두 파일이 서로의 내부 상태를 수정할 수 있다면 파일은 나뉘었어도 책임 경계는 약하다. 모듈은 공개된 함수와 값을 통해 협력하고, 내부 구현을 바꾸어도 호출자의 계약을 유지하는 데 의미가 있다. import 목록은 해당 코드가 무엇에 의존하는지 보여 주는 지도다.

반대로 파일을 지나치게 잘게 나누면 초기화와 의존 경로가 오히려 복잡해질 수 있다. 하나의 규칙과 상태를 함께 변경하는 기능을 한 모듈로 묶고, 외부에 필요한 인터페이스를 작게 내보내는 것을 기준으로 삼는다.

## 3. ESM의 공개와 가져오기

다음 두 파일을 같은 디렉터리에 저장한다. `.mjs` 확장자는 `Node.js에서` ESM임을 명시한다.

```javascript
// math.mjs
export const unit = "KRW";
export function add(a, b) {
  return a + b;
}
export default function double(n) {
  return n * 2;
}
```

```javascript
// main.mjs — node main.mjs
import double, { add, unit as currency } from "./math.mjs";
console.log(add(2, 3), double(4), currency); // 5 8 KRW
```

**named export**는 공개 이름으로 가져오며 `as`로 지역 이름을 바꿀 수 있다. **default export**는 모듈당 하나이며 가져오는 쪽에서 이름을 정한다. `{ add }`는 객체 구조 분해와 닮았지만 import 선언의 문법이며, 실행 시 일반 객체를 구조 분해하는 코드가 아니다.

`Node.js의` 상대 ESM 경로는 확장자까지 명시한다. 브라우저에서는 `<script type="module">`과 URL 로딩 규칙을 사용한다. 같은 소스 파일이라도 호스트의 경로 해석·전송 방식은 다를 수 있다.

### 가져올 이름과 모듈 주소는 다른 검사다

`import { add } from "./math.mjs"`에서는 먼저 어떤 모듈인지 해석하고, 그 모듈이 add라는 export를 제공하는지 연결해야 한다. 파일이 없을 때와 파일은 있지만 export 이름이 없을 때는 실패 원인이 다르다. default export와 named export를 임의로 바꾸어 문제를 해결하려 해서는 안 된다.

`import * as math`는 모듈의 공개 이름을 네임스페이스로 모아 사용한다. 이는 원하는 객체를 생성해 내부 필드를 자유롭게 교체하는 일반적인 객체 리터럴과 다른 목적이다. 가져온 이름의 재할당 금지와, export한 객체 내부의 변경 가능성도 따로 판단한다.

브라우저의 bare specifier 처리에는 import map 같은 호스트 설정이 필요할 수 있다. `Node.js나` 번들러에서 동작한 패키지 이름이 브라우저의 기본 모듈 로더에서 똑같이 해석된다고 가정하지 않는다.

## 4. 로딩·연결·평가와 live binding

ESM은 의존 관계를 읽고 모듈들을 연결한 뒤 본문을 평가한다. 일반적인 정적 `import`는 함수·조건문 안에서 실행하는 호출문이 아니다. 모듈은 기본적으로 자체 범위와 strict mode를 갖는다.

```javascript
// counter.mjs
export let count = 0;
export function increment() { count += 1; }
```

```javascript
// use-counter.mjs — node use-counter.mjs
import { count, increment } from "./counter.mjs";
console.log(count); // 0
increment();
console.log(count); // 1
```

가져온 이름은 내보낸 바인딩과 연결되는 **live binding**이다. 처음 값을 단순 복사한 상수가 아니다. 가져오는 쪽에서 `count = 9`처럼 재할당할 수는 없지만, 원래 모듈의 변경은 관찰된다. import한 객체의 내부 변경 가능성은 바인딩 재할당 금지와 별개다.

순환 의존 자체가 항상 금지되지는 않는다. 다만 연결된 상대 바인딩이 초기화되기 전에 읽으면 실패할 수 있다. 순환이 복잡해지면 공통 기능을 별도의 모듈로 옮겨 의존 방향을 단순화한다.

### 연결과 평가는 같은 단계가 아니다

ESM은 의존 관계를 해석하고 필요한 바인딩을 연결한 다음 본문을 평가한다. 연결 과정에서 존재하지 않는 export를 발견할 수 있고, 평가 과정에서는 모듈의 최상위 코드가 실제 효과를 일으킨다. import 선언이 소스 아래쪽에 있다고 그 위의 최상위 코드가 의존 모듈 초기화보다 먼저 실행된다고 이해하면 안 된다.

live binding은 매번 모듈을 다시 실행한다는 뜻이 아니다. 동일하게 해석된 모듈의 export 바인딩을 읽는 연결이다. 모듈 최상위에 둔 상태는 여러 import 사용자가 공유할 수 있으므로 테스트·사용자별로 반드시 독립 상태가 필요한 경우에는 생성 함수를 export하는 설계도 비교한다.

이 구분은 [ECMAScript의 모듈 레코드와 연결·평가 절차](https://tc39.es/ecma262/multipage/ecmascript-language-scripts-and-modules.html#sec-source-text-module-records)를 기준으로 한다. 네트워크에서 소스를 가져오는 방법과 경로 해석은 호스트 규칙이 더해진다.

## 5. CommonJS의 실행 흐름

```javascript
// math.cjs
function add(a, b) { return a + b; }
module.exports = { add };
```

```javascript
// main.cjs — node main.cjs
const math = require("./math.cjs");
console.log(math.add(2, 3)); // 5
```

`require`는 호출이며 일반적인 CJS 로딩에서는 모듈 코드를 동기적으로 평가하고 `module.exports` 값을 반환한다. `Node.js는` 모듈 파일을 감싸 `module`·`exports`·`require` 같은 이름을 제공한다. 이 이름들은 브라우저의 기본 전역 API가 아니다.

같은 해석된 파일을 다시 `require`하면 보통 캐시된 export를 받는다. 호출할 때마다 모든 본문이 새로 실행된다고 가정하지 않는다. 서로 다른 해석 경로가 항상 같은 캐시 항목이 되는 것도 아니다.

### CommonJS 캐시와 공유 상태

```javascript
// state.cjs
console.log("state evaluated");
module.exports = { count: 0 };
```

```javascript
// use-state.cjs — node use-state.cjs
const first = require("./state.cjs");
const second = require("./state.cjs");
first.count += 1;
console.log(first === second, second.count); // true 1
```

출력은 `state evaluated` 한 번 다음 `true 1`이다. 같은 해석된 파일의 캐시 항목을 재사용하기 때문에 두 호출자가 같은 객체를 받는다. require를 여러 번 했다는 사실만으로 상태가 독립적으로 초기화되지는 않는다.

캐시는 파일의 해석 결과를 기준으로 하므로 설치 경로·패키지 해석이 다르면 같은 이름을 require해도 다른 파일을 가리킬 수 있다. 라이브러리의 단일 인스턴스를 가정하는 버그에서는 실제 해석 경로도 확인한다. [Node.js CommonJS 캐시 규칙](https://nodejs.org/api/modules.html#caching) 참조.

## 6. exports와 `module.exports`

```javascript
// broken.cjs
exports = { value: 1 };
```

```javascript
// check.cjs — node check.cjs
console.log(require("./broken.cjs").value); // undefined
```

초기의 `exports`는 `module.exports`와 같은 객체를 가리키는 지역 이름이다. `exports.value = 1`은 그 객체를 수정하지만 `exports = ...`는 지역 이름의 연결만 바꾼다. export 전체를 교체하려면 `module.exports = ...`를 쓴다. 이 차이는 객체 공유와 재할당의 차이와 연결된다.

CJS export 객체를 구조 분해한 지역 변수는 일반적인 값 대입이다. ESM import의 live binding과 같은 규칙이라고 추측하지 않는다.

### 바깥 객체 교체와 기존 참조

모듈 본문에서 `exports.name = ...`은 초기 export 객체를 수정한다. `module.exports = ...`은 최종 내보낼 연결을 바꾼다. 이후 예전 객체를 가리키는 지역 별칭을 수정해도 새로 내보낸 객체가 바뀌지는 않을 수 있다. 대입의 대상이 어떤 객체인지 그림으로 확인해야 한다.

또한 `const { count } = require(...)`는 CJS 객체의 속성을 읽어 지역 변수에 저장하는 일반적인 구조 분해다. 이후 export 객체의 count가 바뀌어도 그 지역 숫자가 자동 갱신되지 않는다. 객체 전체를 보관하고 실행 시 속성을 읽는 코드와 비교하면 live binding과의 차이가 드러난다.

## 7. 환경에 따른 비교와 주의점

| 기준 | ESM | Node.js CommonJS |
| --- | --- | --- |
| 주요 문법 | `import` / `export` | `require` / `module.exports` |
| 형식 명시 예 | `.mjs`, `type: module` 패키지의 `.js` | `.cjs`, `type: commonjs` 패키지의 `.js` |
| 기본 실행 규칙 | 모듈 범위·strict mode | 파일별 래퍼 함수 |
| 최상위 `this` | `undefined` | 보통 `module.exports` |
| 공개 값 연결 | import 바인딩 연결 | export로 반환된 값 사용 |

`.js`의 형식은 패키지 설정과 실행 조건에 좌우되므로 확장자 하나만 보고 단정하지 않는다. ESM과 CJS의 상호 운용도 Node.js 버전·대상 모듈의 조건에 따라 달라진다. “CJS는 어떤 ESM도 require할 수 없다”처럼 오래된 절대 규칙으로 외우지 말고 현재 런타임 문서를 확인한다.

### 상호 운용과 순환 의존의 한계

현대 `Node.js는` 일부 조건에서 CJS의 require로 ESM을 읽는 경로를 지원하지만, 대상 그래프에 최상위 await가 있는지 등 제약과 버전 차이가 있다. 이 문서는 한쪽은 항상 동기·다른 쪽은 항상 비동기라는 두 문장으로 상호 운용을 단정하지 않는다. 배포할 Node 버전과 [공식 상호 운용 문서](https://nodejs.org/api/esm.html#interoperability-with-commonjs)의 적용 조건을 확인한다.

순환 의존은 A가 B를, B가 A를 필요로 하는 관계다. CJS에서는 평가 중인 모듈의 아직 완성되지 않은 export를 관찰할 수 있고, ESM에서는 연결된 바인딩을 초기화 전에 읽어 오류가 날 수 있다. 공통 상수를 제3 모듈로 옮기거나 의존을 인자로 넘겨 방향을 단순화하면 초기화 순서를 이해하기 쉬워진다.

## 8. 실무 적용과 디버깅 관점

`require is not defined`, `Cannot use import statement outside a module`, export 이름 부재 오류가 나면 먼저 실행한 파일, 확장자, 가장 가까운 `package.json`의 `type`, 실제 명령을 기록한다. 문법을 마구 바꾸기 전에 어느 모듈 로더가 파일을 해석했는지 확인해야 한다.

모듈 공개 인터페이스는 작게 유지한다. 내부 상태를 모두 공개하면 외부 코드가 초기화 순서에 의존하기 쉽다. 외부 통신·화면 접근 같은 호스트 기능과 순수한 계산 모듈을 나누면 환경별 재사용도 쉬워진다.

**[미리보기: import와 비동기]** `import()`는 필요할 때 모듈을 가져오는 표현식이며 Promise를 반환한다. 반환값과 처리 흐름은 Async JavaScript의 Promise 학습 뒤에 연결한다.

### 모듈 오류를 네 단계로 좁힌다

오류가 나면 첫째 형식 판별을 확인한다. 같은 `.js`라도 가장 가까운 `package.json` 설정과 실행 방식의 영향을 받는다. 둘째 실제로 해석한 파일·URL을 확인한다. 셋째 그 파일이 제공하는 export 이름과 가져오는 문법을 맞춘다. 넷째 본문 초기화 중 실패인지 살핀다.

번들러가 코드를 변환했다면 원본 문법과 배포된 코드의 모듈 체계가 다를 수 있다. 실행 환경에서 지원하는 문법과 빌드 도구가 대신 처리하는 문법을 나누어 기록한다. 재현은 프로젝트 전체를 바꾸기보다 확장자를 명시한 두 파일에서 시작하고 필요한 설정을 하나씩 추가하는 편이 원인을 분리하기 쉽다.

### 통합 사례: 공유 모듈 상태와 호출별 상태

모듈을 import할 때마다 새 장바구니가 생길 것으로 기대했는데 모든 사용자가 같은 객체를 보는 문제를 생각하자. 원인은 모듈 최상위에서 객체 하나를 만들어 내보내고 그 객체를 여러 호출자가 공유했기 때문일 수 있다. 모듈 재사용과 기능 인스턴스 생성을 분리해야 한다.

```javascript
// carts.mjs
export const sharedCart = { count: 0 };
export function createCart() { return { count: 0 }; }
```

```javascript
// use-carts.mjs — node use-carts.mjs
import { sharedCart, createCart } from "./carts.mjs";
const a = sharedCart;
const b = sharedCart;
a.count += 1;
const first = createCart();
const second = createCart();
first.count += 1;
console.log(b.count, second.count); // 1 0
```

첫 경로는 하나의 export 객체를 공유하고, 둘째 경로는 공개한 생성 함수를 호출할 때마다 새 객체를 만든다. 두 경우 모두 모듈 기능은 재사용하지만 상태의 생성 위치가 다르다. CJS에서도 하나의 export 객체를 재사용하는지 함수를 호출해 새 값을 만드는지 같은 질문이 필요하다.

이 문제를 캐시 삭제로만 해결하려고 하면 모듈 초기화 효과와 의존 관계까지 다시 실행될 수 있다. 먼저 상태의 원하는 수명을 정하고 그 경계에서 생성하도록 인터페이스를 고친다. 모듈 로더의 재사용 규칙은 유지하면서 기능별 상태를 분리할 수 있는 설계를 선택하는 것이다.

## 9. 핵심 정리

모듈은 파일과 공개 인터페이스의 경계다. ESM import는 바인딩을 연결하고, CommonJS는 `module.exports`를 반환한다. 실행 환경과 모듈 형식을 먼저 확인해야 로딩 오류와 코드 오류를 구분할 수 있다.

## 출처

- [ECMAScript — Modules](https://tc39.es/ecma262/multipage/ecmascript-language-scripts-and-modules.html#sec-modules): 연결·평가·import/export.
- [Node.js — ECMAScript modules](https://nodejs.org/api/esm.html): 형식 명시·확장자·상호 운용.
- [Node.js — CommonJS modules](https://nodejs.org/api/modules.html): 래퍼·캐시·exports와 `module.exports`.

## 10. 이해도 확인 질문

1. named export와 default export를 가져올 때 이름을 정하는 규칙은 어떻게 다른가?
2. export한 `let` 값이 원래 모듈에서 바뀔 때 import한 쪽에 보이는 이유를 설명하라.
3. `exports = { value: 1 }`로 내보내기가 되지 않는 이유와 수정 방법은 무엇인가?
4. 동일한 `.js` 파일이 다른 프로젝트에서 로딩 오류를 내면 어떤 설정과 명령을 확인해야 하는가?
5. CJS export를 구조 분해한 변수와 ESM import 바인딩을 같은 것으로 보면 어떤 문제가 생길 수 있는가?
