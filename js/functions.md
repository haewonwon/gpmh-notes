---
track: A
notion: https://app.notion.com/p/400178c4afac800b9303e6f98bf17636
---

<callout icon="📍" color="gray_bg">
**학습 범위**: Track A · 1–3주 · JavaScript Basics · 함수 · Function Declaration / Expression / Arrow Function

각 예제는 별도로 실행한다. 별도 표시가 없으면 현대 브라우저 또는 `Node.js의` 표준 JavaScript를 기준으로 하며, `console`은 실행 환경이 제공하는 출력 도구다.
</callout>

<table-of-contents />

## 1. 학습 목표

- 함수의 정의·호출·반환을 구분하고 실행 순서를 추적한다.
- 선언문·표현식·화살표 함수의 생성 시점과 기본 사용 기준을 비교한다.
- 인자 전달, 기본값, 반환 누락에서 발생하는 버그를 설명한다.
- 함수가 원본 객체를 변경하는지 계약으로 드러낸다.

## 2. 이 주제를 왜 배우는가

같은 할인 계산을 여러 곳에 복사하면 규칙이 바뀔 때 일부만 수정하기 쉽다. 함수를 만들면 입력과 출력을 한곳에 모아 계산 규칙을 재사용할 수 있다. 하지만 문법만 짧게 바꾸는 것으로 충분하지 않다. 값을 반환하는 함수인지, 화면에 출력하는 함수인지, 외부 객체를 바꾸는 함수인지도 분명해야 한다.

함수는 실행할 코드를 담은 **호출 가능한 객체**다. 이름에 저장하고, 다른 이름에 대입하고, 반환할 수 있다. `calculate`와 `calculate()`는 각각 함수 값과 지금 호출한 결과이므로 서로 바꾸어 사용할 수 없다.

### 재사용보다 먼저 입력과 효과를 정한다

할인 계산을 함수로 추출해도 그 함수가 내부에서 장바구니를 바꾸고 로그를 출력하고 계산값까지 반환한다면 호출자의 의존성은 여전히 복잡하다. 먼저 “입력 가격과 할인율을 받아 숫자를 반환한다”처럼 계산 계약을 적고, 바깥 상태 변경이 필요한지 결정한다.

이 구분은 테스트 가능성과도 연결된다. 같은 입력의 반환값만 확인하면 되는 함수와, 호출 순서에 따라 외부 상태가 달라지는 함수는 검증할 대상이 다르다. 함수를 짧게 만드는 것보다 호출자가 알아야 할 조건을 줄이는 것이 중요하다.

## 3. 정의·호출·입력·출력

**매개변수(parameter)** 는 함수 정의에 적은 입력 이름이고, **인자(argument)** 는 호출할 때 실제로 전달한 값이다. **반환값(return value)** 은 함수 호출 표현식의 결과다.

```javascript
function discounted(price, rate) {
  return price * (1 - rate);
}

const total = discounted(10000, 0.2);
console.log(total); // 8000
```

호출 시 인자를 평가하고, `price`와 `rate`에 전달한 뒤 본문을 실행한다. `return`을 만나면 현재 호출을 끝내고 `8000`을 호출 위치로 돌려준다. 따라서 `total`에는 함수 자체가 아닌 계산 결과가 저장된다.

함수를 정의하는 것만으로 본문이 실행되지는 않는다. 같은 함수라도 호출마다 입력과 지역 변수가 구분된다.

### 호출 표현식의 값은 언제 결정되는가

```javascript
function input(label, value) {
  console.log(label);
  return value;
}
function add(a, b) {
  console.log("body");
  return a + b;
}
const result = add(input("left", 2), input("right", 3));
console.log(result);
```

출력은 `left`, `right`, `body`, `5`다. 인자를 먼저 왼쪽부터 평가하므로 본문에 들어갔을 때 매개변수에는 이미 계산된 값이 연결된다. 인자 평가에서 오류가 발생하면 함수 본문은 시작하지 못할 수도 있다.

매개변수는 호출마다 사용하는 지역 이름이다. 호출자가 사용한 변수 이름이 함수 안으로 그대로 들어가는 것이 아니다. `const price = 10; discounted(price, rate)`에서 함수는 `price`라는 글자를 받은 것이 아니라 그 이름을 읽은 값을 받는다. 이 차이가 재할당과 객체 변경의 차이를 설명하는 기반이다.

## 4. 선언문·표현식·화살표 함수

```javascript
function add(a, b) {
  return a + b;
}
const subtract = function (a, b) {
  return a - b;
};
const multiply = (a, b) => a * b;

console.log(add(7, 2), subtract(7, 2), multiply(7, 2)); // 9 5 14
```

| 형태 | 무엇을 만드는가 | 본문을 사용할 수 있는 시점 |
| --- | --- | --- |
| 함수 선언문 | 이름이 있는 함수 선언 | 일반적인 함수·모듈 범위에서는 선언 앞에서도 호출 가능 |
| 함수 표현식 | 평가 결과가 함수인 표현식 | 표현식 평가와 변수 초기화 이후 |
| 화살표 함수 | 간결한 함수 표현식 | 표현식 평가와 변수 초기화 이후 |

함수 표현식의 이름은 저장하는 변수 이름과 다를 수 있다. 이름 있는 표현식은 내부 재귀나 오류 추적에 유용하다. `const`에 저장했다는 사실이 함수의 본문이나 외부 상태를 불변으로 만들지는 않는다.

화살표 함수는 단순히 `function`의 축약판은 아니다. 자신만의 `this`·`arguments`가 없고 `new`로 호출할 수 없다. 여기서는 값 계산과 문법 차이를 중심으로 배우고, 객체 메서드의 호출 규칙은 후속 주제로 연결한다.

### 선언문은 선언이고 표현식은 평가 결과다

함수 선언문은 주변 코드에 이름 있는 함수를 선언한다. 함수 표현식은 다른 표현식처럼 평가되며 그 결과가 함수 값이다. 따라서 변수에 저장하거나 조건의 결과로 선택할 수 있다. 화살표 함수도 이 표현식 쪽에 속한다.

```javascript
const operation = true
  ? function increase(n) { return n + 1; }
  : function decrease(n) { return n - 1; };
console.log(operation(4)); // 5
console.log(typeof increase); // undefined
```

이름 있는 함수 표현식의 내부 이름 `increase`는 바깥의 독립된 선언 이름이 아니다. 표현식의 이름은 자기 참조와 오류 추적에 도움을 주지만, 바깥에서 사용할 연결은 `operation`이다. “함수에 이름이 있다”와 “그 이름이 현재 범위에 선언된다”를 구분한다. [ECMAScript 함수 정의](https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html#sec-function-definitions)에서 두 생성 규칙을 확인할 수 있다.

## 5. 반환 흐름과 암시적 반환

```javascript
function printSum(a, b) {
  console.log(a + b);
}
const result = printSum(2, 4);
console.log(result); // undefined

const double = n => n * 2;
const missing = n => { n * 2; };
const createItem = name => ({ name: name });
console.log(double(3), missing(3), createItem("pen").name);
```

예상 출력은 순서대로 `6`, `undefined`, `6 undefined pen`이다. `console.log`는 출력 효과이며 반환을 대신하지 않는다. 함수가 끝날 때까지 `return`을 만나지 않으면 보통 `undefined`가 반환된다.

화살표 함수의 본문이 **표현식**이면 그 평가 결과를 반환한다. 중괄호로 감싸면 문장들의 본문이 되므로 `return`이 필요하다. 객체 리터럴을 그대로 반환하려면 `({ ... })`로 감싸서 함수 본문과 구별한다. [화살표 함수 문서](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) 참조.

`return` 다음 줄에 표현식을 쓰면 자동 세미콜론 삽입으로 의도와 달라질 수 있다. 반환할 표현식은 같은 줄에서 시작하거나 괄호로 묶는다. 오류 여부만 보지 말고 호출 결과가 의도한 값인지 확인한다.

### 여러 반환 경로와 자동 세미콜론

```javascript
function classify(n) {
  if (n > 0) return "positive";
  if (n === 0) return "zero";
}
function broken() {
  return
  { ok: true };
}
console.log(classify(-1), broken()); // undefined undefined
```

첫 함수는 음수 입력 경로에 반환이 없고, 둘째는 `return` 뒤 줄바꿈 때문에 의도한 객체 표현식이 반환에 연결되지 않는다. 둘 다 실행은 되지만 호출자는 필요한 값을 받지 못한다. 입력별 경로를 적어 보면 반환 누락과 구문 배치를 분리해서 찾을 수 있다.

객체 반환은 `return { ok: true };`처럼 표현식을 같은 줄에서 시작한다. 긴 객체는 `return (`로 시작해 괄호 안에 작성할 수 있다. 출력문을 추가해 함수가 실행됐다는 사실만 확인하는 것으로는 반환 계약을 검증할 수 없다.

## 6. 인자 개수와 기본값

```javascript
function label(name = "guest", suffix = "!") {
  return name + suffix;
}
console.log(label()); // guest!
console.log(label(undefined, "?")); // guest?
console.log(label(null)); // null!
console.log(label("Kim", "!", "ignored")); // Kim!
```

인자가 없거나 `undefined`이면 기본값을 적용한다. `null`은 실제로 전달한 값이므로 기본값으로 대체하지 않는다. 인자가 부족하거나 많다는 이유만으로 일반 함수가 자동으로 실패하지도 않는다. 필수 입력의 유효성은 함수가 확인해야 한다.

기본값은 함수 정의 때 한 번 고정되는 것이 아니라 호출 시 필요한 경우 평가된다. 기본값을 제공했다고 잘못된 문자열이나 음수까지 검증된 것은 아니다.

### 기본값은 호출 때 필요한 경우에 평가된다

```javascript
let created = 0;
function makeDefault() { created += 1; return created; }
function read(value = makeDefault()) { return value; }
console.log(read(10), created); // 10 0
console.log(read(), read(), created); // 1 2 2
```

인자를 제공하면 기본값 식은 실행하지 않는다. 생략한 호출마다 식을 다시 평가하므로 기본값이 객체를 만들면 일반적으로 호출마다 새 객체가 생긴다. 기본값을 “정의할 때 한 번 계산한 값”으로 이해하면 호출 횟수와 상태 변화를 잘못 예측한다.

기본값에 외부 상태 변경을 많이 넣으면 입력이 생략됐을 때에만 부수 효과가 발생한다. 읽기 쉬운 계약이 중요하다면 복잡한 준비는 본문에서 명시적으로 수행한다. 필수 인자가 빠졌다는 사실을 기본값으로 숨길지도 의도적으로 정한다.

## 7. 객체 인자와 함수 값

```javascript
function update(item) {
  item.count += 1;
  item = { count: 100 };
  return item.count;
}
const cart = { count: 1 };
const action = update;
console.log(action === update); // true
console.log(action(cart)); // 100
console.log(cart.count); // 2
```

매개변수 `item`은 전달받은 객체를 통해 `count`를 바꿀 수 있다. 그러나 매개변수 자체에 새 객체를 대입해도 호출자의 `cart` 연결이 교체되지는 않는다. JavaScript의 인자 전달은 값 전달이며, 객체를 가리키는 값을 공유한다.

이 예제에서 `action = update`는 함수 자체를 보관한다. `action = update(cart)`였다면 지금 함수를 실행하고 숫자를 보관했을 것이다.

### 객체 변경과 새 결과 반환을 비교한다

```javascript
function rename(user, name) {
  return { name: name, age: user.age };
}
const original = { name: "Kim", age: 20 };
const next = rename(original, "Lee");
console.log(original.name, next.name); // Kim Lee
console.log(original === next); // false
```

이 함수는 새 객체를 만들어 반환하므로 호출자가 그 결과를 사용할지 결정한다. 반면 `user.name = name`을 수행하는 함수는 반환값을 버려도 원본에 효과를 남긴다. 따라서 반환값 사용 여부와 부수 효과는 별도로 검토해야 한다.

이 예제의 새 객체는 필드가 단순한 경우를 보여 준다. 모든 속성이나 중첩 구조를 복제하는 일반적인 복사 함수가 아니다. 현재 함수의 입력·출력 계약에 필요한 필드만 명시한 것이다.

## 8. 흔한 오해·예외·주의점

- `return` 뒤에 도달할 수 없는 코드를 추가해도 실행되지는 않는다. 조건 분기마다 필요한 반환이 있는지 확인한다.
- 표현식을 중괄호로 감싼 뒤 반환이 사라지는 버그는 문법 오류가 아닐 수 있다.
- 블록 내부 함수 선언의 비엄격 스크립트 동작에는 역사적인 규칙이 있다. 예측 가능한 코드를 위해 함수·모듈의 명확한 범위에 선언한다.
- 기본 인자는 잘못된 입력 전체를 정상값으로 바꾸지 않는다.
- 계산만 하는 것처럼 보이는 함수가 인자를 수정하면 호출자의 코드에도 영향이 생긴다.

### 오류의 위치는 함수 몸체 밖일 수도 있다

`f()`가 실패했다고 본문에 항상 들어간 것은 아니다. `f`가 함수 값인지 확인하는 단계에서 실패할 수 있고, 호출 인자를 계산하는 다른 함수가 실패할 수도 있다. 오류 줄과 함수 진입 중단점을 함께 보면 본문 전 실패인지 실행 중 실패인지 구분할 수 있다.

함수 이름도 진단 정보다. 의미 있는 이름은 오류 경로에서 어느 책임이 실패했는지 보여 준다. 다만 이름이 같다고 같은 함수 객체이거나 같은 호출 상태라는 뜻은 아니다. 소스 위치와 전달 경로까지 확인한다.

## 9. 실무 적용과 디버깅 관점

함수를 읽을 때 입력, 반환, 외부 변경을 먼저 표시한다. 계산 함수는 결과를 반환하고, 출력·저장 같은 효과는 이름과 호출 위치로 드러내면 문제 범위를 좁히기 쉽다. `getTotal`이 장바구니도 수정한다면 호출자가 예상하기 어렵다.

반환이 `undefined`이면 모든 실행 경로의 `return`, 화살표 함수의 중괄호, 호출 시 인자를 점검한다. 함수가 실행되지 않는다면 함수 값을 전달한 것인지 실제로 호출했는지 확인한다. 디버거에서 진입 전 인자와 반환 직전 값을 나란히 비교하면 계산 오류와 전달 오류를 분리할 수 있다.

### 반환 계약을 기준으로 최소 재현을 만든다

계산값이 잘못됐다면 외부 입력을 고정한 작은 함수 호출로 줄인다. 그 상태에서 입력을 받는 순간, 분기 직전, 반환 직전 값을 확인한다. 값이 `undefined`라면 모든 분기가 값을 반환하는지부터 살피고, 값은 맞지만 호출자 상태가 바뀌었다면 객체 속성 대입을 찾는다.

계산 함수의 검증에서는 단순히 “오류 없이 실행됨”보다 구체적인 반환값이 중요하다. 객체를 반환한다면 필요한 필드뿐 아니라 원본 변경 여부와 새 객체 여부도 요구에 맞춰 확인한다. 함수의 길이·문법 종류보다 호출자가 의존하는 관찰 결과를 검증한다.

## 10. 필요한 미리보기

**[미리보기: Hoisting · TDZ]** 선언문과 `const` 함수 표현식의 선언 전 호출 차이는 실행 준비와 초기화 시점에서 나온다. 정확한 준비 순서는 JavaScript Core의 Hoisting · TDZ에서 다룬다.

**[미리보기: this]** 일반 함수의 `this`는 주로 호출 방식으로 정해지고 화살표 함수는 바깥의 `this`를 사용한다. 객체 메서드·`call`·`apply`·`bind`는 JavaScript Core의 해당 문서에서 다룬다.

## 11. 핵심 정리

함수 정의와 호출, 출력과 반환을 분리한다. 선언문·표현식·화살표 함수는 사용 시점과 호출 특성이 다르다. 좋은 함수의 경계는 입력과 출력뿐 아니라 원본을 바꾸는지까지 드러낸다.

## 출처

- [ECMAScript — Functions and Classes](https://tc39.es/ecma262/multipage/ecmascript-language-functions-and-classes.html): 함수 정의와 호출 의미.
- [MDN — Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions): 매개변수, 함수 표현식, 객체 인자 전달.
- [MDN — Arrow function expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions): 표현식 반환, `this`와 생성자 제약.

## 12. 이해도 확인 질문

1. `const f = n => { n + 1; };`의 호출 결과를 설명하고 값을 반환하도록 고쳐라.
2. `const action = calculate;`와 `const action = calculate();`가 저장하는 값을 비교하라.
3. 기본 매개변수에 인자를 생략한 경우와 `undefined`, `null`을 전달한 경우를 구분하라.
4. 함수 안에서 객체 속성 변경과 매개변수 재할당이 호출자에게 미치는 영향을 설명하라.
5. 함수 선언문을 `const` 함수 표현식으로 바꿀 때 호출 위치에서 무엇을 확인해야 하는가?
