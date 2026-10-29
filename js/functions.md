---
track: A
notion: https://app.notion.com/p/400178c4afac800b9303e6f98bf17636
---

# 함수 · Function Declaration / Expression / Arrow Function

> 📍 **학습 범위**: JavaScript 기초 · 함수
>
> 함수의 실행 흐름을 이해하고 선언문·표현식·화살표 함수의 차이를 비교한다.
>
> 원시값과 참조 공유: [변수와 자료형 · Primitive / Reference](variables-and-data-types.md)

- [1. 학습 목표](#1-학습-목표)
- [2. 이 주제를 왜 배우는가](#2-이-주제를-왜-배우는가)
- [3. 핵심 개념과 용어](#3-핵심-개념과-용어)
- [4. 함수 호출의 실행 흐름](#4-함수-호출의-실행-흐름)
- [5. Function Declaration: 함수 선언문](#5-function-declaration-함수-선언문)
- [6. Function Expression: 함수 표현식](#6-function-expression-함수-표현식)
- [7. Arrow Function: 화살표 함수](#7-arrow-function-화살표-함수)
- [8. 세 방식의 핵심 비교](#8-세-방식의-핵심-비교)
- [9. 일반 함수와 화살표 함수의 this](#9-일반-함수와-화살표-함수의-this)
- [10. 함수와 값: JavaScript의 중요한 특성](#10-함수와-값-javascript의-중요한-특성)
- [11. 매개변수와 인자의 동작](#11-매개변수와-인자의-동작)
- [12. 함수에 객체를 전달할 때](#12-함수에-객체를-전달할-때)
- [13. 코드 예제와 결과 해설](#13-코드-예제와-결과-해설)
- [14. 흔한 오해·예외·주의점](#14-흔한-오해예외주의점)
- [15. 실무 적용과 디버깅 관점](#15-실무-적용과-디버깅-관점)
- [16. 핵심 정리](#16-핵심-정리)
- [17. 이해도 확인 질문](#17-이해도-확인-질문)

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 함수가 필요한 이유와 함수 호출의 기본 흐름을 설명할 수 있다.
- 매개변수(parameter)와 인자(argument)를 구분할 수 있다.
- `return`의 역할과 반환값이 없을 때의 결과를 설명할 수 있다.
- 함수 선언문, 함수 표현식, 화살표 함수의 문법과 핵심 차이를 구분할 수 있다.
- 호이스팅과 함수 선언 방식의 관계를 기본 수준에서 설명할 수 있다.
- 화살표 함수의 `this` 처리 방식이 일반 함수와 다름을 설명할 수 있다.
- 상황에 맞는 함수 표현 방식을 선택하고, 흔한 함수 버그를 디버깅할 수 있다.

---

## 2. 이 주제를 왜 배우는가

함수는 여러 줄의 코드를 이름 있는 작업으로 묶고, 필요한 시점에 재사용할 수 있게 해 준다. 같은 계산이나 처리 절차를 반복해서 작성하지 않아도 되므로 코드의 중복을 줄이고, 역할을 분리하며, 테스트와 수정도 쉬워진다.

JavaScript에서 함수는 단순한 문법 구조가 아니라 값으로 다룰 수 있는 객체다. 따라서 변수에 저장하고, 다른 함수에 전달하고, 객체의 속성으로 넣을 수도 있다. 이 특성은 JavaScript의 비동기 처리, 이벤트 처리, 배열 메서드, 모듈 설계의 중요한 토대가 된다.

이번 페이지에서는 함수의 기본 실행 모델을 먼저 이해하고, 함수 선언문·함수 표현식·화살표 함수가 어떤 차이를 가지는지 정확히 구분한다.

## 3. 핵심 개념과 용어

### 함수(Function)

함수는 특정 작업을 수행하는 코드의 묶음이다. 함수를 정의만 해서는 내부 코드가 실행되지 않고, 호출해야 실행된다.

```javascript
function greet() {
  console.log("안녕하세요.");
}

greet();
```

예상 결과:

```txt
안녕하세요.
```

위 코드에서 `greet`는 함수의 이름이며, `greet()`는 함수를 호출하는 표현식이다.

### 함수 정의와 함수 호출

함수 사용은 크게 두 단계다.

1. 함수 정의: 어떤 작업을 수행할지 작성한다.
2. 함수 호출: 정의된 작업을 실제로 실행한다.

```javascript
function showMessage() {
  console.log("함수가 실행되었습니다.");
}

showMessage();
showMessage();
```

예상 결과:

```txt
함수가 실행되었습니다.
함수가 실행되었습니다.
```

하나의 함수를 여러 번 호출할 수 있다. 호출할 때마다 함수 본문이 위에서 아래로 실행된다.

### 입력과 출력

함수는 외부에서 데이터를 받아 작업하고, 결과를 반환할 수 있다.

```javascript
function add(a, b) {
  return a + b;
}

const result = add(2, 3);

console.log(result);
```

예상 결과:

```txt
5
```

- `a`, `b`: 함수가 받을 입력을 위한 매개변수
- `2`, `3`: 실제 호출 시 전달한 인자
- `return a + b`: 계산 결과를 호출한 위치로 돌려준다.

### 매개변수(Parameter)와 인자(Argument)

**매개변수(parameter)** 는 함수 정의에서 입력값을 받기 위해 만든 변수다.

```javascript
function greet(name) {
  console.log(`안녕하세요, ${name}님.`);
}
```

위 코드의 `name`이 매개변수다.

**인자(argument)** 는 실제 함수 호출 시 전달하는 값이다.

```javascript
greet("해원");
```

위 코드의 `"해원"`이 인자다.

```javascript
function introduce(name, age) {
  console.log(`${name}은(는) ${age}살입니다.`);
}

introduce("해원", 20);
```

- `name`, `age`: 매개변수
- `"해원"`, `20`: 인자

### 반환값(Return Value)

`return`은 함수의 실행 결과를 호출한 곳으로 돌려준다.

```javascript
function multiply(a, b) {
  return a * b;
}

const value = multiply(3, 4);

console.log(value);
```

예상 결과:

```txt
12
```

`return`을 만나면 함수 실행은 즉시 끝난다.

```javascript
function checkAge(age) {
  if (age < 0) {
    return "나이는 음수일 수 없습니다.";
  }

  return "유효한 나이입니다.";
}
```

첫 번째 `return`이 실행되면 아래의 두 번째 `return`까지 도달하지 않는다.

### 반환값이 없는 함수

함수 안에 `return`이 없으면 호출 결과는 `undefined`다.

```javascript
function printHello() {
  console.log("hello");
}

const result = printHello();

console.log(result);
```

예상 결과:

```txt
hello
undefined
```

> 💡 `console.log()`는 화면이나 콘솔에 값을 보여 주는 작업이고, `return`은 함수를 호출한 코드에 값을 전달하는 작업이다. 둘은 다르다.

## 4. 함수 호출의 실행 흐름

### 호출 스택의 기본 개념

JavaScript는 함수 호출을 관리하기 위해 호출 스택(call stack)이라는 실행 구조를 사용한다. 호출 스택은 현재 실행해야 할 작업을 쌓아 두는 구조다.

다음 코드를 보자.

```javascript
function greet(name) {
  const message = `안녕하세요, ${name}님.`;
  return message;
}

function printGreeting(name) {
  const result = greet(name);
  console.log(result);
}

printGreeting("해원");
```

대략적인 실행 순서는 다음과 같다.

1. 전역 코드 실행이 시작된다.
2. `printGreeting("해원")`을 만나 `printGreeting` 함수가 호출 스택에 쌓인다.
3. `printGreeting` 안에서 `greet(name)`을 만나 `greet` 함수가 그 위에 쌓인다.
4. `greet`가 문자열을 만들어 반환하고 호출 스택에서 제거된다.
5. `printGreeting`이 반환값을 받아 출력하고 호출 스택에서 제거된다.
6. 전역 코드 실행이 끝난다.

개념적으로 표현하면 다음과 같다.

```txt
전역 실행
전역 실행 → printGreeting
전역 실행 → printGreeting → greet
전역 실행 → printGreeting
전역 실행
```

함수 내부의 지역 변수는 해당 함수 호출 동안에만 의미가 있다.

```javascript
function createMessage() {
  const message = "hello";
  return message;
}

console.log(createMessage());
console.log(message); // ReferenceError 발생
```

`message`는 `createMessage` 함수 밖에서 접근할 수 없다.

### 재귀 호출과 스택 오버플로

함수가 자기 자신을 계속 호출하면 호출 스택이 끝없이 쌓일 수 있다.

```javascript
function repeat() {
  repeat();
}

repeat();
```

이 코드는 호출이 종료되지 않아 결국 스택 오버플로 오류를 일으킨다. 재귀 함수는 반드시 종료 조건이 필요하다.

```javascript
function countDown(number) {
  if (number === 0) {
    return;
  }

  console.log(number);
  countDown(number - 1);
}

countDown(3);
```

예상 결과:

```txt
3
2
1
```

## 5. Function Declaration: 함수 선언문

### 기본 문법

함수 선언문(function declaration)은 `function` 키워드와 함수 이름으로 함수를 정의하는 방식이다.

```javascript
function add(a, b) {
  return a + b;
}
```

호출은 다음처럼 한다.

```javascript
console.log(add(2, 3)); // 5
```

### 특징

함수 선언문은 이름이 명확하고, 독립적인 기능을 정의할 때 읽기 쉽다.

```javascript
function calculateTotalPrice(price, quantity) {
  return price * quantity;
}
```

여러 곳에서 재사용할 핵심 기능을 정의할 때 적합하다.

### 함수 선언문과 호이스팅

함수 선언문은 코드상 선언보다 앞에서 호출해도 동작한다.

```javascript
console.log(add(2, 3));

function add(a, b) {
  return a + b;
}
```

예상 결과:

```txt
5
```

이 현상은 호이스팅(hoisting)과 관련 있다. 하지만 실제로 함수 코드가 물리적으로 맨 위로 이동하는 것은 아니다. JavaScript 엔진이 코드를 실행하기 전에 선언을 처리하는 과정 때문에 선언문 함수에 접근할 수 있는 것이다.

> 🔭 **미리보기: 실행 컨텍스트와 호이스팅**
>
> JavaScript는 코드를 한 줄씩 실행하기 전에 실행 환경을 준비하는 단계를 거친다.
>
> 함수 선언문은 이 준비 단계에서 함수와 이름의 연결이 만들어져 선언 이전에도 호출할 수 있다.
>
> `var`, `let`, `const`, 함수 표현식은 같은 방식으로 사용할 수 없거나 주의할 점이 있다.
>
> 실행 컨텍스트와 스코프 학습에서 생성 단계와 실행 단계를 자세히 다룬다.

## 6. Function Expression: 함수 표현식

### 기본 문법

함수 표현식(function expression)은 함수를 하나의 값으로 만들고, 그 값을 변수에 할당하는 방식이다.

```javascript
const add = function (a, b) {
  return a + b;
};

console.log(add(2, 3)); // 5
```

`function (a, b) { ... }`가 함수 값이며, 그 함수 값을 `add` 변수에 저장한다.

### 익명 함수와 이름 있는 함수 표현식

함수 표현식은 보통 이름 없이 작성한다. 이를 익명 함수(anonymous function)라고 한다.

```javascript
const greet = function () {
  console.log("hello");
};
```

함수 자체에 이름을 붙일 수도 있다.

```javascript
const greet = function greetUser() {
  console.log("hello");
};
```

일반적으로 호출할 때는 변수 이름을 사용한다.

```javascript
greet();
```

초기 학습과 대부분의 일반적인 코드에서는 변수 이름과 함수 내부 이름을 다르게 둘 필요가 많지 않다. 다만 디버깅 시 함수 이름이 스택 추적에 도움이 될 수 있다.

### 선언 이전 호출의 차이

함수 표현식을 `const`에 저장했다면, 초기화되기 전에는 호출할 수 없다.

```javascript
console.log(add(2, 3));

const add = function (a, b) {
  return a + b;
};
```

이 코드는 `ReferenceError`를 발생시킨다. `add` 변수는 선언되었지만, `const` 변수는 초기화 전 접근이 금지된 구간에 있기 때문이다.

다음처럼 선언 이후에 호출해야 한다.

```javascript
const add = function (a, b) {
  return a + b;
};

console.log(add(2, 3)); // 5
```

### 함수 표현식이 유용한 이유

JavaScript에서 함수는 값이므로 변수에 저장할 수 있다. 이 특성 덕분에 조건에 따라 다른 함수를 선택할 수 있다.

```javascript
const isMember = true;

const getDiscount = isMember
  ? function () {
      return 0.2;
    }
  : function () {
      return 0;
    };

console.log(getDiscount()); // 0.2
```

여기서는 조건에 따라 `getDiscount` 변수에 서로 다른 함수가 저장된다.

## 7. Arrow Function: 화살표 함수

### 기본 문법

화살표 함수(arrow function)는 함수 표현식을 더 짧게 작성할 수 있도록 ES2015에서 추가된 문법이다.

```javascript
const add = (a, b) => {
  return a + b;
};

console.log(add(2, 3)); // 5
```

화살표 함수는 이름이 있는 독립 선언문이 아니라, 함수 표현식의 한 형태다. 따라서 보통 변수에 할당해서 사용한다.

### 매개변수가 하나일 때

매개변수가 하나라면 괄호를 생략할 수 있다.

```javascript
const square = number => {
  return number * number;
};

console.log(square(4)); // 16
```

괄호를 써도 된다.

```javascript
const square = (number) => {
  return number * number;
};
```

팀 규칙이나 일관성을 위해 매개변수가 하나여도 괄호를 유지하기도 한다.

### 매개변수가 없을 때

매개변수가 없으면 빈 괄호가 필요하다.

```javascript
const greet = () => {
  console.log("hello");
};

greet();
```

### 본문이 한 줄일 때: 암시적 반환

화살표 함수 본문이 하나의 표현식이라면 중괄호와 `return`을 생략할 수 있다. 이때 표현식의 결과가 자동으로 반환된다. 이를 암시적 반환(implicit return)이라고 한다.

```javascript
const add = (a, b) => a + b;

console.log(add(2, 3)); // 5
```

위 코드는 다음과 같은 의미다.

```javascript
const add = (a, b) => {
  return a + b;
};
```

### 중괄호를 사용하면 return을 직접 작성한다

중괄호를 사용한 화살표 함수는 자동 반환하지 않는다.

```javascript
const add = (a, b) => {
  a + b;
};

console.log(add(2, 3)); // undefined
```

위 코드에서 `a + b`는 계산되지만 반환되지 않는다. 값을 반환하려면 `return`이 필요하다.

```javascript
const add = (a, b) => {
  return a + b;
};

console.log(add(2, 3)); // 5
```

### 객체를 암시적으로 반환하기

객체 리터럴을 바로 반환할 때는 객체를 소괄호로 감싸야 한다.

```javascript
const createUser = name => ({
  name: name,
  role: "student"
});

console.log(createUser("Haewon"));
```

예상 결과:

```txt
{ name: "Haewon", role: "student" }
```

소괄호 없이 작성하면 JavaScript는 중괄호를 함수 본문으로 해석한다.

```javascript
const createUser = name => {
  name: name;
};
```

이 함수는 객체를 반환하지 않고 `undefined`를 반환한다.

## 8. 세 방식의 핵심 비교

| 구분 | 함수 선언문 | 함수 표현식 | 화살표 함수 |
| --- | --- | --- | --- |
| 기본 형태 | `function add() {}` | `const add = function () {}` | `const add = () => {}` |
| 함수 이름 | 필수 | 선택 가능, 변수명으로 주로 접근 | 변수명으로 주로 접근 |
| 선언 전 호출 | 가능 | `const`·`let`에서는 불가 | `const`·`let`에서는 불가 |
| `this` | 호출 방식에 따라 결정 | 호출 방식에 따라 결정 | 자신만의 `this`를 만들지 않음 |
| `arguments` 객체 | 제공됨 | 제공됨 | 제공되지 않음 |
| 생성자 호출 `new` | 가능 | 가능 | 불가 |
| 주 사용처 | 이름 있는 독립 기능 | 함수를 값으로 저장·전달 | 짧은 콜백, 간결한 함수 표현 |

세 방식은 완전히 우열이 있는 것이 아니다. 코드의 목적, 문맥, `this` 필요 여부에 따라 선택한다.

## 9. 일반 함수와 화살표 함수의 this

### this란 무엇인가

`this`는 함수가 실행될 때 특정 객체를 가리킬 수 있는 특별한 값이다. 일반 함수의 `this`는 주로 **어떻게 호출했는지**에 따라 달라진다.

```javascript
const user = {
  name: "Haewon",
  introduce: function () {
    console.log(this.name);
  }
};

user.introduce();
```

예상 결과:

```txt
Haewon
```

`user.introduce()`처럼 객체의 메서드로 호출했으므로, 이 경우 `this`는 `user`를 가리킨다.

### 화살표 함수는 자신만의 this를 만들지 않는다

화살표 함수는 호출 시점에 새 `this`를 만들지 않고, 자신이 작성된 바깥쪽 환경의 `this`를 사용한다. 이를 어휘적 `this` 또는 lexical `this`라고 한다.

따라서 객체 메서드를 단순히 화살표 함수로 바꾸면 의도한 객체를 가리키지 못할 수 있다.

```javascript
const user = {
  name: "Haewon",
  introduce: () => {
    console.log(this.name);
  }
};

user.introduce();
```

위 코드에서 화살표 함수 내부의 `this`는 `user`가 아니다. 실행 환경에 따라 `undefined`가 출력되거나 다른 바깥쪽 `this`를 참조할 수 있다.

객체 자신의 속성을 `this`로 읽는 메서드에는 일반 함수 문법이 더 적합하다.

```javascript
const user = {
  name: "Haewon",
  introduce() {
    console.log(this.name);
  }
};

user.introduce();
```

이 문법은 메서드 축약 문법이며, 일반 함수처럼 호출 방식에 따라 `this`가 정해진다.

### 화살표 함수가 유용한 this 상황

화살표 함수는 바깥 함수의 `this`를 그대로 유지해야 할 때 유용하다. 다만 이 주제는 객체, 메서드, 비동기 처리, 클래스 학습과 함께 더 정확히 이해할 수 있다.

> 🔭 **미리보기: 콜백과 this**
>
> 어떤 함수에 다른 함수를 전달하면, 전달된 함수는 나중에 다른 시점이나 다른 방식으로 호출될 수 있다.
>
> 일반 함수는 호출 방식에 따라 `this`가 달라질 수 있어 객체 문맥을 잃는 문제가 생길 수 있다.
>
> 화살표 함수는 바깥쪽 `this`를 유지하므로 이런 상황에서 유용할 수 있다.
>
> 콜백, 이벤트, 클래스 학습에서 구체적인 실행 순서와 해결 패턴을 자세히 다룬다.

## 10. 함수와 값: JavaScript의 중요한 특성

JavaScript에서 함수는 객체이므로 값처럼 다룰 수 있다.

```javascript
const sayHello = function () {
  console.log("hello");
};

const action = sayHello;

action();
```

예상 결과:

```txt
hello
```

`action`에 함수 호출 결과를 넣은 것이 아니라, 함수 자체를 넣었다는 점이 중요하다.

```javascript
const action = sayHello; // 함수를 저장
```

반면 괄호를 붙이면 즉시 호출하고, 반환값을 저장한다.

```javascript
const action = sayHello(); // 함수를 실행한 결과를 저장
```

두 표현은 전혀 다르다.

| 코드 | 의미 |
| --- | --- |
| `const action = sayHello;` | `sayHello` 함수 자체를 저장 |
| `const action = sayHello();` | `sayHello`를 지금 호출한 반환값을 저장 |

## 11. 매개변수와 인자의 동작

### 인자가 부족한 경우

함수 호출 시 필요한 인자를 적게 전달해도 문법 오류가 바로 발생하지는 않는다. 전달되지 않은 매개변수는 `undefined`가 된다.

```javascript
function introduce(name, age) {
  console.log(name, age);
}

introduce("Haewon");
```

예상 결과:

```txt
Haewon undefined
```

이 특성은 편리할 수 있지만, 실무에서는 필수 입력값이 누락되는 버그로 이어질 수 있다.

### 기본 매개변수

기본 매개변수(default parameter)를 사용하면 인자가 없거나 `undefined`일 때 기본값을 쓸 수 있다.

```javascript
function greet(name = "익명") {
  return `안녕하세요, ${name}님.`;
}

console.log(greet());
console.log(greet("해원"));
```

예상 결과:

```txt
안녕하세요, 익명님.
안녕하세요, 해원님.
```

단, `null`은 명시적으로 전달된 값이므로 기본값으로 바뀌지 않는다.

```javascript
function greet(name = "익명") {
  return `안녕하세요, ${name}님.`;
}

console.log(greet(null)); // "안녕하세요, null님."
```

### 인자가 많은 경우

함수 정의보다 많은 인자를 전달해도 일반적으로 오류는 발생하지 않는다.

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(1, 2, 3)); // 3
```

이 함수 본문에서는 `a`와 `b`만 사용하므로 세 번째 인자 `3`은 결과에 반영되지 않는다.

### 나머지 매개변수

나머지 매개변수(rest parameter)는 여러 인자를 배열로 모을 수 있다.

```javascript
function sum(...numbers) {
  let total = 0;

  for (const number of numbers) {
    total += number;
  }

  return total;
}

console.log(sum(1, 2, 3, 4)); // 10
```

`...numbers`는 전달된 나머지 인자들을 배열로 만든다. `for...of`와 반복문은 이후 제어문·반복문 학습에서 더 깊게 다룬다.

## 12. 함수에 객체를 전달할 때

함수 인자로 원시값을 전달하면 값이 복사된 것처럼 독립적으로 동작한다.

```javascript
function increase(number) {
  number = number + 1;
  return number;
}

const score = 10;

console.log(increase(score)); // 11
console.log(score); // 10
```

반면 객체를 전달하면 함수와 호출한 코드가 같은 객체를 통해 접근할 수 있다.

```javascript
function changeName(user) {
  user.name = "Kim";
}

const user = {
  name: "Haewon"
};

changeName(user);

console.log(user.name); // "Kim"
```

JavaScript는 값을 전달한다. 객체를 전달할 때 전달되는 값이 객체의 참조이기 때문에, 함수 안에서도 원래 객체를 수정할 수 있는 것이다.

원본 객체를 바꾸지 않으려면 새 객체를 만들어 반환한다.

```javascript
function changeName(user, name) {
  return {
    ...user,
    name: name
  };
}

const user = {
  name: "Haewon"
};

const updatedUser = changeName(user, "Kim");

console.log(user.name); // "Haewon"
console.log(updatedUser.name); // "Kim"
```

## 13. 코드 예제와 결과 해설

### 예제 1: 함수 호출과 반환값

```javascript
function calculateArea(width, height) {
  return width * height;
}

const area = calculateArea(5, 3);

console.log(area);
```

예상 결과:

```txt
15
```

`calculateArea(5, 3)` 호출로 `width`에는 `5`, `height`에는 `3`이 전달된다. 함수는 곱셈 결과 `15`를 반환하고, 그 값이 `area`에 저장된다.

### 예제 2: console.log와 return의 차이

```javascript
function printSum(a, b) {
  console.log(a + b);
}

const result = printSum(2, 3);

console.log("result:", result);
```

예상 결과:

```txt
5
result: undefined
```

함수는 `5`를 콘솔에 출력했지만 `return`하지 않았다. 따라서 함수 호출의 결과값은 `undefined`다.

### 예제 3: 선언문 함수의 호출 위치

```javascript
console.log(getMessage());

function getMessage() {
  return "hello";
}
```

예상 결과:

```txt
hello
```

함수 선언문은 실행 준비 단계에서 이름과 함수의 연결이 만들어지므로 선언 코드보다 먼저 호출할 수 있다.

### 예제 4: 함수 표현식의 호출 위치

```javascript
console.log(getMessage());

const getMessage = function () {
  return "hello";
};
```

예상 결과:

```txt
ReferenceError
```

`const getMessage`는 선언 이전에 사용할 수 없다. 함수 표현식은 변수 초기화가 끝난 뒤에 호출해야 한다.

### 예제 5: 화살표 함수의 암시적 반환

```javascript
const double = number => number * 2;

console.log(double(5));
```

예상 결과:

```txt
10
```

본문이 하나의 표현식이고 중괄호가 없으므로 `number * 2`의 결과가 자동으로 반환된다.

### 예제 6: 중괄호와 return 누락

```javascript
const double = number => {
  number * 2;
};

console.log(double(5));
```

예상 결과:

```txt
undefined
```

중괄호를 사용하면 함수 본문이 된다. 명시적인 `return`이 없으므로 결과는 `undefined`다.

### 예제 7: 객체 반환

```javascript
const createProduct = (name, price) => ({
  name: name,
  price: price
});

console.log(createProduct("Notebook", 3000));
```

예상 결과:

```txt
{ name: "Notebook", price: 3000 }
```

객체 리터럴을 암시적으로 반환하기 위해 객체 전체를 소괄호로 감쌌다.

### 예제 8: 객체 메서드와 화살표 함수

```javascript
const user = {
  name: "Haewon",
  getName() {
    return this.name;
  }
};

console.log(user.getName());
```

예상 결과:

```txt
Haewon
```

`user.getName()`처럼 호출하면 `getName` 내부의 `this`는 `user`를 가리킨다. 객체 자신의 속성을 `this`로 읽는 메서드에는 일반 함수 또는 메서드 축약 문법을 사용한다.

## 14. 흔한 오해·예외·주의점

### 함수를 호출하지 않고 함수 자체를 전달해야 하는 경우

다음 두 코드는 다르다.

```javascript
const result = calculate;
```

```javascript
const result = calculate();
```

첫 번째는 함수 자체를 저장한다. 두 번째는 지금 함수를 실행하고 반환값을 저장한다. 이벤트 처리나 콜백에서 괄호를 실수로 붙이는 버그가 자주 발생한다.

### return 뒤의 코드는 실행되지 않는다

```javascript
function example() {
  return "완료";
  console.log("여기는 실행되지 않습니다.");
}
```

`return`은 값을 돌려주는 동시에 해당 함수의 실행을 종료한다.

### 화살표 함수가 일반 함수를 완전히 대체하지 않는다

화살표 함수는 짧고 편리하지만, 다음 상황에는 적합하지 않거나 사용할 수 없다.

- 객체 자신을 `this`로 사용해야 하는 일반적인 메서드
- 생성자 함수로 `new`를 사용해야 하는 경우
- 자체 `arguments` 객체가 필요한 경우

따라서 “짧으므로 항상 화살표 함수”가 아니라, 함수가 어떤 `this` 규칙과 호출 방식을 필요로 하는지 판단해야 한다.

### 객체 반환에서 중괄호를 주의한다

```javascript
const createUser = name => { name: name };
```

이 코드는 객체를 반환하지 않는다. 객체를 암시적으로 반환하려면 다음처럼 작성한다.

```javascript
const createUser = name => ({ name: name });
```

### 매개변수 이름은 함수 내부에서만 사용된다

```javascript
function greet(name) {
  return `안녕하세요, ${name}님.`;
}

console.log(name); // ReferenceError 발생
```

매개변수 `name`은 `greet` 함수 호출이 실행되는 동안에만 존재하는 지역 변수다.

### return 없는 함수의 결과는 undefined다

```javascript
function logMessage() {
  console.log("message");
}

const result = logMessage();

console.log(result); // undefined
```

출력과 반환을 혼동하지 않아야 한다. 다른 코드에서 계산 결과를 사용해야 한다면 `console.log`만 하지 말고 `return`해야 한다.

## 15. 실무 적용과 디버깅 관점

### 함수 이름은 역할을 드러내게 작성한다

함수 이름은 “무엇을 하는가”를 드러내야 한다.

```javascript
function getUserName(user) {
  return user.name;
}
```

`doSomething`, `handleData`처럼 지나치게 모호한 이름보다, `calculateTotalPrice`, `validateEmail`, `createUser`처럼 행동과 대상을 함께 표현하는 이름이 유지보수에 유리하다.

### 한 함수에는 한 가지 핵심 책임을 둔다

함수가 입력 검증, 데이터 변환, 화면 출력, 네트워크 요청까지 모두 처리하면 읽기와 테스트가 어려워진다. 하나의 함수가 하나의 분명한 일을 하도록 작게 나누는 편이 좋다.

```javascript
function calculateDiscountedPrice(price, discountRate) {
  return price * (1 - discountRate);
}
```

계산 함수는 콘솔 출력이나 외부 상태 변경 없이 입력을 받아 결과를 반환하도록 만들면 재사용과 테스트가 쉬워진다.

### return 값과 호출 위치를 확인한다

함수 결과가 `undefined`라면 다음을 확인한다.

- 함수 안에 `return`이 있는가
- 모든 실행 경로에서 필요한 값이 반환되는가
- 함수 자체를 저장하려던 것인지, 호출 결과를 저장하려던 것인지
- 중괄호를 추가하면서 화살표 함수의 암시적 반환을 잃지 않았는가

```javascript
const getPrice = () => {
  1000;
};

console.log(getPrice()); // undefined
```

위 코드의 문제는 `return 1000;`이 없다는 것이다.

### 공유 객체 변경을 피한다

함수가 받은 객체를 직접 수정하면 호출한 코드의 데이터도 바뀔 수 있다.

```javascript
function setAdmin(user) {
  user.role = "admin";
  return user;
}
```

원본 보존이 필요한 경우 새 객체를 반환한다.

```javascript
function setAdmin(user) {
  return {
    ...user,
    role: "admin"
  };
}
```

함수 이름과 문서화로 “원본을 변경하는 함수인지” 또는 “새 값을 반환하는 함수인지”를 분명히 하는 습관이 중요하다.

---

## 16. 핵심 정리

> ✅ - 함수는 작업을 묶고 재사용하기 위한 코드 단위이며, 정의와 호출은 다르다.

- 매개변수는 함수 정의의 입력 변수이고, 인자는 호출할 때 전달하는 실제 값이다.
- `return`은 값을 호출한 위치로 돌려주고, 동시에 함수 실행을 끝낸다.
- `console.log()`는 출력일 뿐 반환이 아니다.
- 함수 선언문은 선언보다 앞에서 호출할 수 있지만, `const` 또는 `let`에 저장한 함수 표현식과 화살표 함수는 초기화 뒤에 호출해야 한다.
- 화살표 함수는 함수 표현식의 간결한 문법이며, 한 줄 표현식에서는 암시적 반환을 사용할 수 있다.
- 중괄호를 쓰는 화살표 함수는 `return`을 직접 작성해야 한다.
- 객체를 암시적으로 반환하려면 `({ key: value })`처럼 소괄호로 감싼다.
- 일반 함수의 `this`는 호출 방식에 따라 정해지고, 화살표 함수는 자신만의 `this`를 만들지 않는다.
- 객체 자신의 속성을 `this`로 사용하는 메서드에는 일반 함수 또는 메서드 축약 문법이 적합하다.
- 객체 인자는 참조를 통해 같은 객체를 수정할 수 있으므로, 원본 변경 여부를 의도적으로 설계해야 한다.

---

## 17. 이해도 확인 질문

> 📝 답을 정리하며 개념을 점검해 보세요. 코드가 있는 문항은 실행 결과를 먼저 예측한 뒤 직접 확인해 보세요.

<details>
<summary>확인 질문 전체 보기 (35문항)</summary>

1. 함수가 필요한 이유를 코드 중복, 역할 분리, 재사용 관점에서 설명해 보세요.
2. 함수 정의와 함수 호출의 차이는 무엇인가요?
3. 매개변수와 인자의 차이를 예시와 함께 설명해 보세요.
4. `return`의 역할은 무엇인가요?
5. `return`을 실행한 뒤 함수 내부의 아래 코드는 왜 실행되지 않나요?
6. `console.log()`와 `return`의 차이를 설명해 보세요.
7. 다음 코드의 출력 결과를 예측해 보세요.
   ```javascript
   function add(a, b) {
     console.log(a + b);
   }

   const result = add(2, 3);

   console.log(result);
   ```
8. 반환값이 없는 함수 호출의 결과는 무엇인가요?
9. 함수 선언문을 작성하는 기본 문법을 적어 보세요.
10. 함수 표현식을 작성하는 기본 문법을 적어 보세요.
11. 화살표 함수를 작성하는 기본 문법을 적어 보세요.
12. 다음 코드가 실행되는 이유를 설명해 보세요.
    ```javascript
    console.log(add(1, 2));

    function add(a, b) {
      return a + b;
    }
    ```
13. 다음 코드에서 발생하는 오류의 종류와 원인을 설명해 보세요.
    ```javascript
    console.log(add(1, 2));

    const add = function (a, b) {
      return a + b;
    };
    ```
14. 함수 선언문과 함수 표현식의 선언 전 호출 가능 여부가 다른 이유를 설명해 보세요.
15. 화살표 함수에서 매개변수가 하나일 때 생략할 수 있는 문법은 무엇인가요?
16. 화살표 함수에서 매개변수가 없을 때 필요한 문법은 무엇인가요?
17. 다음 두 함수가 같은 결과를 반환하는지 판단하고 이유를 설명해 보세요.
    ```javascript
    const a = x => x * 2;

    const b = x => {
      return x * 2;
    };
    ```
18. 다음 함수가 `undefined`를 반환하는 이유를 설명해 보세요.
    ```javascript
    const double = x => {
      x * 2;
    };
    ```
19. 객체를 암시적으로 반환하는 화살표 함수를 작성해 보세요.
20. 다음 코드가 의도한 객체를 반환하지 않는 이유를 설명해 보세요.
    ```javascript
    const createUser = name => {
      name: name
    };
    ```
21. 함수는 JavaScript에서 값이라는 말이 무엇을 뜻하는지 설명해 보세요.
22. 다음 두 코드의 차이를 설명해 보세요.
    ```javascript
    const action = greet;
    ```

    ```javascript
    const action = greet();
    ```
23. 함수 호출 시 인자를 하나도 전달하지 않으면, 해당 매개변수는 어떤 값을 가지나요?
24. 기본 매개변수는 어떤 문제를 줄이는 데 도움이 되나요?
25. 다음 코드의 출력 결과를 예측해 보세요.
    ```javascript
    function greet(name = "익명") {
      return `안녕하세요, ${name}님.`;
    }

    console.log(greet());
    console.log(greet(null));
    ```
26. 함수 정의보다 많은 인자를 전달하면 기본적으로 어떤 일이 일어나나요?
27. 나머지 매개변수 `...numbers`는 무엇을 하나요?
28. 다음 코드에서 호출 후 `score`가 10인 이유를 설명해 보세요.
    ```javascript
    function increase(number) {
      number = number + 1;
      return number;
    }

    const score = 10;

    increase(score);

    console.log(score);
    ```
29. 다음 코드에서 호출 후 `user.name`이 `"Kim"`인 이유를 설명해 보세요.
    ```javascript
    function changeName(user) {
      user.name = "Kim";
    }

    const user = { name: "Haewon" };

    changeName(user);

    console.log(user.name);
    ```
30. 원본 객체를 수정하지 않고 이름만 변경한 새 객체를 반환하는 함수를 작성해 보세요.
31. 일반 함수에서 `this`는 주로 무엇에 따라 달라지나요?
32. 객체의 메서드에서 화살표 함수를 무분별하게 사용하면 문제가 될 수 있는 이유는 무엇인가요?
33. 화살표 함수가 자신만의 `this`를 만들지 않는다는 말은 무엇을 뜻하나요?
34. 함수 호출이 중첩될 때 호출 스택에는 어떤 순서로 함수가 쌓이고 제거되나요?
35. 함수가 계속 자기 자신을 호출하고 종료 조건이 없다면 어떤 문제가 발생하며, 이를 막기 위해 무엇이 필요한가요?

</details>
