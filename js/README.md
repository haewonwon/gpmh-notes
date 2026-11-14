# js

JavaScript 언어 본체를 정리합니다.
[Full Stack 커리큘럼](https://app.notion.com/p/3dc178c4afac800b891ed30b433ee075)의 Track A · 1–3주 JavaScript 항목을 기준으로 합니다.

## 범위

- 변수와 자료형, 함수, 배열과 객체, 구조 분해, 컬렉션, 모듈
- 실행 컨텍스트, 호출 스택, 스코프, 호이스팅, 클로저, this, 프로토타입, 클래스, 복사와 불변성
- 동기·비동기, 이벤트 루프, 큐, 콜백, Promise, async/await, 비동기 조합, 오류 처리

## 범위 밖

- TypeScript 타입 시스템 → [`../ts/`](../ts/)
- React/Next.js 등 프레임워크 → [`../frontend/`](../frontend/)
- Node.js·서버·API 구현 → [`../backend/`](../backend/)

## 노트

### JavaScript Basics

- [변수와 자료형 · Primitive / Reference](variables-and-data-types.md)
- [함수 · Function Declaration / Expression / Arrow Function](functions.md)
- [Array / Object](array-object.md)
- [Destructuring · Spread / Rest](destructuring-spread-rest.md)
- [Map / Set](map-set.md)
- [Module · ESM / CommonJS 개념](modules-esm-commonjs.md)

### JavaScript Core

- [Execution Context](execution-context.md)
- [Call Stack](call-stack.md)
- [Scope · Lexical Scope · Scope Chain](scope-lexical-scope-scope-chain.md)
- [Hoisting · TDZ](hoisting-tdz.md)
- [Closure](closure.md)
- [this · call / apply / bind](this-call-apply-bind.md)
- [Prototype · Prototype Chain](prototype-chain.md)
- [Class](class.md)
- [Shallow Copy / Deep Copy · Immutability](shallow-deep-copy-immutability.md)

### Async JavaScript

- [Synchronous / Asynchronous](synchronous-asynchronous.md)
- [Event Loop](event-loop.md)
- [Task Queue / Microtask Queue](task-microtask-queue.md)
- [Callback](callback.md)
- [Promise](promise.md)
- [async / await](async-await.md)
- [Promise.all / allSettled / race](promise-combinators.md)
- [Error Handling](error-handling.md)
