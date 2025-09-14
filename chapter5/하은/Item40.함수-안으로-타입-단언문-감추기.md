# 아이템 40. 함수 안으로 타입 단언문 감추기

> 함수의 내부에서는 타입 단언을 사용하더라도, 함수 외부로 드러나는 타입 정의는 명확히 명시한다.

## 예제: 캐싱 함수

```ts
// 두 객체의 얕은 동등성 비교 함수
declare function shallowEqual(a: any, b: any): boolean;

// 함수를 받아서 결과를 캐싱하는 고차 함수
function cacheLast<T extends Function>(fn: T): T {
    let lastArgs: any[] | null = null; // 마지막으로 호출했던 인자 배열
    let lastResult: any; // 마지막 호출 반환값

    return function (...args: any[]) {
        // '(...args: any[]) => any' 형식은 'T' 형식에 할당할 수 없습니다.
        if (!lastArgs || !shallowEqual(lastArgs, args)) {
            lastResult = fn(...args);
            lastArgs = args;
        }
        return lastResult;
    } as unknown as T;
}
```

## 예제: 객체의 얕은 동등성 비교 함수

```ts
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== b[k]) {
            return false;
        }
    }
    return Object.keys(a).length === Object.keys(b).length;
}
```

-   `Object.entries(a)`로 객체 a의 모든 키-값 쌍을 순회한다.
-   각 `k`에 대해 `k in b` 이고 `aval == b[k]`인지 확인한다.
-   모든 키를 확인한 후, 두 객체의 키 개수가 같은지 확인한다.

> 구조적으로는 올바른 코드이지만, `b[k]`에서 에러가 발생한다. 이 경우 `any`로 단언할 수 밖에 없다.

```ts
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== (b as any)[k]) {
            return false;
        }
    }
    return Object.keys(a).length === Object.keys(b).length;
}
```

## 요약

-   타입 선언문은 일반적으로 타입 안전성을 헤치지만, 상황에 따라 현실적인 해결책이 될 수 있다. 불가피하게 사용해야 한다면 **정확한 정의를 가지는 함수 안에 숨긴다**.
