---
track: A
notion: https://app.notion.com/p/415178c4afac8166b1e5fe0ebe8560d5
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · this · call / apply / bind

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 일반 함수의 호출 표현식에서 `this`를 판별한다.
- `call`·`apply`·`bind`의 실행 시점과 인자 전달을 비교한다.
- 화살표 함수의 어휘적 `this`와 명시적 바인딩의 한계를 설명한다.
- 메서드 분리와 생성자 호출의 예외를 디버깅한다.

## 2. 이 주제를 왜 배우는가

`user.read()`는 동작하는데 `const read = user.read; read()`는 실패할 수 있다. 함수 본문은 같은데 호출 표현식이 달라졌기 때문이다. `this`를 함수가 속한 객체라고만 외우면 이 차이를 설명하지 못한다.

**`this`** 는 함수 실행에서 사용할 문맥 값이다. 일반 함수는 주로 호출 방식으로 결정하고, 화살표 함수는 자신의 `this` 바인딩을 만들지 않아 바깥 것을 사용한다. 객체 안에 작성했다는 위치만으로 고정되지 않는다.

### 코드 위치와 실행 문맥을 따로 표시한다

객체에 함수를 저장해도 그 함수가 그 객체만의 전용 함수가 되는 것은 아니다. 여러 객체가 같은 함수 값을 공유할 수 있고, 외부 코드가 그 함수를 꺼내 부를 수도 있다. this는 이런 호출마다 어느 수신자의 데이터를 사용할지 연결하는 방식이다.

앞서 배운 렉시컬 스코프는 일반 변수 이름을 정의 위치로 해석하는 규칙이다. 일반 함수의 this를 그 규칙과 똑같이 읽으면 호출 방식이 바뀌는 사례를 설명할 수 없다. 단, 화살표 함수는 자신만의 this 바인딩을 만들지 않아 바깥 연결을 사용한다.

## 3. 일반 함수의 호출 규칙

```javascript
"use strict";
function read() { return this; }
const first = { read };
const second = { read };
console.log(first.read() === first); // true
console.log(second.read() === second); // true
const detached = first.read;
console.log(detached()); // undefined
```

`first.read()`의 수신자는 `first`, `second.read()`의 수신자는 `second`다. 함수 자체가 달라진 것이 아니라 같은 함수를 다른 수신자로 호출했다. 분리한 함수를 단독 호출하면 strict mode에서 `this`는 `undefined`다.

비엄격 일반 함수의 단독 호출에서는 `null`·`undefined` 문맥이 전역 객체로 대체되고 원시 값 문맥은 객체로 감싸질 수 있다. 아래 예제들은 이러한 암묵적 보정을 피하기 위해 strict mode를 명시한다. ESM은 기본적으로 strict mode다.

### 같은 본문을 다른 수신자로 호출한다

```javascript
"use strict";
function getName() { return this.name; }
const a = { name: "A", getName };
const b = { name: "B", getName: a.getName };
console.log(a.getName(), b.getName()); // A B
console.log(a.getName === b.getName); // true
```

두 메서드는 같은 함수 값이지만 호출 표현식에서 수신자가 다르다. 함수가 어느 객체의 속성에서 처음 발견됐는지보다 실제 호출 때의 기준 객체가 중요하다. `a.getName`을 읽는 것과 `a.getName()`을 실행하는 것도 구분한다.

strict mode는 단독 호출의 this를 전역 객체로 자동 보정하지 않는다. 따라서 메서드를 꺼내 잘못 호출하면 오류가 일찍 드러날 수 있다. 전역 객체에 우연히 같은 속성이 있어 버그를 숨기는 비엄격 코드와 다른 점이다.

## 4. call과 apply: 지금 호출하기

```javascript
"use strict";
function total(extra, tax) {
  return this.price + extra + tax;
}
const item = { price: 100 };
console.log(total.call(item, 20, 5)); // 125
console.log(total.apply(item, [20, 5])); // 125
```

둘 다 첫 인자를 `this`로 사용하여 함수를 즉시 실행한다. `call`은 이어지는 개별 인자를 전달하고, `apply`는 배열 또는 배열 유사 객체로 인자를 공급한다. `apply`가 원본 함수를 영구적으로 수정하는 것은 아니다.

`call`로 함수를 한 번 실행했다고 이후 단독 호출의 `this`까지 바뀌지는 않는다. 실행 결과와 호출 규칙을 분리한다.

### strict 함수에 전달한 원시 문맥

```javascript
"use strict";
function inspect(a, b) {
  return [this === null, typeof this, a + b].join(",");
}
console.log(inspect.call(null, 2, 3)); // true,object,5
console.log(inspect.apply(7, [2, 3])); // false,number,5
```

strict 일반 함수에서는 명시한 null이나 원시 값이 그대로 this가 될 수 있다. 위 첫 결과의 typeof null이 object인 점은 this의 객체 변환 때문이 아니라 typeof null의 별도 규칙이다. 비엄격 함수에서는 null·undefined의 전역 대체와 원시 값의 객체 변환 규칙이 더해질 수 있다.

[ECMAScript OrdinaryCallBindThis](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinarycallbindthis)는 함수의 this 모드에 따라 이 차이를 처리한다. 호출자 파일이 strict인지뿐 아니라 실제 호출되는 함수의 규칙을 확인한다.

## 5. bind: 나중에 호출할 함수 만들기

```javascript
"use strict";
function total(extra, tax) {
  return this.price + extra + tax;
}
const item = { price: 100 };
const withExtra = total.bind(item, 20);
console.log(typeof withExtra); // function
console.log(withExtra(5)); // 125
console.log(withExtra.call({ price: 1000 }, 5)); // 125
```

`bind`는 즉시 계산하지 않고 **바인딩된 새 함수**를 반환한다. `this`와 앞부분 인자를 보관하며 나중에 추가 인자를 이어 받는다. 일반 호출에서 이미 바인딩된 `this`는 다시 `call`해도 교체되지 않는다.

`item.price`는 호출 때 읽는다. `bind`가 객체 내부의 속성 값까지 스냅샷으로 복사하는 것은 아니다. `bind`를 호출할 때마다 새 함수이므로 함수를 보관·해제하는 API에서는 같은 참조를 재사용해야 할 수 있다.

### 부분 적용과 객체의 현재 상태

```javascript
"use strict";
function describe(prefix, suffix) { return prefix + this.name + suffix; }
const user = { name: "Kim" };
const label = describe.bind(user, "[");
user.name = "Lee";
console.log(label("]")); // [Lee]
console.log(label === describe.bind(user, "[")); // false
```

bind는 앞부분 인자를 보관하는 **부분 적용(partial application)** 에도 사용할 수 있다. 그러나 `user.name의` 값을 생성 시점에 복사하지 않고 보관한 객체를 호출 시 읽으므로 Lee가 나온다. 첫 인자에 객체를 바인딩하는 것과 그 객체의 필드를 스냅샷으로 저장하는 것은 다르다.

같은 인자로 bind해도 매번 새 함수다. 등록·해제 API가 함수 참조의 동일성을 요구한다면 bind 결과를 변수에 보관해야 한다. bind는 원본 함수를 수정하는 명령이 아니라 연결 정보를 가진 다른 함수를 만드는 연산이다.

## 6. 화살표 함수의 this

```javascript
"use strict";
const user = {
  name: "Kim",
  makeReader() {
    return () => this.name;
  }
};
const reader = user.makeReader();
console.log(reader()); // Kim
console.log(reader.call({ name: "Lee" })); // Kim
```

`makeReader`는 `user`를 수신자로 실행된다. 그 안에서 만든 화살표 함수는 바깥 호출의 `this`를 사용한다. 화살표 함수에 `call`·`apply`·`bind`를 적용해도 자신의 `this`를 새로 만들거나 교체하지 않는다.

객체 리터럴 자체는 별도의 `this` 범위를 만들지 않는다. 따라서 `const user = { read: () => this.name }`에서 화살표의 `this`는 `user`가 아니라 그 객체 리터럴이 놓인 바깥 환경을 따른다. 최상위 `this`는 브라우저 스크립트·ESM·CJS에서 다르므로 출력값도 환경별로 달라질 수 있다.

### 바깥 this도 어떤 호출이었는지 확인한다

```javascript
"use strict";
function makeReader() {
  return () => this.name;
}
const a = makeReader.call({ name: "A" });
const b = makeReader.call({ name: "B" });
console.log(a(), b(), a.call({ name: "C" })); // A B A
```

화살표 함수 두 개는 각 makeReader 호출의 this를 사용한다. “화살표의 this는 언제나 전역”도, “언제나 객체 리터럴 자신”도 아니다. 생성되는 바깥 코드의 this 해석을 한 번 더 추적해야 한다.

call·apply·bind로 화살표의 this를 바꿀 수 없어도 인자 전달·부분 적용의 효과까지 전부 없다는 뜻은 아니다. 무엇을 바꾸려는지 this와 일반 인자를 나누어 해석한다.

## 7. 생성자 호출과 우선순위의 한계

`new`는 생성 가능한 함수를 사용해 새 객체를 만들고 해당 생성 과정의 `this`를 설정한다. 바인딩된 생성 가능 함수를 `new`로 호출하면 바인딩해 둔 `this`보다 새 인스턴스가 사용된다. 따라서 “bind가 어떤 상황에서도 무조건 이긴다”는 우선순위 암기는 부정확하다.

**[미리보기: Prototype · Class]** 생성자 호출은 새 객체와 프로토타입 연결도 준비한다. 인스턴스 생성과 클래스 메서드 배치는 이후 Prototype · Prototype Chain과 Class에서 다룬다.

화살표 함수는 생성 가능 함수가 아니므로 `new`로 호출할 수 없다. 인자 전달을 묶고 싶은 것과 객체 생성이 필요한 것은 다른 요구다.

### new와 bind가 만나는 경우

```javascript
function Item(name) { this.name = name; }
const existing = { name: "old" };
const BoundItem = Item.bind(existing, "new");
const made = new BoundItem();
console.log(made.name, existing.name); // new old
console.log(made instanceof Item); // true
```

생성 가능한 원본 함수를 bind한 뒤 new로 호출하면 새 인스턴스를 준비한다. 바인딩된 existing은 생성 결과의 this로 사용하지 않지만 미리 바인딩한 일반 인자 name은 전달된다. this 규칙과 인자 결합 규칙을 별도로 읽어야 하는 사례다.

화살표 함수처럼 원본이 생성 가능하지 않으면 bind한다고 생성자가 되지 않는다. 새 객체 생성의 전체 원리는 후속 Prototype과 Class에서 연결한다.

## 8. 흔한 오해·예외·주의점

| 오해 | 확인할 사실 |
| --- | --- |
| 함수는 선언된 객체를 기억한다 | 일반 함수는 호출 수신자를 확인 |
| `bind`는 바로 실행한다 | 새 함수 반환, 호출은 나중 |
| `call`하면 영구적으로 고정된다 | 해당 호출에만 문맥 지정 |
| 화살표는 언제나 메서드에 적합하다 | 자기 객체를 수신자로 쓰려면 일반 메서드가 자연스러움 |
| `const f = obj.f`도 메서드 호출이다 | `f()`에서는 수신자 정보가 사라짐 |

### 규칙을 한 줄의 우선순위로 줄이는 한계

“new > bind > call > 메서드” 같은 표는 일부 예제를 외우는 데만 도움이 된다. 실제로는 호출 대상이 일반 함수인지 화살표인지, 이미 bound 함수인지, 생성 가능 함수인지에 따라 적용할 규칙부터 달라진다. 함수 종류를 먼저 고르고 해당 호출 절차를 따라가야 한다.

또한 `obj.method()`라는 모양이어도 method가 이미 바인딩된 함수라면 obj를 새로운 this로 쓰지 않을 수 있다. 점 표기만 보고 수신자를 확정하지 말고 함수 값의 생성 경로까지 확인한다.

## 9. 실무 적용과 디버깅 관점

`this` 오류는 정의부보다 호출 줄부터 본다. 점 표기 수신자가 있는지, 함수가 분리되었는지, `bind`된 함수인지, 화살표인지 순서대로 확인한다. 객체 메서드를 다른 코드에 넘길 때는 `obj.method.bind(obj)`로 고정할지 `() => obj.method()`로 감쌀지 선택한다.

후자의 래퍼는 실행 시점에 `obj.method`를 다시 조회한다. `bind`는 생성 시점의 함수 객체를 사용한다. 나중에 메서드를 교체할 수 있는 코드라면 이 차이가 중요하다.

### 메서드 소실을 찾는 순서

실패한 함수의 본문에서 this의 실제 값과 필요한 속성을 확인한다. 다음으로 호출 줄을 보고, 그 함수 값이 변수 대입·구조 분해·다른 API 전달을 거쳤는지 추적한다. 마지막으로 this를 고정할 계약인지 호출자에 맡길 계약인지 정한다.

고정이 목적이라면 bind 결과를 보관하고, 실행 시 최신 메서드를 다시 조회하는 것이 목적이라면 래퍼를 고려한다. 단순히 화살표로 바꾸어 오류를 없애면 객체 리터럴 바깥의 엉뚱한 this를 사용할 수도 있으므로 바깥 환경까지 함께 설명할 수 있어야 한다.

### 통합 사례: 함수 전달 방식 세 가지 비교

다른 코드에 읽기 작업을 맡길 때 메서드 자체, bind 결과, 래퍼 함수를 전달할 수 있다. 지금 보이는 결과가 같아도 나중에 객체의 메서드가 바뀌면 차이가 드러난다.

```javascript
"use strict";
const user = { name: "Kim", read() { return this.name; } };
const bound = user.read.bind(user);
const wrapped = () => user.read();
user.read = function () { return "updated:" + this.name; };
console.log(bound(), wrapped()); // Kim updated:Kim
```

bound는 생성할 때의 함수 객체와 user 문맥을 보관한다. wrapped는 실행할 때 `user.read를` 다시 조회하므로 교체된 메서드를 사용한다. 둘 다 `user.name의` 현재 값은 호출 시점에 읽을 수 있지만, 어느 함수 본문을 실행하는지는 다르다.

여기서 raw 메서드를 꺼내 단독으로 호출하는 세 번째 방식은 strict mode의 this를 잃을 수 있다. 따라서 요구를 “객체를 유지” 하나로 줄이지 말고 수신자 유지·함수 본문 고정·실행 시 재조회 중 무엇이 필요한지 정한다.

검증에서는 메서드 교체 전후, 객체 속성 갱신 전후, 전달된 함수 참조의 동일성을 각각 확인한다. 호출 규칙을 정확히 정하면 짧은 문법을 고르는 것보다 오래 유지되는 함수 전달 코드를 예측하기 쉬워진다.

## 10. 핵심 정리

일반 함수의 `this`는 호출 표현식으로 읽는다. `call`·`apply`는 즉시 호출하고 `bind`는 나중에 호출할 함수를 만든다. 화살표의 `this`는 바깥 환경에서 오며 생성자 호출에는 별도 규칙이 있다.

## 출처

- [MDN — this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this): 수신자·strict mode·화살표 함수.
- [ECMAScript — Function.prototype methods](https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-properties-of-the-function-prototype-object): call·apply·bind.
- [MDN — Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions): 어휘적 this와 생성자 제약.

- [ECMAScript — OrdinaryCallBindThis](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinarycallbindthis): strict·비엄격·어휘적 this 모드.

## 11. 이해도 확인 질문

1. strict mode에서 `obj.read()`와 분리한 `const f = obj.read; f()`의 `this`를 비교하라.
2. 같은 함수를 `call`, `apply`, `bind`로 사용하면 실행 시점과 인자 형식이 어떻게 다른가?
3. `bind`한 함수를 다른 객체로 `call`해도 원래 문맥을 읽는 이유는 무엇인가?
4. 객체 리터럴 안의 화살표가 자동으로 그 객체를 `this`로 삼지 않는 이유를 설명하라.
5. 메서드의 bind 결과와 `() => obj.method()` 래퍼는 나중에 메서드가 교체되면 어떻게 달라질 수 있는가?
