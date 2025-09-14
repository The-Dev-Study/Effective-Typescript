# 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

## 함수에서 `any` 사용하기

함수에 전달하는 인자의 타입이 함수의 매개변수 타입과 일치하지 않는 경우, 이를 두 가지 방법으로 해결할 수 있다.

### (1) `x : any` 사용하기

```ts
function f1() {
    const x: any = expressionReturningFoo();
    processBar(x);
}
```

-   권장되지 않는 방법이다.
-   `x`의 타입이 `expressionReturningFoo` 호출 이후에도 `any`이다.
-   `return`문에 `any`타입 값이 포함되어 있으면 해당 함수를 호출하는 모든 코드에 영향을 주게 된다.
-

### (2) `func(x as any)`

```ts
function f2() {
    const x = expressionReturningFoo();
    processBar(x as any);
}
```

-   더 권장되는 방법이다.
-   `any`타입이 `processBar`함수의 매개변수에서만 사용되고 이후의 코드에는 영향을 미치지 않는다.
-   `any`타입이 함수 바깥으로 영향을 미치지 않는다.

### `@ts-ignore`를 사용해 오류 무시하기

`@ts-ignore`를 사용하면 오류를 무시할 수 있다. 그러나 근본적인 원인을 해결한 것은 아니다!

```ts
function f1() {
    const x = expressionReturningFoo();
    // @ts-ignore
    processBar(x);
}
```

## 객체 안에서 `any` 사용하기

어떤 큰 객체 안에서 한 개의 속성이 타입 오류를 가지는 상황을 생각해보자.

```ts
const config: Config = {
    a: 1,
    b: 2,
    c: {
        key: value,
        // ~~~ 'foo' 속성이 'foo' 타입에 필요하지만 'Bar'타입에는 없습니다.
    },
};
```

`any`타입을 써서 해결하는 방법에는 두 가지가 있다.

### (1) `{} as any` 사용하기

```ts
const config: Config = {
    a: 1,
    b: 2,
    c: {
        key: value,
    },
} as any;
```

-   **객체 전체**를 `any`로 단언하는 방법이다.
-   모든 속성의 타입체크가 되지 않는다는 문제점이 있다.

### (2) `{ ..., key: value as any}` 사용하기

```ts
const config: Config = {
    a: 1, //  타입 체크 됨
    b: 2,
    c: {
        key: value as any,
    },
};
```

-   최소한의 범위에 `any`를 사용하는 방법이다.
-   `any`를 적용한 속성을 제외하고는 타입 체크가 된다.

## 요약

-   의도치 않은 타입 안전성 손실을 피하기 위해 `any`를 최대한 좁은 범위에 적용하기
-   함수의 반환타입으로 `any`쓰는 거 지양하기
-   강제로 타입 오류를 제거할 때에는 `@ts-ignore` 사용하기
