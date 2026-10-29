---
track: A
notion: https://app.notion.com/p/400178c4afac80c6a04bedec2bad011a
---

# 변수와 자료형 · Primitive / Reference

> 📍 **학습 범위**: JavaScript 기초 · 변수와 자료형
>
> 값, 변수, 원시값과 참조 공유의 차이를 순서대로 살펴본다.
>
> 함수와 인자 전달: [함수 · Function Declaration / Expression / Arrow Function](functions.md)

- [1. 학습 목표](#1-학습-목표)
- [2. 이 주제를 왜 배우는가](#2-이-주제를-왜-배우는가)
- [3. 핵심 개념과 용어](#3-핵심-개념과-용어)
- [4. 변수 선언: let, const, var](#4-변수-선언-let-const-var)
- [5. 자료형이란 무엇인가](#5-자료형이란-무엇인가)
- [6. JavaScript의 자료형 분류](#6-javascript의-자료형-분류)
- [7. Primitive: 원시값](#7-primitive-원시값)
- [8. Reference: 참조값](#8-reference-참조값)
- [9. 대입: 값 복사와 참조 복사](#9-대입-값-복사와-참조-복사)
- [10. 동등 비교: ===와 객체](#10-동등-비교-와-객체)
- [11. 배열도 참조값이다](#11-배열도-참조값이다)
- [12. 얕은 복사: 새로운 바깥 객체 만들기](#12-얕은-복사-새로운-바깥-객체-만들기)
- [13. 동작 원리: 변수와 메모리 모델](#13-동작-원리-변수와-메모리-모델)
- [14. 코드 예제와 결과 해설](#14-코드-예제와-결과-해설)
- [15. 흔한 오해·예외·주의점](#15-흔한-오해예외주의점)
- [16. 실무 적용과 디버깅 관점](#16-실무-적용과-디버깅-관점)
- [17. 핵심 정리](#17-핵심-정리)
- [18. 이해도 확인 질문](#18-이해도-확인-질문)

## 1. 학습 목표

이 페이지를 마치면 다음을 할 수 있어야 한다.

- 변수와 값이 무엇인지 구분해 설명할 수 있다.
- `let`, `const`, `var`의 기본적인 차이와 사용 기준을 설명할 수 있다.
- JavaScript의 원시값(Primitive)과 참조값(Reference)의 차이를 설명할 수 있다.
- 대입과 비교에서 값이 복사되는 경우와 참조가 복사되는 경우를 구분할 수 있다.
- 객체나 배열을 수정했을 때 원래 변수에도 변화가 보이는 이유를 설명할 수 있다.
- 참조 공유로 인한 버그를 발견하고 기본적인 방식으로 디버깅할 수 있다.

---

## 2. 이 주제를 왜 배우는가

프로그램은 데이터를 저장하고, 읽고, 변경하면서 동작한다. 이름을 붙여 데이터를 다루는 방법이 변수이며, 데이터의 종류와 저장·전달 방식은 자료형이 결정한다.

JavaScript에서 특히 중요한 점은 모든 값이 같은 방식으로 복사되지 않는다는 것이다. 숫자, 문자열, 불리언 같은 원시값은 값 자체가 복사되는 것처럼 동작하지만, 객체와 배열 같은 참조값은 같은 객체를 가리키는 참조가 복사된다. 이 차이를 모르면 “분명 다른 변수인데 왜 함께 바뀌지?” 같은 버그를 자주 만나게 된다.

이 주제는 이후 배우게 될 함수 인자 전달, 객체, 배열 메서드, 상태 관리, 비동기 코드의 데이터 변경 문제를 이해하는 기반이 된다.

## 3. 핵심 개념과 용어

### 값(Value)

값은 프로그램이 다루는 데이터 자체다. 예를 들어 `10`, `"hello"`, `true`는 모두 값이다.

```javascript
10
"hello"
true
```

값은 계산 결과일 수도 있다.

```javascript
2 + 3 // 5라는 값이 된다.
```

### 변수(Variable)

변수는 값을 기억하거나 다시 찾아오기 위해 붙이는 이름이다.

```javascript
let age = 20;
```

위 코드에서:

- `age`는 변수 이름이다.
- `20`은 값이다.
- `=`는 오른쪽의 값을 왼쪽 변수에 대입한다는 뜻이다.

변수는 값을 담는 상자라고 비유할 수 있지만, JavaScript의 실제 동작을 이해할 때는 **값에 접근하기 위한 이름 또는 바인딩(binding)** 이라고 생각하는 편이 더 정확하다.

### 선언(Declaration), 초기화(Initialization), 할당(Assignment)

변수와 관련된 동작은 세 단계로 나누어 생각할 수 있다.

```javascript
let score;       // 선언
score = 100;     // 할당
```

`let score = 100;`처럼 한 줄에 쓰면 선언과 초기화를 함께 수행한다.

- **선언**: `score`라는 변수를 만든다.
- **초기화**: 변수에 처음 사용할 값을 준비한다.
- **할당**: 변수에 값을 넣거나 기존 값을 바꾼다.

```javascript
let score = 80;
score = 100;

console.log(score); // 100
```

두 번째 줄은 `score`라는 변수 자체를 새로 만드는 것이 아니라, 기존 변수에 새 값을 할당한다.

### 식별자(Identifier)

변수, 함수, 클래스처럼 프로그램의 대상을 가리키는 이름을 식별자라고 한다.

```javascript
let userName = "haewon";
const MAX_COUNT = 10;
```

식별자 규칙은 다음과 같다.

- 영문자, 숫자, `_`, `$`를 사용할 수 있다.
- 숫자로 시작할 수 없다.
- 공백이나 를 사용할 수 없다.
- JavaScript 예약어는 사용할 수 없다.
- 대소문자를 구분한다.

```javascript
let userName = "haewon";
let username = "other";

console.log(userName); // "haewon"
console.log(username); // "other"
```

`userName`과 `username`은 서로 다른 식별자다.

## 4. 변수 선언: let, const, var

### let

`let`은 재할당이 가능한 변수를 선언할 때 사용한다.

```javascript
let score = 80;

score = 90;

console.log(score); // 90
```

점수처럼 실행 중 값이 달라질 수 있는 데이터에 적합하다.

### const

`const`는 **변수 자체에 대한 재할당을 금지**한다. 선언과 동시에 초기값을 반드시 지정해야 한다.

```javascript
const birthYear = 2000;

birthYear = 2001; // TypeError 발생
```

다음 코드는 오류다.

```javascript
const name; // SyntaxError 발생
```

`const`는 “값이 절대 변하지 않는다”가 아니라, “이 변수에 다른 값을 다시 대입할 수 없다”는 뜻이다.

```javascript
const user = {
  name: "Haewon"
};

user.name = "Kim";

console.log(user.name); // "Kim"
```

객체 내부의 속성은 바꿀 수 있다. `user` 변수에 완전히 다른 객체를 할당하는 것만 금지된다.

```javascript
const user = {
  name: "Haewon"
};

user = {
  name: "Kim"
}; // TypeError 발생
```

### var

`var`는 과거 JavaScript에서 사용하던 변수 선언 방식이다. 현대 JavaScript 코드에서는 특별한 이유가 없다면 `let`과 `const`를 사용한다.

```javascript
var count = 1;
var count = 2;

console.log(count); // 2
```

`var`는 같은 스코프에서 같은 이름으로 다시 선언할 수 있고, 블록 스코프 규칙도 `let`, `const`와 다르다. 이런 특성은 의도하지 않은 변수 변경과 버그를 만들기 쉽다.

```javascript
if (true) {
  var message = "hello";
}

console.log(message); // "hello"
```

반면 `let`과 `const`는 블록 내부에서 선언하면 블록 밖에서 접근할 수 없다.

```javascript
if (true) {
  const message = "hello";
}

console.log(message); // ReferenceError 발생
```

실무의 기본 기준은 다음과 같다.

- 기본적으로 `const`를 사용한다.
- 재할당이 꼭 필요할 때만 `let`을 사용한다.
- 새 코드에서는 `var`를 사용하지 않는다.

## 5. 자료형이란 무엇인가

자료형(type)은 값의 종류와 그 값에 가능한 연산을 구분하는 분류다.

```javascript
const age = 20;
const name = "Haewon";
const isStudent = true;
```

각 값은 서로 다른 성질을 가진다.

- 숫자 `20`에는 산술 연산을 할 수 있다.
- 문자열 `"Haewon"`에는 문자열을 이어 붙이는 연산을 할 수 있다.
- 불리언 `true`는 조건 판단에 사용한다.

JavaScript는 **동적 타입 언어**다. 변수 선언 시 자료형을 미리 적지 않아도 되고, 같은 변수에 다른 종류의 값을 다시 할당할 수도 있다.

```javascript
let value = 10;
value = "ten";

console.log(value); // "ten"
```

이 특성은 빠르게 코드를 작성하기 편하지만, 의도와 다른 자료형이 들어오는 오류를 런타임에 만날 수 있다는 뜻이기도 하다.

## 6. JavaScript의 자료형 분류

JavaScript 값은 크게 두 범주로 구분할 수 있다.

| 분류 | 자료형 | 예시 | 핵심 특징 |
| --- | --- | --- | --- |
| 원시값 | `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null` | `"text"`, `42`, `true` | 값 자체로 다루며 변경할 수 없다 |
| 참조값 | 객체(Object) | `{}`, `[]`, `function () {}` | 객체가 있는 위치를 가리키는 참조를 통해 다룬다 |

함수도 JavaScript에서는 객체로 취급된다. 함수는 다음 페이지에서 별도로 깊게 다룬다.

## 7. Primitive: 원시값

### 원시값의 종류

JavaScript의 원시값은 7가지다.

| 자료형 | 예시 | 설명 |
| --- | --- | --- |
| `string` | `"hello"` | 텍스트 데이터 |
| `number` | `42`, `3.14` | 정수와 실수를 함께 표현하는 숫자 |
| `bigint` | `123n` | 매우 큰 정수 |
| `boolean` | `true`, `false` | 참과 거짓 |
| `undefined` | `undefined` | 값이 아직 정해지지 않았음을 나타냄 |
| `null` | `null` | 의도적으로 “값 없음”을 나타냄 |
| `symbol` | `Symbol("id")` | 고유한 식별자 값 |

처음에는 `string`, `number`, `boolean`, `undefined`, `null`을 가장 자주 사용하게 된다.

### string: 문자열

문자열은 텍스트를 표현하는 값이다. 작은따옴표, 큰따옴표, 백틱으로 만들 수 있다.

```javascript
const greeting1 = "hello";
const greeting2 = 'hello';
const greeting3 = `hello`;
```

문자열은 변경할 수 없는 값이다. 이를 **불변(immutable)** 이라고 한다.

```javascript
let text = "hello";

text[0] = "H";

console.log(text); // "hello"
```

문자열의 첫 글자를 바꾸려 해도 원래 문자열은 바뀌지 않는다. 새 문자열을 만들어 변수에 다시 할당해야 한다.

```javascript
let text = "hello";

text = "Hello";

console.log(text); // "Hello"
```

### number: 숫자

JavaScript의 `number`는 정수와 실수를 모두 표현한다.

```javascript
const integer = 10;
const decimal = 3.14;

console.log(integer + decimal); // 13.14
```

나눗셈의 결과도 항상 `number`다.

```javascript
console.log(5 / 2); // 2.5
```

계산 결과가 정상적인 숫자가 아닐 때 `NaN`이 나타날 수 있다.

```javascript
console.log("hello" * 2); // NaN
```

`NaN`은 “Not a Number”라는 이름이지만, JavaScript에서는 `number` 자료형에 속한다.

```javascript
console.log(typeof NaN); // "number"
```

### boolean: 불리언

불리언은 `true` 또는 `false` 둘 중 하나만 가질 수 있는 값이다. 조건을 판단할 때 사용한다.

```javascript
const isLoggedIn = true;
const hasPermission = false;
```

```javascript
if (isLoggedIn) {
  console.log("로그인한 사용자입니다.");
}
```

`if`의 세부 동작과 조건식은 제어문 학습에서 더 깊게 다룬다.

### undefined

`undefined`는 값이 정해지지 않았거나, 아직 값을 할당하지 않았다는 상태를 나타낸다.

```javascript
let nickname;

console.log(nickname); // undefined
```

함수가 명시적으로 값을 반환하지 않을 때도 결과는 `undefined`다.

```javascript
function printHello() {
  console.log("hello");
}

const result = printHello();

console.log(result); // undefined
```

### null

`null`은 개발자가 의도적으로 “값이 없음”을 나타낼 때 사용하는 값이다.

```javascript
let selectedUser = null;
```

예를 들어 아직 선택된 사용자가 없음을 표현할 수 있다.

```javascript
let selectedUser = null;

selectedUser = {
  name: "Haewon"
};
```

`undefined`와 `null`은 모두 값이 없다는 상황과 관련 있지만, 일반적인 의도는 다르다.

- `undefined`: 값이 아직 할당되지 않았거나, JavaScript가 기본으로 넣은 상태
- `null`: 개발자가 명시적으로 비어 있음을 표현한 상태

### typeof와 null의 예외

`typeof` 연산자는 값의 자료형을 문자열로 알려 준다.

```javascript
console.log(typeof "hello"); // "string"
console.log(typeof 42); // "number"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
```

하지만 `null`에는 오래된 JavaScript 호환성 문제로 인한 예외가 있다.

```javascript
console.log(typeof null); // "object"
```

`null`은 객체가 아니다. `typeof null === "object"`는 JavaScript 초창기부터 유지된 역사적인 오류다. 따라서 `null`인지 확인하려면 직접 비교해야 한다.

```javascript
const value = null;

console.log(value === null); // true
```

### 원시값의 불변성

원시값 자체는 수정할 수 없다. 다만 `let`으로 선언된 변수는 다른 원시값을 다시 가리키도록 재할당할 수 있다.

```javascript
let score = 10;

score = 20;

console.log(score); // 20
```

위 코드는 숫자 `10`을 수정한 것이 아니다. `score`가 이전에는 `10`을 가리키다가, 이후에는 `20`이라는 다른 값을 가리키게 된 것이다.

## 8. Reference: 참조값

### 객체란 무엇인가

객체는 여러 데이터를 하나의 단위로 묶어 표현하는 값이다.

```javascript
const user = {
  name: "Haewon",
  age: 20
};
```

위 객체는 `name`, `age`라는 속성(property)을 가진다.

```javascript
console.log(user.name); // "Haewon"
console.log(user.age); // 20
```

배열도 객체의 한 종류다.

```javascript
const fruits = ["apple", "banana", "orange"];
```

객체, 배열, 함수는 원시값과 달리 참조 방식으로 다뤄진다.

### 참조란 무엇인가

객체가 만들어지면 그 객체는 메모리 어딘가에 존재한다. 변수에는 객체 전체가 복사되어 들어간다기보다, 그 객체를 찾아갈 수 있는 참조가 저장된다고 이해하면 된다.

```javascript
const user = {
  name: "Haewon"
};
```

개념적으로 다음처럼 생각할 수 있다.

```txt
user ──> { name: "Haewon" }
```

이 구조 때문에 하나의 객체를 여러 변수가 함께 참조할 수 있다.

```javascript
const originalUser = {
  name: "Haewon"
};

const copiedUser = originalUser;
```

개념적으로는 다음과 같다.

```txt
originalUser ──┐
               ├──> { name: "Haewon" }
copiedUser ────┘
```

## 9. 대입: 값 복사와 참조 복사

### 원시값 대입

원시값을 다른 변수에 대입하면 값이 복사된다.

```javascript
let originalScore = 80;
let copiedScore = originalScore;

copiedScore = 100;

console.log(originalScore); // 80
console.log(copiedScore); // 100
```

`copiedScore`를 바꿔도 `originalScore`는 변하지 않는다. 각 변수가 서로 독립적인 숫자 값을 가지기 때문이다.

### 참조값 대입

객체를 다른 변수에 대입하면 객체 자체가 복제되는 것이 아니라 참조가 복사된다.

```javascript
const originalUser = {
  name: "Haewon"
};

const copiedUser = originalUser;

copiedUser.name = "Kim";

console.log(originalUser.name); // "Kim"
console.log(copiedUser.name); // "Kim"
```

두 변수가 같은 객체를 참조하기 때문에 `copiedUser.name`을 바꾸면 `originalUser.name`으로 접근해도 변경된 값이 보인다.

중요한 사실은 `const`로 선언했더라도 객체 내부를 바꿀 수 있다는 점이다.

```javascript
const user = {
  name: "Haewon"
};

user.name = "Kim"; // 가능
```

하지만 변수에 새 객체를 할당하는 것은 불가능하다.

```javascript
const user = {
  name: "Haewon"
};

user = {
  name: "Kim"
}; // TypeError 발생
```

## 10. 동등 비교: ===와 객체

### 원시값 비교

원시값은 값 자체를 비교할 수 있다.

```javascript
console.log(10 === 10); // true
console.log("hello" === "hello"); // true
console.log(true === false); // false
```

`===`는 엄격한 동등 비교 연산자다. 값과 자료형이 모두 같을 때만 `true`를 반환한다.

```javascript
console.log(10 === "10"); // false
```

### 참조값 비교

객체끼리 `===`로 비교하면 내부 내용이 같은지가 아니라 **같은 객체를 참조하는지**를 확인한다.

```javascript
const user1 = {
  name: "Haewon"
};

const user2 = {
  name: "Haewon"
};

console.log(user1 === user2); // false
```

두 객체의 속성은 같아 보이지만, 서로 별개로 생성된 객체다.

```txt
user1 ──> { name: "Haewon" }
user2 ──> { name: "Haewon" }
```

반면 같은 객체의 참조를 복사했다면 `true`다.

```javascript
const user1 = {
  name: "Haewon"
};

const user2 = user1;

console.log(user1 === user2); // true
```

```txt
user1 ──┐
        ├──> { name: "Haewon" }
user2 ──┘
```

## 11. 배열도 참조값이다

배열도 객체이므로 대입 시 참조가 복사된다.

```javascript
const originalFruits = ["apple", "banana"];
const copiedFruits = originalFruits;

copiedFruits.push("orange");

console.log(originalFruits); // ["apple", "banana", "orange"]
console.log(copiedFruits); // ["apple", "banana", "orange"]
```

`push()`는 배열 자체를 변경하는 메서드다. 두 변수는 같은 배열을 참조하므로 결과가 함께 변한다.

## 12. 얕은 복사: 새로운 바깥 객체 만들기

### 객체 펼침 문법

객체를 독립적으로 다루고 싶다면 새 객체를 만들어야 한다. 가장 기본적인 방법 중 하나는 펼침 문법(spread syntax) `...`다.

```javascript
const originalUser = {
  name: "Haewon",
  age: 20
};

const copiedUser = {
  ...originalUser
};

copiedUser.name = "Kim";

console.log(originalUser.name); // "Haewon"
console.log(copiedUser.name); // "Kim"
```

`copiedUser`는 새 객체이므로 최상위 속성을 바꿔도 `originalUser`는 바뀌지 않는다.

### 배열 펼침 문법

배열도 같은 방식으로 새 배열을 만들 수 있다.

```javascript
const originalFruits = ["apple", "banana"];
const copiedFruits = [...originalFruits];

copiedFruits.push("orange");

console.log(originalFruits); // ["apple", "banana"]
console.log(copiedFruits); // ["apple", "banana", "orange"]
```

### 얕은 복사의 한계

펼침 문법은 최상위 수준만 새로 복사한다. 객체 내부에 객체나 배열이 있다면, 내부 값의 참조는 여전히 공유될 수 있다. 이를 **얕은 복사(shallow copy)** 라고 한다.

```javascript
const originalUser = {
  name: "Haewon",
  address: {
    city: "Busan"
  }
};

const copiedUser = {
  ...originalUser
};

copiedUser.address.city = "Seoul";

console.log(originalUser.address.city); // "Seoul"
console.log(copiedUser.address.city); // "Seoul"
```

바깥쪽 `copiedUser`는 새 객체지만, `address`는 기존 내부 객체를 함께 참조한다.

```txt
originalUser ──> { name, address ──> { city: "Busan" } }
copiedUser ────> { name, address ──> 같은 내부 객체 }
```

깊은 복사(deep copy)는 중첩된 내부 객체까지 별도로 복사하는 것을 말한다. 어떤 방식을 써야 하는지는 데이터 구조와 환경에 따라 달라진다.

## 13. 동작 원리: 변수와 메모리 모델

> 💡 변수와 메모리의 실제 구현은 JavaScript 엔진마다 다를 수 있다. 따라서 “원시값은 무조건 스택, 객체는 무조건 힙”처럼 단정하는 설명은 정확하지 않다.

학습을 위한 개념 모델로는 다음을 이해하면 좋다.

1. JavaScript 엔진은 변수를 선언하면 식별자와 값을 연결한다.
2. 원시값을 대입하면 독립적인 값으로 동작한다.
3. 객체를 대입하면 객체를 찾을 수 있는 참조가 복사된다.
4. 여러 변수가 같은 참조를 가지면 한 객체의 변경 결과를 공유한다.
5. 어떤 변수도 더 이상 객체에 접근할 수 없게 되면, 엔진은 나중에 그 객체의 메모리를 정리할 수 있다.

마지막 과정은 가비지 컬렉션(garbage collection)과 관련 있다.

> 🔭 **미리보기: 가비지 컬렉션**
>
> JavaScript 엔진은 더 이상 접근할 수 없는 객체를 자동으로 찾아 메모리를 회수한다.
>
> 이 과정은 개발자가 직접 해제 명령을 호출하지 않아도 일어난다.
>
> 다만 참조가 남아 있으면 더 이상 필요하지 않은 객체도 메모리에 남을 수 있다.
>
> 이 개념은 이후 메모리 관리와 성능 디버깅 학습에서 자세히 다룬다.

## 14. 코드 예제와 결과 해설

### 예제 1: 원시값은 독립적으로 변경된다

```javascript
let first = 1;
let second = first;

second = 2;

console.log(first);
console.log(second);
```

예상 결과:

```txt
1
2
```

`second = first`에서 숫자 `1`이 복사된다. 이후 `second`에 `2`를 할당해도 `first`가 가리키는 값은 바뀌지 않는다.

### 예제 2: 객체는 같은 대상을 공유할 수 있다

```javascript
const firstUser = {
  name: "Haewon"
};

const secondUser = firstUser;

secondUser.name = "Kim";

console.log(firstUser.name);
console.log(secondUser.name);
```

예상 결과:

```txt
Kim
Kim
```

`firstUser`와 `secondUser`가 같은 객체를 참조한다. 따라서 `secondUser.name`을 변경한 것은 공유 중인 하나의 객체를 변경한 것이다.

### 예제 3: 객체를 새로 만들면 참조가 분리된다

```javascript
const firstUser = {
  name: "Haewon"
};

const secondUser = {
  ...firstUser
};

secondUser.name = "Kim";

console.log(firstUser.name);
console.log(secondUser.name);
```

예상 결과:

```txt
Haewon
Kim
```

펼침 문법으로 새 객체를 생성했기 때문에 두 변수는 서로 다른 객체를 참조한다.

### 예제 4: const와 객체 변경

```javascript
const settings = {
  darkMode: false
};

settings.darkMode = true;

console.log(settings.darkMode);
```

예상 결과:

```txt
true
```

`settings`에 다른 객체를 재할당한 것이 아니라, 같은 객체의 `darkMode` 속성을 바꿨기 때문에 가능하다.

### 예제 5: null 확인

```javascript
const selectedUser = null;

console.log(typeof selectedUser);
console.log(selectedUser === null);
```

예상 결과:

```txt
object
true
```

`typeof null`이 `"object"`를 반환하는 것은 JavaScript의 오래된 예외다. `null` 여부는 `=== null`로 확인한다.

## 15. 흔한 오해·예외·주의점

### const는 객체를 완전히 불변으로 만들지 않는다

`const`는 변수 재할당을 막을 뿐, 객체 내부 속성 변경을 막지 않는다.

```javascript
const profile = {
  nickname: "haewon"
};

profile.nickname = "study-js"; // 가능
```

객체 자체의 변경을 막는 방법은 별도의 개념과 도구가 필요하다. 지금 단계에서는 `const`의 보호 범위가 “변수 재할당”이라는 점을 정확히 기억하면 된다.

### 객체 리터럴이 같아 보여도 서로 다른 객체다

```javascript
console.log({ name: "Haewon" } === { name: "Haewon" }); // false
```

객체 리터럴을 두 번 작성하면 객체가 두 번 생성된다. 내부 속성이 같더라도 참조가 다르므로 `===` 결과는 `false`다.

### == 대신 ===를 기본으로 사용한다

느슨한 동등 비교 연산자 `==`는 서로 다른 자료형을 비교할 때 자동 변환을 수행한다.

```javascript
console.log(0 == false); // true
console.log("" == false); // true
```

이런 자동 변환은 읽는 사람의 예측을 어렵게 한다. 특별한 이유가 없다면 `===`와 `!==`를 기본으로 사용한다.

```javascript
console.log(0 === false); // false
```

### 배열도 객체다

```javascript
console.log(typeof []); // "object"
console.log(typeof {}); // "object"
```

배열인지 확인해야 한다면 `typeof`가 아니라 `Array.isArray()`를 사용한다.

```javascript
console.log(Array.isArray([])); // true
console.log(Array.isArray({})); // false
```

### 얕은 복사는 중첩 객체까지 복사하지 않는다

```javascript
const a = {
  nested: {
    value: 1
  }
};

const b = {
  ...a
};

b.nested.value = 2;

console.log(a.nested.value); // 2
```

`...a`는 바깥 객체만 새로 만든다. 중첩된 `nested` 객체는 계속 공유된다.

## 16. 실무 적용과 디버깅 관점

### 상태를 변경하기 전, 공유 참조를 의심한다

객체나 배열을 수정한 뒤 예상하지 못한 다른 화면·다른 변수·다른 로직의 결과까지 바뀌었다면, 먼저 같은 객체를 공유하는지 확인한다.

```javascript
console.log(userA === userB);
```

결과가 `true`라면 두 변수는 같은 객체를 참조한다.

### 변경 전후 값을 확인한다

객체 변경 버그는 `console.log()`로 변경 전후를 분명하게 기록하며 추적할 수 있다.

```javascript
console.log("변경 전:", user);

user.name = "Kim";

console.log("변경 후:", user);
```

브라우저 개발자 도구나 Node.js 디버거에서는 중단점을 설정해 변수의 참조 관계와 속성 변경 시점을 확인할 수 있다.

### 입력 데이터를 직접 수정하지 않는다

함수나 컴포넌트가 전달받은 객체·배열을 직접 수정하면, 호출한 쪽도 예상치 못한 영향을 받을 수 있다.

```javascript
function addRole(user) {
  user.role = "student";
  return user;
}
```

위 함수는 전달받은 원본 객체를 바꾼다. 원본을 보존해야 한다면 새 객체를 만들어 반환하는 방식이 더 안전하다.

```javascript
function addRole(user) {
  return {
    ...user,
    role: "student"
  };
}
```

이 방식도 중첩된 구조에는 얕은 복사라는 한계가 있다는 점을 기억해야 한다.

---

## 17. 핵심 정리

> ✅ - 변수는 값을 다시 찾아오기 위한 이름이며, 값은 프로그램이 다루는 데이터다.

- `const`는 재할당을 막고, 객체 내부 속성 변경까지 막지는 않는다.
- `let`은 재할당이 필요한 경우에 사용하며, 새 코드에서는 `var`보다 `let`과 `const`를 사용한다.
- JavaScript의 원시값은 `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`이다.
- 객체, 배열, 함수는 참조 방식으로 다뤄지는 객체다.
- 원시값 대입은 독립적인 값처럼 동작하지만, 객체 대입은 같은 객체를 가리키는 참조를 공유할 수 있다.
- 객체의 `===` 비교는 내용이 같은지가 아니라 같은 객체를 참조하는지 확인한다.
- 펼침 문법 `...`은 새 바깥 객체나 배열을 만들 수 있지만, 중첩 객체까지 복사하지는 않는 얕은 복사다.
- `typeof null`은 `"object"`라는 역사적 예외가 있으므로, `null`은 `value === null`로 검사한다.

---

## 18. 이해도 확인 질문

> 📝 답을 정리하며 개념을 점검해 보세요. 코드가 있는 문항은 실행 결과를 먼저 예측한 뒤 직접 확인해 보세요.

<details>
<summary>확인 질문 전체 보기 (35문항)</summary>

1. 변수와 값의 차이를 자신의 말로 설명해 보세요.
2. `let`과 `const`를 선택하는 기준은 무엇인가요?
3. `const`로 선언한 객체의 속성은 왜 변경할 수 있나요?
4. `const` 변수에 재할당하려 할 때 어떤 일이 발생하나요?
5. `var`보다 `let`과 `const`를 권장하는 이유를 설명해 보세요.
6. JavaScript의 원시값 7가지를 적어 보세요.
7. `undefined`와 `null`은 어떤 의도 차이가 있나요?
8. `typeof null`의 결과가 `"object"`인 이유와 주의할 점은 무엇인가요?
9. `NaN`의 자료형은 무엇인가요?
10. 문자열의 특정 인덱스에 문자를 대입해도 원래 문자열이 바뀌지 않는 이유는 무엇인가요?
11. 다음 코드의 출력 결과를 예측하고 이유를 설명해 보세요.
    ```javascript
    let a = 10;
    let b = a;
    b = 20;

    console.log(a, b);
    ```
12. 다음 코드의 출력 결과를 예측하고 이유를 설명해 보세요.
    ```javascript
    const a = { count: 1 };
    const b = a;

    b.count = 2;

    console.log(a.count, b.count);
    ```
13. 다음 코드에서 오류가 발생하는 줄과 이유를 설명해 보세요.
    ```javascript
    const user = { name: "Haewon" };
    user.name = "Kim";
    user = { name: "Lee" };
    ```
14. 원시값 대입과 객체 대입의 핵심 차이를 설명해 보세요.
15. 객체를 두 변수에 대입한 뒤 한쪽의 속성을 변경했을 때 다른 쪽도 바뀌는 이유는 무엇인가요?
16. 다음 비교 결과가 `false`인 이유를 설명해 보세요.
    ```javascript
    const a = { name: "Haewon" };
    const b = { name: "Haewon" };

    console.log(a === b);
    ```
17. 다음 비교 결과가 `true`인 이유를 설명해 보세요.
    ```javascript
    const a = { name: "Haewon" };
    const b = a;

    console.log(a === b);
    ```
18. `===`와 `==`의 차이를 설명하고, 기본적으로 어느 연산자를 사용해야 하는지 말해 보세요.
19. 다음 코드의 결과를 예측해 보세요.
    ```javascript
    console.log(0 == false);
    console.log(0 === false);
    ```
20. 배열이 참조값이라는 것을 보여 주는 짧은 코드를 작성해 보세요.
21. `typeof []`의 결과는 무엇이며, 배열 여부를 확인할 때는 무엇을 사용해야 하나요?
22. 펼침 문법을 사용해 객체를 얕게 복사하는 코드를 작성해 보세요.
23. 다음 코드의 출력 결과를 예측하고 이유를 설명해 보세요.
    ```javascript
    const original = ["a", "b"];
    const copied = [...original];

    copied.push("c");

    console.log(original);
    console.log(copied);
    ```
24. 얕은 복사란 무엇인가요?
25. 다음 코드에서 `original.address.city`가 바뀌는 이유를 설명해 보세요.
    ```javascript
    const original = {
      address: { city: "Busan" }
    };

    const copied = {
      ...original
    };

    copied.address.city = "Seoul";

    console.log(original.address.city);
    ```
26. 다음 코드에서 `user1 === user2`의 결과를 예측해 보세요.
    ```javascript
    const user1 = { name: "A" };
    const user2 = { ...user1 };

    console.log(user1 === user2);
    ```
27. 함수의 인자로 전달한 객체가 함수 밖에서도 바뀌었다면, 어떤 문제를 먼저 의심해야 하나요?
28. 원본 객체를 직접 변경하지 않고 `role` 속성을 추가한 새 객체를 반환하는 함수를 작성해 보세요.
29. 객체 변경 버그를 조사할 때 `console.log()`로 무엇을 확인할 수 있나요?
30. 하나의 객체를 여러 변수가 공유하는 상황이 항상 나쁜가요? 아니라면 어떤 경우에는 의도적으로 사용할 수 있을지 생각해 보세요.
31. `let selectedUser;`와 `let selectedUser = null;`이 각각 표현하는 상태의 차이를 설명해 보세요.
32. 다음 코드의 출력 결과를 예측해 보세요.
    ```javascript
    const x = null;

    console.log(typeof x);
    console.log(x === null);
    ```
33. JavaScript가 동적 타입 언어라는 말은 무엇을 뜻하나요?
34. 다음 코드가 가능한 이유를 설명해 보세요.
    ```javascript
    let value = 10;
    value = "ten";
    ```
35. 참조 공유로 인한 버그를 줄이기 위해 객체·배열을 다룰 때 적용할 수 있는 습관을 세 가지 제안해 보세요.

</details>

---

출처
