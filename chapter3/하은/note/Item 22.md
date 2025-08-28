# 타입 좁히기

> **핵심 개념:** 타입 좁히기는 TypeScript가 넓은 타입에서 더 구체적인 타입으로 범위를 줄여나가는 방법이다. 이를 통해 런타임에서 안전한 연산을 보장하고 더 정확한 타입 추론이 가능하다.

## 1. 타입 좁히기란?

타입 좁히기는 **타입 넓히기의 반대 개념**이다. 유니온 타입이나 넓은 타입에서 조건문이나 타입가트를 사용하면 더 구체적인 타입으로 범위를 좁힐 수 있다.

```ts
// 넓은 타입에서 시작
function processValue(value: string | number | null) {
    // 여기서 value는 string | number | null

    if (value === null) {
        // 이 블록에서 value는 null로 좁혀짐
        console.log('값이 없습니다');
        return;
    }

    if (typeof value === 'string') {
        // 이 블록에서 value는 string으로 좁혀짐
        console.log(value.toUpperCase()); // string 메서드 안전하게 사용 가능
    } else {
        // 이 블록에서 value는 number로 좁혀짐
        console.log(value.toFixed(2)); // number 메서드 안전하게 사용 가능
    }
}
```

## 2. 조건문을 통한 타입 좁히기

### 2.1 null/undefined 체크하기

```ts
function processUser(user: User | undefined) {
    if (!user) {
        console.log('사용자가 없습니다');
        return;
    }
    // 여기서 user는 User로 좁혀짐
    console.log(user.email);
}
```

### 2.2 `typeof` 사용해 원시 타입 체크하기

```ts
function formatInput(input: string | number) {
    if (typeof input === 'string') {
        // 여기서 input은 string
        return input.trim().toLowerCase();
    } else {
        // 여기서 input은 number
        return input.toFixed(2);
    }
}
```

## 3. 분기문에서 예외나 반환을 통한 타입 좁히기

### 3.1 조기 반환

```ts
function processUser(user: User | null) {
    if (user === null) {
        return null; // 조기 반환으로 null 케이스 제거
    }
    // 이후 코드에서 user는 User로 좁혀짐
    const profile = user.profile;
    const preferences = user.preferences;
    // ...
}
```

### 3.2 예외 던지기

```ts
function validateAndProcess(input: string | null | undefined) {
    if (input == null) {
        throw new Error('Input cannot be null or undefined');
    }
    // 이후 코드에서 input은 string으로 좁혀짐
    return input.toUpperCase();
}
s;
```

## 4. instanceof를 사용해 객체 타입 좁히기

```ts
function contains(text: string, search: string | RegExp) {
    if (search instanceof RegExp) {
        search; // Type is RegExp
        return !!search.exec(text);
    }
    search; // Type is string
    return text.includes(search);
}
```

```ts
class Dog {
    bark() {
        console.log('Woof!');
    }
}

class Cat {
    meow() {
        console.log('Meow!');
    }
}

function makeSound(animal: Dog | Cat) {
    if (animal instanceof Dog) {
        // 여기서 animal은 Dog
        animal.bark();
    } else {
        // 여기서 animal은 Cat
        animal.meow();
    }
}

// DOM 요소 타입 좁히기
function handleElement(element: Element) {
    if (element instanceof HTMLInputElement) {
        // 여기서 element는 HTMLInputElement
        console.log(element.value);
    } else if (element instanceof HTMLButtonElement) {
        // 여기서 element는 HTMLButtonElement
        console.log(element.textContent);
    }
}
```

## 5. 내장함수/ 연산자를 사용한 타입 좁히기

### 5.1 Array.isArray

```ts
function processData(data: string | string[]) {
    if (Array.isArray(data)) {
        // 여기서 data는 string[]
        return data.map((item) => item.toUpperCase());
    } else {
        // 여기서 data는 string
        return [data.toUpperCase()];
    }
}
```

### 5.2 `in` 연산자

```ts
interface Bird {
    fly(): void;
    layEggs(): void;
}

interface Fish {
    swim(): void;
    layEggs(): void;
}

function move(animal: Bird | Fish) {
    if ('fly' in animal) {
        // 여기서 animal은 Bird
        animal.fly();
    } else {
        // 여기서 animal은 Fish
        animal.swim();
    }
}
```

## 6. 태그 붙이기

```ts
interface LoadingState {
    kind: 'loading';
}

interface SuccessState {
    kind: 'success';
    data: string;
}

interface ErrorState {
    kind: 'error';
    error: string;
}

type AppState = LoadingState | SuccessState | ErrorState;

// 판별 유니온 사용
function renderState(state: AppState) {
    switch (
        state.kind // 태그를 통한 타입 좁히기
    ) {
        case 'loading':
            // 여기서 state는 LoadingState
            return '로딩 중...';
        case 'success':
            // 여기서 state는 SuccessState
            return `데이터: ${state.data}`;
        case 'error':
            // 여기서 state는 ErrorState
            return `오류: ${state.error}`;
        default:
            // 모든 케이스를 다뤘으므로 여기는 실행되지 않음
            const _exhaustive: never = state;
            return _exhaustive;
    }
}
```

## 7. 사용자 정의 타입 가드

### 7.1 기본 타입 가드

사용자 정의 타입 가드는 `value is Type` 형태의 반환 타입을 갖는 함수이다.

> 런타임 체크와 컴파일 타임 좁히기를 동시 제공한다!

```ts
function isString(value: unknown): value is string {
    return typeof value === 'string';
}
```

### 7.2 객체 타입 가드

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
    return 'value' in el;
}

function getElementContent(el: HTMLElement) {
    if (isInputElement(el)) {
        el; // Type is HTMLInputElement
        return el.value;
    }
    el; // Type is HTMLElement
    return el.textContent;
}
```

```ts
interface User {
    id: number;
    name: string;
    email: string;
}

function isUser(value: unknown): value is User {
    return (
        typeof value === 'object' &&
        value !== null &&
        typeof (value as User).id === 'number' &&
        typeof (value as User).name === 'string' &&
        typeof (value as User).email === 'string'
    );
}
```

> `value is User` -> `true` 면 타입 체커에게 매개변수 `value`의 타입을 좁힐 수 있다고 알려준다!

### 책 예제 중 좋았던 예제

```ts
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map((who) => jackson5.find((n) => n === who)); // Type is (string | undefined)[]
```

`find()`의 반환타입은 다음과 같다.

-   조건을 만족하는 값을 찾으면 해당 요소 반환
-   없으면 `undefined` 반환
    따라서 반환 타입은 `T | undefined`이다!

```ts
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map((who) => jackson5.find((n) => n === who)).filter((who) => who !== undefined); // Type is (string | undefined)[]
```

그런데 필터만 사용해서 타입을 좁힐 수 없다!!

왜냐면...

-   filter()에 논리식을 넣으면 실행 시점에는 `undefined`가 제거되지만, 타입스크립트는 그 사실을 확신할 수 없다!
-   따라서 여전히 타입이 `(T | undefined)[]` 로 남음

이는 타입 가드를 통해 해결할 수 있다.

```ts
function isDefined<T>(x: T | undefined): x is T {
    return x !== undefined;
}

const members = ['Janet', 'Michael'].map((who) => jackson5.find((n) => n === who)).filter(isDefined); // 타입이 string[]
```

-   `filter()`에 타입 가드 함수를 전달하면, TypeScript가 정확히 `T[]`임을 추론할 수 있다!
