---
track: A
notion: https://app.notion.com/p/415178c4afac817bb073f1210e1bb52d
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · Class

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 클래스·생성자·인스턴스의 관계를 설명한다.
- 인스턴스 필드·프로토타입 메서드·정적 요소를 구분한다.
- 상속·super·private 필드의 실행 조건을 추적한다.
- 메서드 분리와 초기화 순서 오류를 디버깅한다.

## 2. 이 주제를 왜 배우는가

프로토타입을 직접 연결하면 객체의 동작을 공유할 수 있지만 생성과 메서드 배치가 여러 줄에 흩어진다. **클래스(class)** 는 인스턴스 생성과 공통 동작을 하나의 선언으로 표현하는 문법이다. JavaScript의 프로토타입 기반 동작 위에 있지만 단순한 함수 치환으로 모든 차이를 설명할 수는 없다.

**인스턴스(instance)** 는 보통 `new`로 클래스를 통해 만든 객체다. 클래스가 사용자라면 인스턴스는 각각의 사용자이며, 객체별 이름·잔액은 서로 분리되어야 한다.

### 생성 규칙과 사용 규칙을 함께 묶는다

서로 다른 인스턴스가 같은 초기화 규칙과 동작 계약을 가져야 할 때 클래스가 도움이 된다. 단순히 속성 몇 개가 비슷하다는 이유만으로 상속 구조를 만들 필요는 없다. 객체가 무엇을 할 수 있고 어떤 상태를 유지해야 하는지 먼저 정한다.

예를 들어 지갑이라면 잔액을 읽고 입금하는 동작과 허용 금액의 규칙이 있다. 클래스는 이 상태와 동작의 경계를 표현하지만 입력 검증이나 오류 처리가 자동으로 생기는 것은 아니다. 메서드가 보존할 조건은 직접 구현해야 한다.

## 3. 생성자·필드·메서드

```javascript
class Counter {
  value = 0;
  constructor(step) { this.step = step; }
  increment() {
    this.value += this.step;
    return this.value;
  }
}
const a = new Counter(2);
const b = new Counter(1);
console.log(a.increment(), a.increment(), b.increment()); // 2 4 1
console.log(Object.hasOwn(a, "value")); // true
console.log(Object.hasOwn(a, "increment")); // false
console.log(a.increment === b.increment); // true
```

**constructor**는 새 인스턴스를 초기화한다. `value` 필드와 `this.step`은 각 인스턴스의 자신의 상태이고, 일반적인 인스턴스 메서드 `increment`는 `Counter.prototype`에 놓여 공유된다.

```text
a { value, step } ─┐
                  ├─> Counter.prototype { increment }
b { value, step } ─┘
```

필드 초기값으로 배열·객체를 만들면 인스턴스마다 새로 평가된다. 프로토타입에 하나의 배열을 공유해 둔 경우와 다르다.

### 인스턴스 필드의 초기값은 매번 평가된다

```javascript
class Bag {
  items = [];
  add(value) { this.items.push(value); }
}
const a = new Bag();
const b = new Bag();
a.add("pen");
console.log(a.items.join(","), b.items.length); // pen 0
console.log(a.items === b.items, a.add === b.add); // false true
```

필드의 `[]`는 각 인스턴스를 초기화할 때 새로 평가된다. 일반 메서드는 프로토타입에서 공유한다. 따라서 필드 문법과 메서드 문법을 모두 “클래스 안에 있으니 하나만 생긴다”로 설명하면 틀린다.

화살표 함수를 필드 초기값으로 만들면 함수도 인스턴스마다 새로 평가된다. this 연결을 유지하는 장점과 함수 공유·동일성의 차이를 함께 고려한다. 문법이 한 줄로 줄었다는 이유만으로 모든 메서드를 화살표 필드로 바꿀 필요는 없다.

## 4. 정적 요소와 접근 위치

```javascript
class User {
  constructor(name) { this.name = name; }
  static from(text) { return new User(text.trim()); }
  label() { return "user:" + this.name; }
}
const user = User.from(" Kim ");
console.log(user.label()); // user:Kim
console.log(typeof user.from, typeof User.label); // undefined undefined
```

**정적(static) 메서드**는 클래스 자체의 속성이다. 인스턴스를 만들기 전 수행하는 생성 보조·분류 작업 등에 적합하다. 인스턴스 메서드와 호출 위치가 다르다. `static`이 전역 변수나 모든 인스턴스에 복사된 함수라는 뜻은 아니다.

### 정적 상태와 인스턴스 상태

```javascript
class Ticket {
  static issued = 0;
  constructor() {
    Ticket.issued += 1;
    this.number = Ticket.issued;
  }
}
const a = new Ticket();
const b = new Ticket();
console.log(a.number, b.number, Ticket.issued); // 1 2 2
console.log(Object.hasOwn(a, "issued")); // false
```

issued는 클래스 쪽에서 공유하고 number는 각 인스턴스에 저장한다. 같은 카운트를 각 인스턴스의 필드로 두었다면 전체 발급 횟수가 되지 않는다. 어떤 상태를 모든 객체가 공유해야 하는지에 따라 위치를 정한다.

정적 메서드 안의 this도 호출 수신자 규칙을 따른다. 항상 특정 클래스 이름으로 고정된 것이 아니다. 상속된 정적 메서드를 파생 클래스에서 부를 수 있으므로 this와 명시적 클래스 이름을 어느 목적으로 썼는지 설명할 수 있어야 한다.

## 5. 상속과 super의 실행 순서

```javascript
class Account {
  constructor(owner) { this.owner = owner; }
  label() { return this.owner; }
}
class PremiumAccount extends Account {
  level = "premium";
  constructor(owner) {
    super(owner);
    this.active = true;
  }
  label() { return super.label() + ":" + this.level; }
}
const account = new PremiumAccount("Kim");
console.log(account.label(), account.active); // Kim:premium true
```

`extends`는 인스턴스와 클래스 사이에 상속 연결을 준비한다. 파생 생성자에서는 `super(...)`로 상위 생성 과정을 진행한 뒤 `this`를 사용할 수 있다. 일반적인 경우 기본 클래스 필드는 기본 생성자 본문 전, 파생 클래스 필드는 `super` 반환 뒤 파생 본문을 이어 가기 전에 초기화된다.

`super.label()`은 상위 쪽에서 메서드를 찾지만 `this`는 현재 인스턴스다. 부모 인스턴스를 새로 만들어 메서드를 호출하는 것이 아니다. 기본 생성자에서 재정의 가능한 메서드를 호출하면 파생 필드 초기화 전일 수 있으므로 주의한다.

### 상속은 두 종류의 연결을 만든다

```javascript
class Base {
  static type() { return "base"; }
  describe() { return "instance"; }
}
class Derived extends Base {}
const value = new Derived();
console.log(value.describe(), Derived.type()); // instance base
console.log(Object.getPrototypeOf(Derived) === Base); // true
console.log(Object.getPrototypeOf(Derived.prototype) === Base.prototype); // true
```

클래스 자체의 연결은 정적 요소 조회에, prototype 객체들의 연결은 인스턴스 동작 조회에 사용된다. 두 화살표를 구분하면 static이 인스턴스에 복사된다는 오해를 피할 수 있다.

초기화 순서에서는 기본 생성자가 실행되는 동안 파생 클래스의 인스턴스 필드가 아직 준비되지 않았을 수 있다. 재정의 가능한 메서드를 기본 생성자에서 호출하면 파생 필드에 너무 일찍 접근할 수 있다. 생성 과정에서 외부로 객체를 노출하거나 덮어쓴 메서드를 호출하는 설계는 초기화 조건을 특히 분명히 한다.

## 6. private 필드와 불변성의 구분

```javascript
class Wallet {
  #balance = 0;
  deposit(amount) { this.#balance += amount; }
  get balance() { return this.#balance; }
}
const wallet = new Wallet();
wallet.deposit(10);
wallet["#balance"] = 999;
console.log(wallet.balance, wallet["#balance"]); // 10 999
```

`#balance`는 언어가 접근을 제한하는 private 요소다. 문자열 키 `"#balance"`와 다른 공간이며 일반 속성처럼 꺼내거나 열거하지 못한다. private은 값이 영원히 바뀌지 않는다는 뜻은 아니다. 클래스의 메서드는 위처럼 갱신할 수 있다.

메서드가 private 요소를 읽을 때는 수신 객체가 해당 private 요소를 가진 유효한 객체여야 한다. 아무 객체나 `call`해서 같은 모양의 속성만 제공하는 것으로 대체할 수 없다.

### private 요소는 일반 문자열 키가 아니다

private 이름은 선언된 클래스 문법 안에서 해석되며, 호출 수신자가 해당 private 요소를 실제로 가진 객체인지 확인하는 규칙도 갖는다. 같은 모양의 공개 속성을 넣은 가짜 객체로 메서드를 call하면 private 접근이 실패할 수 있다. 속성 이름을 숨기기 위한 관례적 밑줄과도 다르다.

상속받은 인스턴스는 기본 클래스 생성 과정을 통해 기본 클래스 private 상태를 가질 수 있다. 그러나 파생 클래스 본문에서 부모가 선언한 private 이름을 직접 사용하는 것은 같은 접근 권한을 자동으로 얻는 것이 아니다. 부모의 공개·보호 계약에 해당하는 메서드를 통해 동작을 연결해야 한다.

[ECMAScript Class Definitions](https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-class-definitions)는 클래스 평가·private 이름·인스턴스 요소 초기화를 함께 정의한다. 단순한 함수와 prototype 대입으로 완전히 치환된다고 설명하지 않는 이유다.

## 7. 일반 함수와 다른 제약

클래스 본문은 strict mode로 실행되고, 클래스를 `new` 없이 일반 함수처럼 호출하면 `TypeError`가 난다. 선언 이전에는 TDZ의 제약을 받는다. 클래스 메서드는 일반적으로 프로토타입의 비열거 속성이므로 객체 리터럴 메서드와 속성 열거에서도 차이가 있다. [클래스 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) 참조.

메서드를 꺼내 단독 호출하면 원래 인스턴스의 `this`가 자동으로 유지되지 않는다. 화살표 필드 `increment = () => ...`는 바깥 인스턴스 `this`를 사용할 수 있지만 인스턴스마다 별도의 함수가 생기는 선택이다. 공유 메서드와 같은 메모리·식별성 특성이라고 생각하지 않는다.

### 메서드 호출 방식의 손실

```javascript
class Reader {
  constructor(name) { this.name = name; }
  read() { return this.name; }
}
const value = new Reader("Kim");
const read = value.read.bind(value);
console.log(read()); // Kim
```

bind 없이 `value.read만` 꺼내 호출하면 클래스 메서드의 this는 자동으로 value에 고정되지 않는다. 클래스 본문이 strict mode라는 사실은 이런 잘못된 단독 호출의 문제를 숨기지 않게 한다.

get 접근자도 메서드와 사용 형식이 다르다. `value.balance`를 읽을 때 실행되는 계산과 `value.balance()`를 호출하는 함수는 같은 계약이 아니다. 이름만 보고 속성인지 메서드인지 추측하지 말고 선언을 확인한다.

## 8. 흔한 오해·예외·주의점

- **클래스마다 인스턴스 메서드를 복사한다**: 일반 메서드는 프로토타입으로 공유한다.
- **static 메서드는 인스턴스에서 호출한다**: 클래스 자체에서 사용한다.
- **private은 readonly다**: 접근 제한과 변경 제한은 다르다.
- **상속은 코드 재사용의 유일한 방법이다**: 작은 객체의 조합이나 일반 함수로 요구를 더 단순하게 표현할 수도 있다.

### 객체를 강제로 불변으로 만드는 문법은 아니다

class·private·static은 객체 생성과 접근 위치를 조직하는 문법이다. 이 중 어느 하나도 모든 내부 상태의 변경을 자동 금지하지 않는다. private 필드를 갖는 인스턴스라도 해당 클래스 메서드가 값을 바꾸면 상태가 바뀐다.

상속을 코드 중복 제거 도구로만 사용하면 부모의 호출 순서와 숨은 상태에 강하게 의존할 수 있다. 하위 객체가 부모의 사용 계약을 지킬 수 있는지 먼저 확인하고, 단순한 기능 조합이면 일반 객체나 함수를 인자로 결합하는 방식도 비교한다.

## 9. 실무 적용과 디버깅 관점

메서드가 없으면 속성이 클래스·인스턴스·프로토타입 중 어디에 놓였는지 검사한다. 초기값이 `undefined`라면 파생 필드 초기화 전 호출인지 확인한다. `this` 오류라면 전달 과정에서 메서드가 분리됐는지 확인한다.

여러 객체가 같은 규칙으로 상태를 갖고 행동할 때 클래스를 고려한다. 단순 데이터 변환만 필요하다면 함수와 레코드로도 충분할 수 있다. 상속을 늘리기 전 두 객체가 같은 동작 계약을 지키는지 확인한다.

### 초기화 문제를 단계별로 기록한다

인스턴스 생성 시작, 기본 필드 준비, 기본 생성자 본문, super 반환, 파생 필드 준비, 파생 생성자 나머지 코드를 구분해 기록한다. 값이 undefined인 시점이 단순 오타 때문인지 아직 초기화되지 않은 순서 때문인지 나눌 수 있다.

메서드가 보이지 않으면 `Object.hasOwn으로` 인스턴스의 필드를 확인한 뒤 prototype으로 조회를 이어 간다. static으로 선언된 요소라면 클래스 자체를 검사한다. 선언 위치·조회 위치·호출 수신자를 함께 확인해야 구조적인 오류를 찾을 수 있다.

### 통합 사례: 기본 생성자에서 파생 메서드를 부르는 함정

클래스의 모든 필드가 생성자 시작 시 한꺼번에 준비된다고 생각하면 상속 초기화 순서를 놓칠 수 있다. 기본 생성자에서 파생 메서드로 동적 호출이 이어지는 사례를 보자.

```javascript
class Base {
  constructor() { this.recorded = this.describe(); }
  describe() { return "base"; }
}
class Derived extends Base {
  label = "ready";
  describe() { return this.label; }
}
const value = new Derived();
console.log(value.recorded, value.describe()); // undefined ready
```

Base 생성자에서 `this.describe를` 조회하면 실제 인스턴스의 파생 메서드를 찾는다. 하지만 그 시점에는 Derived의 label 필드가 아직 초기화되지 않아 recorded에 undefined가 저장된다. super의 기본 생성 과정이 돌아온 뒤 파생 필드가 준비되어 나중 호출은 ready를 읽는다.

이 차이는 this가 다른 객체였기 때문이 아니라 같은 인스턴스의 초기화 단계가 달랐기 때문이다. 초기화 중 재정의 가능한 메서드를 호출하지 않거나, 모든 필드가 준비된 뒤 명시적인 준비 단계를 수행하는 설계를 고려한다.

검증에서는 객체 생성 직후의 최종 상태만 보지 말고 생성 과정 중 메서드 호출이 언제 일어났는지 기록한다. 같은 코드의 메서드를 호출해도 초기화 전후의 입력 상태가 달라지면 결과가 달라질 수 있다.

## 10. 핵심 정리

클래스는 생성·상태·공통 동작의 경계를 만든다. 개별 상태는 인스턴스, 일반 메서드는 프로토타입, 정적 요소는 클래스 자체에 놓인다. strict mode·TDZ·private·초기화 순서를 포함해야 동작을 정확히 설명할 수 있다.

## 출처

- [MDN — Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes): 필드·메서드·정적 요소·상속.
- [ECMAScript — Class Definitions](https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-class-definitions): 클래스 평가·생성·private 요소.

## 11. 이해도 확인 질문

1. 인스턴스 두 개의 일반 메서드는 같은 함수인데 필드 값은 다른 이유를 설명하라.
2. `static create()`를 인스턴스에서 호출하면 생기는 문제와 올바른 호출 위치는 무엇인가?
3. 파생 생성자의 `super` 전에 `this`를 읽으면 안 되는 이유를 실행 순서로 설명하라.
4. `obj["#value"]` 대입이 실제 private 필드를 바꾸지 못하는 이유는 무엇인가?
5. 일반 메서드를 화살표 필드로 바꾸면 this와 함수 공유에 어떤 차이가 생기는가?
