---
track: A
notion: https://app.notion.com/p/415178c4afac8180b8ffd968129e988c
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Core · Prototype · Prototype Chain

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 자신의 속성과 상속받은 속성을 구분한다.
- `[[Prototype]]`과 생성자 함수의 `.prototype`을 비교한다.
- 속성 읽기·가리기·삭제와 메서드의 `this`를 추적한다.
- 공통 동작 공유와 프로토타입 오염 문제를 설명한다.

## 2. 이 주제를 왜 배우는가

빈 배열에도 `map`을 호출할 수 있고, 직접 넣지 않은 `toString`이 객체에서 발견된다. 모든 객체에 같은 함수를 매번 복사한 것은 아니다. 객체는 다른 객체와 연결되어 자신에게 없는 속성 조회를 이어 갈 수 있다.

**프로토타입(prototype)** 은 객체가 속성 조회를 이어 가는 대상 객체다. 객체의 내부 연결을 명세에서는 **`[[Prototype]]`** 으로 나타낸다. 이 연결을 따라가는 구조가 **프로토타입 체인(prototype chain)** 이다.

### 복제 없이 동작을 공유하는 구조

사용자 객체마다 같은 greet 함수를 새로 만들어 넣으면 각 객체는 독립된 함수 속성을 갖는다. 프로토타입에 하나의 greet를 두면 필요한 때 그 연결을 따라 찾을 수 있다. 이 구조는 같은 동작을 공유하면서 이름·점수 같은 상태는 인스턴스별로 두게 한다.

공유가 가능하다는 사실은 부모 상태까지 모두 함께 둬야 한다는 뜻이 아니다. 읽기 위임, 함수 호출의 수신자, 실제 변경되는 객체를 나누어 추적해야 공통 메서드와 공유 배열의 차이를 설명할 수 있다.

## 3. 자신의 속성과 상속 속성

```javascript
const base = { role: "reader" };
const user = Object.create(base);
user.name = "Kim";
console.log(user.name, user.role); // Kim reader
console.log(Object.hasOwn(user, "role")); // false
console.log("role" in user); // true
console.log(Object.getPrototypeOf(user) === base); // true
```

`Object.create(base)`는 `base`를 프로토타입으로 연결한 새 객체를 만든다. `name`은 자신의 속성, `role`은 프로토타입에서 찾은 속성이다. `Object.hasOwn`은 자신만 검사하고 `in`은 체인도 검사한다.

```text
user { name: "Kim" }
  → base { role: "reader" }
  → Object.prototype
  → null
```

끝의 `null`은 더 찾을 프로토타입이 없다는 뜻이다. 체인 어디에도 속성이 없으면 일반적인 속성 읽기는 `undefined`를 반환한다. 속성이 없다는 것과 값이 `undefined`인 속성이 존재한다는 것은 다르다.

### 속성 읽기의 결정 절차

일반 객체의 읽기는 먼저 자신의 속성 기술자를 찾는다. 데이터 속성이면 값을 반환하고, 접근자 속성이면 getter를 호출한다. 자신의 속성이 없으면 프로토타입으로 같은 조회를 이어 가며 끝이 null이면 undefined를 반환한다.

```text
현재 객체에 자신의 속성이 있는가?
├─ 데이터 속성 → 저장된 값 반환
├─ 접근자 속성 → getter를 원래 수신자로 호출
└─ 없음 → [[Prototype]]으로 이동, 없으면 undefined
```

여기서 **속성 기술자(property descriptor)** 는 값·쓰기 가능 여부 또는 getter/setter 등의 성질을 나타내는 기록이다. 프로토타입 검색을 단순한 여러 객체의 얕은 병합으로 설명하면 getter의 수신자와 자신의 속성 검사 차이를 놓친다. [ECMAScript OrdinaryGet](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinaryget)이 이 절차를 정의한다.

## 4. 속성 검색과 가리기

```javascript
const base = { score: 1 };
const child = Object.create(base);
console.log(child.score); // 1
child.score = 5;
console.log(child.score, base.score); // 5 1
delete child.score;
console.log(child.score); // 1
```

읽을 때는 자신부터 바깥 프로토타입으로 찾는다. 일반적인 쓰기 가능한 데이터 속성의 경우 `child.score = 5`는 자식의 자신의 속성을 만들어 부모 값을 가린다. 삭제하면 가리던 속성이 없어져 부모의 값이 다시 보인다.

모든 대입이 반드시 자신의 속성을 만드는 것은 아니다. 상속된 setter가 있거나 쓰기 불가 속성이 있으면 다른 결과가 생긴다. 따라서 읽기 체인 규칙을 쓰기 전체의 규칙으로 단순 확장하지 않는다. [프로토타입 체인 설명](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain) 참조.

### getter와 setter는 원래 수신자를 사용한다

```javascript
const base = {
  get label() { return this.name; },
  set label(value) { this.name = value; }
};
const child = Object.create(base);
child.name = "Kim";
console.log(child.label); // Kim
child.label = "Lee";
console.log(child.name, Object.hasOwn(child, "label")); // Lee false
console.log(Object.hasOwn(base, "name")); // false
```

getter·setter는 base에서 찾았지만 this는 child다. setter를 통해 `child.name을` 갱신했으므로 label이라는 자신의 데이터 속성을 새로 만들지 않았다. 읽기에서 발견한 객체와 실제 값을 읽거나 쓰는 수신자가 다른 사례다.

상속된 속성이 쓰기 불가 데이터 속성이면 대입이 실패할 수도 있다. 따라서 “대입하면 무조건 자식 속성이 생긴다”는 규칙은 쓰기 가능한 일반 데이터 속성으로 조건을 제한해야 한다.

## 5. 생성자의 prototype과 객체의 내부 연결

```javascript
function User(name) {
  this.name = name;
}
User.prototype.greet = function () {
  return "hello " + this.name;
};
const a = new User("Kim");
const b = new User("Lee");
console.log(a.greet(), b.greet()); // hello Kim hello Lee
console.log(a.greet === b.greet); // true
console.log(Object.getPrototypeOf(a) === User.prototype); // true
```

`User.prototype`은 함수 객체에 있는 보통의 이름 붙은 속성이다. `new User`의 일반적인 생성 과정에서 새 객체의 `[[Prototype]]`에 이 객체가 연결된다. 함수 `User` 자신의 `[[Prototype]]`과 `User.prototype`은 같은 개념이 아니다.

`greet`는 각 인스턴스에 복사되지 않고 공유된다. 하지만 `a.greet()`의 `this`는 여전히 `a`다. 메서드를 찾은 위치와 메서드 호출의 수신자는 다를 수 있다.

### 생성자 함수도 객체라는 점을 함께 본다

```javascript
function User() {}
const user = new User();
console.log(Object.getPrototypeOf(User) === Function.prototype); // true
console.log(Object.getPrototypeOf(user) === User.prototype); // true
console.log(Object.getPrototypeOf(User.prototype) === Object.prototype); // true
```

첫 줄은 함수 객체 User 자신의 상속 연결이고, 둘째는 생성된 user의 연결이다. `User.prototype은` 인스턴스 쪽에 사용할 객체를 가리키는 속성이다. 도식에서 화살표를 하나로 합치면 함수의 메서드와 인스턴스의 메서드를 혼동한다.

또한 생성 뒤 `User.prototype을` 다른 객체로 교체해도 기존 인스턴스의 연결이 소급해서 새 객체로 바뀌지는 않는다. 이미 연결된 객체의 메서드를 수정하는 것과 생성자 속성에 새 객체를 대입하는 것은 다른 변경이다.

## 6. 공유 객체를 놓았을 때의 함정

```javascript
function Basket() {}
Basket.prototype.items = [];
const a = new Basket();
const b = new Basket();
a.items.push("pen");
console.log(b.items.join(",")); // pen
console.log(Object.hasOwn(a, "items")); // false
```

`a.items` 읽기가 공유 배열을 찾아 반환하고, `push`는 그 배열을 바꾼다. `a.items`에 새 배열을 대입한 것이 아니므로 자신의 속성이 생기지 않는다. 인스턴스마다 달라야 할 상태는 생성자에서 `this.items = []`로 만든다. 공통 동작과 개별 상태를 구분하는 이유다.

### 공유된 메서드와 공유된 변경 가능 상태

```javascript
function Basket() { this.items = []; }
Basket.prototype.add = function (name) { this.items.push(name); };
const a = new Basket();
const b = new Basket();
a.add("pen");
console.log(a.items.join(","), b.items.length); // pen 0
console.log(a.add === b.add, a.items === b.items); // true false
```

메서드는 공유하지만 그 메서드가 this로 읽는 items는 각 인스턴스에 있다. 같은 동작을 공유하는 것과 같은 데이터를 공유하는 것을 분리했기 때문에 a의 변경이 b에 전달되지 않는다.

버그가 생기면 메서드를 인스턴스마다 새로 만드는 것으로 해결하려고 하기 전에, 메서드가 읽는 변경 가능 객체를 어느 위치에 두었는지 확인한다. 함수 공유 자체가 원인이 아닐 수 있다.

## 7. 검사와 경계

`Object.getPrototypeOf`로 실제 연결을 확인하고 `Object.hasOwn`으로 속성의 소유 위치를 확인한다. `for...in`은 상속된 열거 가능한 속성도 볼 수 있어 입력 객체 순회에 주의해야 한다. `Object.keys`는 자신의 열거 가능한 문자열 키를 다룬다.

`Object.create(null)`은 `Object.prototype`과 연결되지 않은 객체를 만든다. 이름 사전으로 유용할 수 있지만 `toString` 같은 기본 메서드가 없다는 차이도 있다. 동적 키 저장소라면 앞서 배운 Map도 비교한다.

`instanceof`는 보통 생성자의 prototype이 체인에 있는지 확인한다. 객체의 모든 업무상 속성이 올바른지 검증하는 도구는 아니며, 서로 다른 realm이나 사용자 정의 동작도 고려해야 한다.

### 자신의 속성만 처리할 때의 기준

```javascript
const base = { inherited: 1 };
const child = Object.create(base);
child.own = 2;
console.log(Object.keys(child).join(",")); // own
const seen = [];
for (const key in child) seen.push(key);
console.log(seen.join(",")); // own,inherited
```

for...in은 상속된 열거 가능 속성까지 방문할 수 있다. 외부 입력의 직접 필드만 검증하려면 그 범위에 맞는 열거·소유 검사 API를 선택한다. 객체가 hasOwnProperty라는 이름을 직접 가질 수도 있으므로 대상 객체의 같은 이름 메서드를 무조건 호출하기보다 `Object.hasOwn` 같은 형태를 사용할 수 있다.

`constructor` 속성도 인스턴스의 변경 불가능한 출생 증명서가 아니다. 보통 프로토타입에서 찾는 보통 속성이므로 덮어쓰거나 다른 객체로 연결될 수 있다. 실제 연결과 업무상 입력 검증을 constructor 이름 하나로 대신하지 않는다.

## 8. 흔한 오해·예외·주의점

- **프로토타입은 부모 객체의 복사본이다**: 연결을 통해 조회를 공유한다.
- **모든 함수는 new로 호출할 수 있다**: 화살표·메서드 등은 생성 가능하지 않을 수 있다.
- **prototype 속성과 내부 프로토타입은 같다**: 어느 객체의 어느 연결인지 구분한다.
- **속성 읽기와 쓰기는 항상 같은 체인을 따라 수정한다**: 자신의 속성 생성·setter·속성 제한이 작용한다.

### 상속 연결을 바꾸는 것과 속성을 수정하는 것

기존 프로토타입 객체의 메서드를 바꾸면 그 객체를 연결해 사용하는 인스턴스들이 바뀐 조회 결과를 볼 수 있다. 반면 인스턴스가 자신의 같은 이름 속성으로 가리고 있으면 프로토타입 쪽 변경이 보이지 않을 수 있다. 같은 변경이 어떤 객체에는 반영되고 어떤 객체에는 안 보일 때 소유 위치를 비교한다.

실행 중 `Object.setPrototypeOf로` 연결을 반복 교체하면 의미와 성능 추적이 어려워질 수 있다. 생성 시 구조를 정하고 필요한 개별 데이터를 자신의 속성에 넣는 설계를 우선 검토한다. 언어가 허용하는 조작과 유지하기 쉬운 구조는 같은 기준이 아니다.

## 9. 실무 적용과 디버깅 관점

예상하지 못한 속성이 보이면 직접 가진 것인지 체인에서 온 것인지 검사한다. 내장 프로토타입을 임의로 수정하면 무관한 코드에 영향을 줄 수 있으므로 공통 기능을 별도 함수로 두는 편이 안전하다.

외부 입력을 깊게 병합하는 코드에서 특수 키를 통해 프로토타입이나 공유 객체를 오염시키지 않도록 허용 필드를 제한한다. 이 문제는 상속과 입력 검증이 만나는 지점이다. 런타임 중 프로토타입을 자주 교체하기보다 생성 시 구조를 명확히 정한다.

**[미리보기: Class]** 클래스 문법도 인스턴스의 프로토타입을 통해 메서드를 공유한다. 생성자·인스턴스 필드·상속 문법은 다음 Class에서 다룬다.

### 예상하지 못한 속성을 추적하는 절차

문제의 키에 대해 `Object.hasOwn과` `Object.getPrototypeOf를` 번갈아 적용해 어디서 처음 발견되는지 적는다. getter라면 그 호출의 this도 확인하고, 변경이라면 setter를 거쳤는지 자신의 데이터 속성을 만든 것인지 살핀다.

외부 데이터 병합 후 전혀 다른 객체의 기본값이 바뀌었다면 공유된 프로토타입이나 공통 설정 객체에 쓴 경로를 조사한다. 자신이 가진 허용 필드만 처리하도록 입력 경계를 제한하면 상속된 특수 속성과 정상 데이터 속성을 섞는 문제를 줄일 수 있다.

### 통합 사례: 공통 기본값과 개별 재정의

기본 설정을 프로토타입에서 읽고 일부 인스턴스만 자신의 값으로 재정의한 상황을 보자. 공통 기본값 변경이 모든 인스턴스에 무조건 보이는 것은 아니다.

```javascript
const defaults = { theme: "light" };
const first = Object.create(defaults);
const second = Object.create(defaults);
second.theme = "custom";
defaults.theme = "dark";
console.log(first.theme, second.theme); // dark custom
delete second.theme;
console.log(second.theme); // dark
```

first에는 자신의 theme가 없으므로 바뀐 defaults 값을 읽는다. second에는 자신의 데이터 속성이 있어 공통 값을 가린다. 삭제 후에는 다시 defaults에서 찾는다. 공통 기본값 수정, 개별 덮어쓰기, 개별 덮어쓰기 제거는 각각 다른 조작이다.

이 동작을 객체 병합으로 이해하면 defaults 변경이 왜 나중에 보이는지 설명하기 어렵다. 병합으로 자신의 속성에 값을 복사한 경우와, 프로토타입을 따라 필요할 때 읽는 경우를 구분해야 한다. 복사인지 연결인지에 따라 변경 전파가 달라진다.

디버깅에서는 어떤 객체에서 처음 속성을 발견하는지 확인하고, 공통 기본값을 바꾸어도 개별 재정의가 유지되어야 하는지 요구를 확인한다. 프로토타입은 접근을 위임하는 구조이지 모든 객체의 값을 자동으로 동기화하는 장치가 아니다.

## 10. 핵심 정리

프로토타입 체인은 속성 조회의 위임 구조다. 객체의 내부 연결과 생성자의 prototype 속성을 구분한다. 공유할 메서드는 프로토타입에, 인스턴스별 상태는 각 객체에 두는 이유를 실행 결과로 설명할 수 있어야 한다.

## 출처

- [MDN — Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain): 속성 검색·가리기·생성자 연결.
- [ECMAScript — Ordinary Object Internal Methods](https://tc39.es/ecma262/multipage/ordinary-and-exotic-objects-behaviours.html#sec-ordinary-object-internal-methods-and-internal-slots): Get·Set과 내부 프로토타입 규칙.

## 11. 이해도 확인 질문

1. 자신의 속성이 없는 객체에서 `obj.role`을 읽을 때 검색 순서를 설명하라.
2. 상속받은 데이터 속성을 자신의 값으로 가린 뒤 삭제하면 무엇을 읽는가?
3. `Object.getPrototypeOf(instance)`와 `Constructor.prototype`의 관계를 일반적인 new 호출로 설명하라.
4. 프로토타입에 배열을 놓았더니 여러 인스턴스의 항목이 함께 바뀐다. 원인과 수정 위치는 무엇인가?
5. `Object.hasOwn(obj, key)`와 `key in obj` 중 외부 레코드의 직접 필드 확인에는 무엇이 적합한가?
