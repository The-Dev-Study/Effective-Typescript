# 아이템 41 any의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다. 그 후에 정제될 수 있지만, 새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입과 관련해서 예외인 경우가 존재한다.

자바스크립트에서 일정 범위의 숫자들을 생성하는 함수를 예로 들어보자

``` js
function range(start, limit) {
    const out = [];
    for (let i = start; i < limit; i++) {
        out.push(i);
    }
    return out;
}
```
이 코드를 타입스크립트로 변환하면 예상한 대로 정확히 동작한다.
``` tsx
function range(start: number, limit: number) {
    const out = [];
    for (let i = start; i < limit; i++) {
        out.push(i);
    }
    return out; // 반환 타입이 number[]로 추론됨.
}
```

그러나 자세히 살펴보면 한 가지 이상한 점을 발견할 수 있다. out의 타입이 처음에는 any 타입 배열인 []로 초기화되었는데, 마지막에는 number[]로 추론되고 있다.

코드에 out이 등장하는 세 가지 위치를 조사해 보면 이유를 알 수 있다.
``` tsx
function range(start: number, limit: number) {
    const out = []; // 타입이 any[]
    for (let i = start; i < limit; i++) {
        out.push(i); // out 의 타입이 any[]
    }
    return out; // 타입이 number[]
}
```

out의 타입은 any[]로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 number[]로 진화한다.

타입의 진화는 타입 좁히기와 다르다. 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화한다.

``` tsx
const result = [] // 타입이 any[]
result.push('a');
result // 타입이 string[]
result.push(1);
result // 타입이 (string | number)[]
```

또한 조건문에서는 분기에 따라 타입이 변할 수도 있다. 다음 코드에서는 배열이 아닌 단순 값으로 예를 들어보자
```tsx
let val; // 타입이 any
if (Math.random() < 0.5) {
    val = /hello/;
    val // 타입이 RegExp
} else {
    val = 12
    val // 타입이 number
}
val // 타입이 number | RegExp
```

변수의 초기값이 null인 경우도 any의 진화가 일어난다. 보통은 try/catch 블록 안에서 변수를 할당하는 경우에 나타난다.

any 타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 암시적 any인 경우에만 일어난다. 그러나 다음처럼 명시적으로 any를 선언하면 타입이 그대로 유지된다.

```tsx
let val: any; // 타입이 any
if (Math.random() < 0.5) {
    val = /hello/;
    val // 타입이 any
} else {
    val = 12
    val // 타입이 any
}
val // 타입이 any
```

다음 코드처럼, 암시적 any 상태인 변수에 어떠한 할당도 하지 않고 사용하려고 하면 암시적 any 오류가 발생하게 된다.

``` tsx
function range(start: number, limit: number) {
    const out = [];
    //    ~~~ out 변수는 형식을 확인할 수 없는 경우 일부 위치에서 암시적으로 any[] 형식입니다.
    if (start === limit) {
        return out;
        //      ~~~ out 변수에는 암시적으로 any[] 형식이 포함됩니다.
    }
    for (let i = start; i < limit; i++) {
        out.push(i);
    }
    return out;
 }
```

any 타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생한다.
그리고 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생한다.

암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다. 다음 코드에서 forEach 안의 화살표 함수는 추론에 영향을 미치지 않는다.

```tsx
function makeSquares(start: number, limit: number) {
    const out = [];
    //    ~~~ out 변수는 일부 위치에서 암시적으로 any[] 형식입니다.
    range(start, limit).forEach(i => {
        out.push(i * i);
    });
    return out;
    //      ~~~ out 변수에는 암시적으로 any[]  형식이 포함됩니다.
}
```

앞의 코드와 같은 경우라면 루프로 순회하는 대신, 배열의 map과 filter 메서드를 통해 단일 구문으로 배열을 생성하여 any 전체를 진화시키는 방법을 생각해 볼 수 있다.

any가 진화하는 방식은 일반적인 변수가 추론되는 원리와 동일하다.
예를 들어 진화한 배열의 타입이 (string|number)[]라면 원래 number[] 타입이어야 하지만 실수로 string이 섞여서 잘못 진화한 것일 수 있다. 타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계이다.

## 요약
- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[] 타입은 진화할 수 있다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.