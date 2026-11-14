---
track: A
notion: https://app.notion.com/p/415178c4afac811b8bbfd5c4b122676b
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · Scope · Lexical Scope · Scope Chain

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 전역·모듈·함수·블록 범위에서 이름을 해석한다.
- 렉시컬 스코프와 호출 위치의 차이를 설명한다.
- 섀도잉과 스코프 체인을 이용해 실행 결과를 예측한다.
- 같은 이름의 변수가 다른 값을 보이는 버그를 추적한다.

## 2. 이 주제를 왜 배우는가

다른 함수에서 같은 이름의 변수를 선언했다고 현재 함수의 값이 바뀌지는 않는다. 이 규칙이 없다면 함수를 호출한 곳의 지역 변수에 따라 결과가 무작위로 달라질 것이다. 스코프는 이름과 값의 연결을 어느 코드 영역에서 사용할 수 있는지 정한다.

**스코프(scope)** 는 식별자를 해석하는 범위다. JavaScript는 소스 코드의 중첩 구조를 바탕으로 바깥 이름을 찾는 **렉시컬 스코프(lexical scope)** 를 사용한다. 함수를 호출한 위치가 아니라 정의한 위치가 바깥 환경을 결정한다.

### 이름 충돌을 통제하는 이유

같은 name을 서로 다른 기능에서 사용할 수 있어야 작은 함수를 조합하기 쉽다. 지역 범위는 이름의 영향 영역을 줄인다. 모든 값을 전역에 두면 한 기능의 대입이 다른 기능의 입력을 바꾸어 재현 순서에 따라 결과가 달라질 수 있다.

스코프는 접근 권한 전체나 객체 소유권을 뜻하지는 않는다. 현재 이름을 찾을 수 있더라도 그 값이 가리키는 객체를 다른 코드도 공유할 수 있다. 이름 검색과 데이터 공유를 분리해서 보면 지역 변수를 썼는데 외부 객체가 바뀌는 문제도 설명할 수 있다.

## 3. 범위의 종류와 선언

| 범위 | 예 | 이름을 사용할 수 있는 영역 |
| --- | --- | --- |
| 전역 | 일반 스크립트의 최상위 | 해당 전역 환경의 코드 |
| 모듈 | ESM 최상위 | 모듈 내부와 명시적 export/import 경계 |
| 함수 | 매개변수·함수 안 `var` | 해당 함수 호출의 범위 |
| 블록 | `{ ... }` 안 `let`·`const` | 해당 블록 내부 |

```javascript
function show() {
  if (true) {
    var visible = "function";
    let local = "block";
    console.log(local); // block
  }
  console.log(visible); // function
  // console.log(local); // 별도 실행 시 ReferenceError
}
show();
```

중괄호가 항상 모든 선언에 같은 범위를 만드는 것은 아니다. `var`는 일반적인 블록을 넘어 함수 범위에 속하고, `let`·`const`는 블록 범위를 따른다. 객체 리터럴의 중괄호는 변수 선언용 블록과도 다르다.

### 중괄호의 역할을 문법으로 구별한다

```javascript
function example() {
  const record = { value: 1 };
  if (record.value === 1) {
    let message = "inside";
    console.log(message); // inside
  }
  console.log(record.value); // 1
}
example();
```

객체 리터럴의 중괄호는 속성들을 묶고, if의 중괄호는 문장 블록을 만든다. 중괄호가 보인다는 이유만으로 동일한 변수 범위를 만드는 것은 아니다. 함수 매개변수·함수 본문·블록 선언은 각각 어떤 이름을 도입하는지 읽어야 한다.

반복문·조건문 안에서 선언한 `var`가 함수 범위를 따른다는 점도 여기에 연결된다. 실행이 조건문 안을 지났는지와 이름이 함수 어디에서 해석되는지는 별도의 규칙이다.

## 4. 스코프 체인의 이름 검색

현재 환경에 이름이 없으면 바깥 환경으로 연결을 따라 찾는다. 이 연결을 **스코프 체인(scope chain)** 이라고 부른다. 가장 가까운 같은 이름의 바인딩이 선택되고, 끝까지 없으면 미해결 이름이 된다.

```javascript
const currency = "KRW";
function invoice() {
  const tax = 10;
  function line(price) {
    return price + tax + " " + currency;
  }
  return line(100);
}
console.log(invoice()); // 110 KRW
```

```text
line의 지역 환경: price
→ invoice의 환경: tax
→ 바깥 환경: currency
```

`price`, `tax`, `currency`의 검색 경로는 서로 다르다. `line`을 `invoice` 밖에서 직접 호출하는 것은 이름이 보이지 않아 불가능하지만, 함수 값을 다른 곳으로 전달하는 것과는 별개의 문제다.

### 검색은 가장 가까운 선언에서 멈춘다

```javascript
const value = "global";
function outer() {
  const value = "outer";
  function inner() {
    const local = "inner";
    return local + ":" + value;
  }
  return inner();
}
console.log(outer()); // inner:outer
```

inner는 local을 현재 환경에서 찾고, value는 outer의 환경에서 찾는다. 그곳에 같은 이름이 있으므로 더 바깥의 global value까지 계속 검색하지 않는다. 바깥 이름을 전부 한꺼번에 합친 뒤 아무 값이나 고르는 방식이 아니다.

명세의 [ResolveBinding](https://tc39.es/ecma262/multipage/executable-code-and-execution-contexts.html#sec-resolvebinding)은 지정한 환경 또는 현재 어휘 환경에서 이름을 해석하는 출발점을 정한다. 환경을 따라가며 바인딩을 찾는 과정과 객체의 속성을 읽는 과정은 서로 다른 연산이다.

## 5. 호출 경로와 렉시컬 경로

```javascript
const label = "outer";
function read() {
  return label;
}
function caller() {
  const label = "caller";
  return read();
}
console.log(caller()); // outer
```

실제 호출 경로는 최상위 → `caller` → `read`다. 그러나 `read`의 바깥 환경은 소스상 정의된 최상위 환경이다. `caller`의 지역 `label`은 `read`의 바깥 환경에 끼어들지 않는다.

호출 스택은 누가 누구를 불렀는지를, 스코프 체인은 이름을 어디서 찾는지를 설명한다. 둘을 같은 화살표로 그리면 이 예제에서 잘못된 값을 예측하게 된다. [렉시컬 스코프 설명](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures#lexical_scoping) 참조.

### 함수의 전달도 정의 위치를 바꾸지 않는다

```javascript
const label = "definition";
function read() { return label; }
function use(fn) {
  const label = "call site";
  return fn();
}
console.log(use(read)); // definition
```

함수 값을 매개변수로 넘겨 호출해도 원래 함수의 정의 환경은 use의 지역 환경으로 교체되지 않는다. 호출 위치를 바꾸는 것과 소스상 중첩 위치를 바꾸는 것은 다른 변경이다.

반대로 use 내부에 새 함수를 직접 정의해 그 안에서 label을 읽는다면 정의 위치가 달라진다. 이 차이는 함수를 다른 파일로 이동하거나 공통 함수로 추출할 때 중요하다. 암묵적으로 사용하던 바깥 이름이 더는 보이지 않는다면 필요한 값을 인자로 명시하는 설계를 고려한다.

## 6. 섀도잉과 이름의 경계

**섀도잉(shadowing)** 은 안쪽 범위의 이름이 바깥의 같은 이름을 가리는 것이다. 바깥 값이 삭제되거나 바뀌었다는 뜻은 아니다.

```javascript
const status = "idle";
{
  const status = "loading";
  console.log(status); // loading
}
console.log(status); // idle
```

같은 범위의 `let`·`const` 중복 선언은 오류지만 서로 다른 중첩 범위의 같은 이름은 가능하다. 가능하다는 이유만으로 모든 변수 이름을 재사용할 필요는 없다. 바깥 `user`를 안쪽에서 또 다른 `user`로 가리면 디버깅 시 어느 바인딩인지 혼동하기 쉽다.

**[미리보기: Hoisting · TDZ]** 안쪽의 같은 이름은 초기화 전에도 바깥 이름을 가릴 수 있다. 선언 전 접근이 왜 실패하는지는 다음 Hoisting · TDZ에서 다룬다.

### 섀도잉은 바깥 값의 덮어쓰기가 아니다

```javascript
let result = "outer";
function update() {
  let result = "inner";
  result = "changed inner";
  return result;
}
console.log(update(), result); // changed inner outer
```

같은 글자의 이름이지만 함수 안 대입은 안쪽 바인딩을 갱신한다. 바깥 값을 바꾸려던 코드라면 안쪽 선언이 문제일 수 있고, 지역 결과만 만들려던 코드라면 정상이다. 이름이 같다는 이유만으로 변경이 전파된다고 가정하지 않는다.

반대로 안쪽에 선언이 없으면 대입이 바깥에서 찾은 바인딩을 갱신할 수 있다. strict mode에서 선언되지 않은 이름에 대입하면 오류가 되므로 암묵적 전역을 만들려는 코드에 기대지 않는다. 코드 검토에서는 읽는 이름뿐 아니라 대입 대상 이름의 선언 위치도 찾는다.

## 7. 전역·모듈과 객체 속성의 차이

브라우저 일반 스크립트의 최상위 `var`와 `let`·`const`는 전역 객체 속성과의 관계가 다르다. ESM 최상위 선언은 모듈 범위에 속한다. Node.js CommonJS도 파일 래퍼가 있으므로 최상위 선언을 곧 전역 변수로 읽으면 안 된다.

객체의 `name` 속성과 식별자 `name`도 다르다. `user.name`은 `user`를 스코프로 찾은 뒤 객체에서 속성을 읽는다. 속성을 찾는 과정이 변수 스코프 체인을 따라 같은 이름을 검색하는 과정은 아니다.

### 객체 속성 조회는 스코프 체인의 다음 단계가 아니다

```javascript
const name = "variable";
const user = { name: "property" };
const { name: extracted } = user;
console.log(name, user.name, extracted); // variable property property
```

첫 이름은 변수 검색으로, 두 번째는 user라는 변수를 찾은 뒤 속성 조회로, 세 번째는 구조 분해로 만들어진 지역 이름을 읽는다. 각 단계의 데이터 경로가 다르다. 객체 속성 이름과 변수 이름의 철자가 같아도 하나의 바인딩이 아니다.

브라우저 일반 스크립트에서는 일부 전역 이름과 전역 객체 속성이 연결되지만 모든 선언에 동일하게 적용되지는 않는다. ESM 또는 CJS에서 같은 코드를 실행했을 때 보이는 이름이 달라지면 실행 소스 형식을 먼저 확인한다.

## 8. 흔한 오해·예외·주의점

- **호출한 함수의 지역 변수를 마음대로 읽는다**: 정의 위치의 환경을 따른다.
- **블록 밖에서 안 보이면 값이 즉시 메모리에서 삭제된다**: 이름의 접근 범위와 데이터 수명은 다르다.
- **스코프 체인이 실행할 때마다 호출자 방향으로 바뀐다**: 동적 호출 경로와 구분한다.
- **모든 최상위 이름은 전역 객체 속성이다**: 실행 형식과 선언 종류를 구분한다.

### 전역 값이 편리해도 의존성이 숨을 수 있다

함수가 바깥 전역 설정을 읽으면 호출 시 인자가 같아도 그 설정이 바뀌어 결과가 달라질 수 있다. 이것이 항상 잘못은 아니지만 호출 계약에 드러나지 않는 의존성이다. 테스트가 순서에 따라 달라지면 공유 전역 바인딩의 변경을 확인한다.

일반적인 직접 코드의 이름 해석을 이해하는 단계에서 eval이나 with 같은 특수 기능을 기본 모델에 섞지 않는다. 현재 설명은 일반적인 렉시컬 코드와 모듈을 대상으로 한다. 예외적인 이름 해석 기능을 만났다면 별도 조건으로 분석해야 한다.

## 9. 실무 적용과 디버깅 관점

같은 이름인데 예상과 다른 값을 읽으면 현재 줄의 가장 가까운 선언부터 찾는다. 디버거의 Scope 화면에서 지역·바깥·모듈·전역 항목을 구분하고, Call Stack의 호출자 변수와 혼합하지 않는다. 전역 의존을 매개변수로 바꾸면 입력이 명시되어 재현이 쉬워질 수 있다.

**[미리보기: Closure]** 함수가 정의된 환경을 통해 바깥 바인딩을 계속 사용할 수 있다는 성질은 바깥 호출이 끝난 뒤에도 중요하다. 이 수명과 공유 문제는 JavaScript Core의 Closure에서 다룬다.

### 이름 검색 버그를 표로 정리한다

현재 줄에서 읽거나 쓰는 이름을 적고, 가장 가까운 선언의 소스 위치를 연결한다. 그다음 현재 지역 환경에 없는 이름만 바깥 단계로 이동시킨다. 호출자의 변수 창에서 우연히 같은 이름을 발견했다고 그 값을 사용하는 것으로 결론 내리지 않는다.

리팩터링 후 값이 달라졌다면 함수 정의를 옮기면서 바깥 환경이 바뀌었는지 확인한다. 함수 본문이 같아도 정의 위치가 달라지면 읽는 바인딩이 달라질 수 있다. 필요한 입력을 매개변수로 드러내면 이런 환경 의존을 줄일 수 있다.

### 통합 사례: 공통 함수로 옮긴 뒤 값이 달라지는 이유

내부 함수가 바깥 세율을 읽다가 공통 함수로 추출된 상황을 보자. 함수의 계산식은 비슷해도 그 함수가 정의된 위치의 환경이 다르므로 같은 이름을 다른 바인딩으로 해석할 수 있다.

```javascript
const rate = 0.1;
function shared(price) { return price * rate; }
function invoice(price) {
  const rate = 0.2;
  function local() { return price * rate; }
  return [local(), shared(price)];
}
console.log(invoice(100).join(",")); // 20,10
```

local의 price와 rate는 invoice의 환경에서 찾는다. shared의 price는 자신의 매개변수이고 rate는 최상위에서 찾는다. shared가 invoice에서 호출되었다는 사실은 invoice의 rate를 바깥 환경으로 삽입하지 않는다.

리팩터링 전과 같은 계산을 원한다면 필요한 rate를 shared의 명시적 입력으로 전달하는 설계를 고려한다. 이렇게 하면 함수가 어느 환경에 놓였는지에 대한 숨은 의존을 줄일 수 있다. 반대로 항상 전역 공통 세율을 쓰는 것이 의도라면 shared의 결과가 맞다.

검증에서는 호출 순서표와 이름 검색표를 따로 만든다. 같은 호출 경로에서도 검색 경로는 다를 수 있고, 입력을 매개변수로 명시하면 두 경로의 연결이 더 분명해진다.

## 10. 핵심 정리

스코프는 이름을 찾는 규칙이고 렉시컬 스코프는 정의 위치의 중첩 구조를 따른다. 현재 범위부터 바깥으로 찾으며 가까운 이름이 바깥 이름을 가린다. 호출 경로·객체 속성 검색·변수 이름 검색은 각각 구분한다.

## 출처

- [ECMAScript — Environment Records](https://tc39.es/ecma262/multipage/executable-code-and-execution-contexts.html#sec-environment-records): 바인딩과 바깥 환경 연결.
- [MDN — Closures: Lexical scoping](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures#lexical_scoping): 정의 위치와 함수·블록 범위.

## 11. 이해도 확인 질문

1. 최상위에서 정의한 함수를 같은 이름의 지역 변수가 있는 함수에서 호출하면 어느 변수를 읽는가?
2. 같은 블록 안의 `var`와 `let`을 블록 밖에서 읽을 때 무엇이 다른가?
3. `user.name`의 속성 검색과 식별자 `name`의 스코프 검색은 어떻게 다른가?
4. 안쪽 `const status`를 변경하지 않아도 바깥 `status`와 다른 값이 출력되는 이유는 무엇인가?
5. 호출 스택과 스코프 체인을 각각 어떤 버그 추적에 사용할 수 있는가?
