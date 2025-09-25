# 아이템 54 객체를 순회하는 노하우

객체를 순회할 때 실행은 되지만, 편집기 오류가 발생하는 예제가 있음

> 객체를 순회하는 키와 관련된 오류들

```ts
const obj: {
    one: string;
    two: string;
    three: string;
};

for (const k in obj) {
    const v = obj[k];
    // const k: string
}
```

-   `k`의 타입은 `string` 이고, 이는 `obj`의 키 타입의 범위인 (`one | two | three`)보다 넓음

## `keyof` 사용하기

```ts
const obj =  {
    one: string;
    two: string;
    three: string;
}

let k: keyof typeof obj;

for (k in obj) { // k: 'one' | 'two' |'three'
    const v = obj[k];
}
```

## `Object.entries`

함수 내부에서 파라미터로 들어온 객체를 순회할 때에는 `keyof`를 사용해도 문제가 발생할 수 있음.

```ts
type ABC = { a: string; b: string; c: number };

function foo(abc: ABC) {
    let k: keyof ABC;
    for (const k in abc) {
        // k : 'a', 'b', 'c'
        v; // string | number
    }
}

foo({ a: 'a', b: 'b', c: 2, d: new Date() });
```

-   `ABC`타입에 할당 가능한 어떤 값이든 매개변수로 허용하기 때문에, `key`와 `value` 모두 범위가 너무 좁음

이런 타입 문제는 `Object.entries`를 사용해 순회하면 해결할 수 있음

```ts
function foo(abc: ABC) {
    for (const [k, v] of Object.entries(abc)) {
        k; // string 타입
        v; // any 타입
    }
}
```
