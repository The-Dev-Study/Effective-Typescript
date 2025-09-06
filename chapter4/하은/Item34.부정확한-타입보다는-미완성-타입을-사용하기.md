# 아이템 34. 부정확한 타입보다는 미완성 타입을 사용하기

> 실수가 발생하기 쉬운 잘못된 타입은 타입이 없는 것보다 못하다.

타입이 구체적일 수록 버그를 많이 잡고 도구를 사용할 수 있지만, 타입의 정밀도를 높이는 과정에서 실수가 발생하기 쉽다.

## 예제

### 1단계 - 모두 허용

모든 것을 허용하는 타입인 `any`나 숫자, 문자열, 배열만 허용

```ts
type Expression1 = any;

const tests: Expression1[] = [
    10,
    'red',
    true, // ❗
    ['+', 10, 5],
    ['case', ['>', 20, 10], 'red', 'blue', 'green'], // ❗
    ['**', 2, 31], // ❗
    ['rgb', 255, 128, 64],
    ['rgb', 255, 0, 127, 0], // ❗
```

-   ❗ 표시가 된 부분은 전부 에러가 나야 하는 부분인데, 실제로 에러가 안남
-   모든 값이 들어갈 수 있어서, 타입이 엄격하지 않아 오류를 잘 못 잡는 것!

### 2단계 - 문자열, 숫자, 배열 허용

```ts
type Expression2 = number | string | any[];

const tests: Expression2[] = [
    10,
    'red',
    true, // ✔️문법적 에러
    ['+', 10, 5],
    ['case', ['>', 20, 10], 'red', 'blue', 'green'], // ❗타입 에러는 안나는데, 말이 안됨 string은 두 개만 가능함
    ['**', 2, 31], // ❗'**'는 함수가 아니기 때문에 오류가 발생해야 하지만 안남
    ['rgb', 255, 128, 64],
    ['rgb', 255, 0, 127, 0], ❗// 값이 너무 많이 있음 3개까지 들어갈 수 있는데 4개 들어가 있음
];
```

-   `boolean` 형태는 걸러냈지만, 아직도 해결못한 에러들이 존재함

### 3단계 - 문자열, 숫자, 정의된 함수 이름으로 시작하는 배열 허용

`'case'`, `'rgb'`들은 함수를 의미하므로 이를 묶어서 하나의 `FnName` 타입으로 만들고, `callExpression`이라는 추가적인 타입 생성함

```ts
type FnName = '+' | '-' |'*'| '/' | '>' | '<' | 'case' | 'rgb';
type callExpression = [FnName, ...any[]];
type Expression3 = number | string | callExpression;

const tests: Expression3[] = [
    10,
    'red',
    true, // ✔️
    ['+', 10, 5],
    ['case', ['>', 20, 10], 'red', 'blue', 'green'], // ❗타입 에러는 안나는데, 말이 안됨 string은 두 개만 가능함
    ['**', 2, 31], // ✔️
    ['rgb', 255, 128, 64],
    ['rgb', 255, 0, 127, 0], ❗// 값이 너무 많이 있음 3개까지 들어갈 수 있는데 4개 들어가 있음
];
```

-   `**`와 같이 함수 이름이 아닌 값이 들어오면 에러를 발생시킴

### 4단계 - 각 함수가 받는 매개변수의 개수 확인하기

`'case'`, `'rgb'`들은 함수를 의미하고, 각 함수별로 받아야하는 매개변수 개수가 다르기 때문에 이를 타입에 반영한다.

```ts
type Expression4 = number | string | callExpression;

type callExpression = MathCall | CaseCall | RGBCall;

interface MathCall {
    0: '+' | '-' | '/' | '*' | '>' | '<';
    1: Expression4;
    2: Expression4;
}
interface CaseCall {
    0: 'case';
    1: Expression4;
    2: Expression4;
    3: Expression4;
    length: 4 | 6 | 8 | 10 | 12 | 14 | 16;
}

interface RGBCall {
    0: 'rgb';
    1: Expression4;
    2: Expression4;
    3: Expression4;
    length: 4;
}

const tests: Expression4[] = [
    10,
    'red',
    true, // ✔️
    ['+', 10, 5],
    ['case', ['>', 20, 10], 'red', 'blue', 'green'], // ✔️ 말이 안됨 string은 두 개만 가능함
    ['**', 2, 31], // ✔️
    ['rgb', 255, 128, 64],
    ['rgb', 255, 0, 127, 0], // ✔️ length 가 4를 넘어서 에러
    ['-', 12], // 에러가 나면 안되지만 에러가 발생함
    ['+', 1, 2, 3], // 에러가 나면 안되지만 에러가 발생함
];
```

-   에러를 다 잡기는 했지만, 타입 선언의 복잡성으로 인해 버그 발생확률도 같이 높아짐

## 요약

-   어설프게 완벽을 추구하다 보면 오히려 역효과가 발생할 수 있다.
-   정확하게 모델링할 수 없는 경우, 부정확한 모델링을 하지 말아야 한다.
-   `any`와 `unknown`을 구별해서 사용해야 한다.
-   타입 정보를 구체적으로 만들 수록 오류 메세지와 자동 완성 기능에 주의를 기울여야 한다.
