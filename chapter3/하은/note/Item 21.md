# 타입 넓히기

> **핵심 개념:** TypeScript는 런타임에 변경될 수 있는 값에 대해 더 일반적인 타입으로 추론한다. 이를 "타입 넓히기"라고 하며, 때로는 예상과 다른 타입 추론 결과를 가져올 수 있다.

## 1. 타입 넓히기란?

### 기본 개념

TypeScript는 초기값을 바탕으로 변수의 타입을 추론할 때, 미래에 할당될 가능성이 있는 값들을 고려하여 더 넓은 타입으로 추론한다.

```ts
let x = 'hello'; // string 타입으로 추론
```

-   ``x의` 타입은 `'hello'`라는 할당된 단일 값을 바탕으로 추론된다.
-   이 때 `string literal` 타입 `'string'`이 아닌 `string`타입으로 추론되게 된다.

**타입 넓히기의 이유**:

-   `let` 변수는 재할당이 가능하므로, 초기값과 다른 string 값들이 할당될 수 있음
-   TypeScript는 이를 고려하여 'hello' 리터럴 타입이 아닌 string 타입으로 추론함

### 타입 넓히기가 발생하는 상황들

```ts
// 1. let 변수 선언
let name = 'Alice'; // string으로 추론
let age = 30; // number로 추론
let isActive = true; // boolean으로 추론

// 2. 객체 프로퍼티
const person = {
    name: 'Alice', // string으로 추론
    age: 30, // number로 추론
};

// 3. 배열 요소
const numbers = [1, 2, 3]; // number[]로 추론
const mixed = [1, 'two']; // (string | number)[]로 추론
```

## 2. 타입 넓히기 제어 방법

### 2.1 const 사용하기

`const`로 선언된 변수는 재할당이 불가능하므로, 더 좁은 타입으로 추론된다.

```ts
const x = 'hello'; // 리터럴 타입 'hello'로 추론
const y = 30; // 리터럴 타입 30으로 추론
const isTrue = true; // 리터럴 타입 true로 추론

// 함수에서 사용할 때의 차이
function greet(name: 'Alice' | 'Bob') {
    console.log(`Hello, ${name}!`);
}

let userName = 'Alice'; // string으로 추론
const constName = 'Alice'; // 'Alice'로 추론

greet(constName); // ✅ 정상 동작
greet(userName); // ❌ Error: string 타입은 'Alice' | 'Bob'에 할당 불
```

#### const의 한계: 객체와 배열

const는 원시값에 대해서만 타입을 좁혀준다. 객체와 배열의 내부는 넓은 타입으로 추론된다.

```ts
// 객체의 경우
const person = {
    name: 'Alice', // string (not 'Alice')
    age: 30, // number (not 30)
};

person.name = 'Bob'; // ✅ 정상 동작 - string이므로

// 배열의 경우
const colors = ['red', 'green', 'blue']; // string[] (not ['red', 'green', 'blue'])
colors.push('yellow'); // ✅ 정상 동작
```

> 이유: 객체와 배열은 참조 타입이므로, const는 참조 자체의 재할당만을 금지하고 내부 프로퍼티의 변경은 허용한다!

### 2.2 명시적 타입 어노테이션

가장 직접적인 방법은 원하는 타입을 명시적으로 선언하는 방법이다.

```ts
const v: { x: 1 | 3 | 5 } = { x: 1 };
v.x = 3; // o
v.x = 4; // x
```

### 2.3 const 단언문 사용하기

값 뒤에 `as const`를 붙이면 타입스크립트는 **최대한 좁은 타입으로 추론**한다.

```ts
const v1 = { x: 1, y: 2 }; // { x: number, y: number }
const v2 = { x: 1 as const, y: 2 }; // { x: 1, y: number }
const v3 = { x: 1, y: 2 } as const; // { readonly x: 1, readonly y: 2 }
```
