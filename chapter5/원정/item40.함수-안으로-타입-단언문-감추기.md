# 아이템 40 함수 안으로 타입 단언문 감추기

함수를 작성하다 보면, 외부로 드러나 타입 정의는 간단하지만 내부 로직이 복잡해서 안전한 타입으로 구현하기 어려운 경우가 많다. 함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없다.

함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 명확히 명시하는 정도로 끝내는 게 낫다. 프로젝트 전반에 위험한 타입 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계이다.

예를 들어, 어떤 함수가 자신의 마지막 호출을 캐시하도록 만든다고 가정해보자.
함수 캐싱은 리액트 같은 프레임워크에서 실행 시간이 오래 걸리는 함수 호출을 개선하는 일반적인 기법이다. 어떤 함수든 캐싱할 수 있도록 래퍼 함수 cacheLast를 만들어보겠다.

``` tsx
declare function cacheLast<T extends Function>(fn: T): T;

declare function shallowEqual(a: any, b: any): boolean;

function cacheLast<T extends Function>(fn: T): T {
    let lastArgs: any[] | null = null;
    let lastResult: any;

    return function(...args: any[]) {
        // ~~~~~~~~~~~~~~~~~~~~~~~~~~
        // (...args: any[]) => any 형식은 T 형식에 할당할 수 없다.
        if (!lastArgs || !shallowEqual(lastArgs, args)) {
            lastResult = fn(...args);
            lastArgs = args;
        }
        return lastResult;
    }
}
```

타입스크립트는 반환문에 있는 함수와 원본 함수 T 타입이 어떤 관련이 있는지 알지 못하기 때문에 오류가 발생했다. 그러나 결과적으로 원본 함수 T 타입과 동일한 매개변수로 호출되고 반환값 역시 예상한 결과가 되기 때문에, 타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지는 않는다.

``` tsx
function cacheLast<T extends Function>(fn: T): T {
    let lastArgs: any[] | null = null;
    let lastResult: any;

    return function(...args: any[]) {
        if (!lastArgs || !shallowEqual(lastArgs, args)) {
            lastResult = fn(...args);
            lastArgs = args;
        }
        return lastResult;
    } as unknown as T;
}
```

실제로 함수를 실행해 보면 잘 동작한다. 함수 내부에는 any가 꽤 많이 보이지만 타입 정의에는 any가 없기 때문에, cacheLast를 호출하는 쪽에서는 any가 사용됐는지 알지 못한다.

한편 앞 예제에 나온 shallowEqual은 두 개의 배열을 매개변수로 받아서 비교하는 함수이며 타입 정의와 구현이 간단하다. 그러나 객체를 매개변수로 하는 shallowObjectEqual은 타입 정의는 간단하지만 구현이 조금 복잡하다.

``` tsx
declare function shallowObjectEqual<T extends object>(a: T, b: T): boolean;
```
객체 매개변수 a와 b가 동일한 키를 가진다는 보장이 없기 때문에 구현할 때는 주의해야 한다.

``` tsx
declare function shallowEqual(a: any, b: any): boolean;

function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== b[k]) {
                            //    ~~~~  '{}' 형식에 인덱스 시그니처가 없으므로 요소에 암시적으로 any 형식이 있습니다.
            return false;
        }
    }
    return Object.keys(a).length === Object.keys(b).length;
}
```

if 구분의 k in b 체크로 b 객체에 k 속성이 있다는 것을 확인했지만 b[k]부분에서 오류가 발생하는 것이 이상하다. 어쨌든 실제 오류가 아니라는 것을 알고 있기 때문에 any로 단언하는 수 밖에 없다.

``` tsx
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
    for (const [k, aVal] of Object.entries(a)) {
        if (!(k in b) || aVal !== (b as any)[k]) {
            return false;
        }
    }
    return Object.keys(a).length === Object.keys(b).length;
}
```
b as any 타입 단언문은 안전하며 (k in b 체크를 했으므로) 결국 정확한 타입으로 정의되고 제대로 구현된 함수가 된다. 객체가 같은지 체크하기 위해 객체 순회와 단언문이 코드에 직접 들어가는 것보다, 앞의 코드처럼 별도의 함수로 분리해 내는 것이 훨씬 좋은 설계이다.

## 요약
- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 한다.