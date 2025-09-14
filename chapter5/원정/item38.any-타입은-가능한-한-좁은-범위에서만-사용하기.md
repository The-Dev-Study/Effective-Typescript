# 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

먼저, 함수와 관련된 any의 사용법을 알아보자
``` tsx
function processBar(b: Bar) { }

function f() {
    const x = expressionReturningFoo();
    processBar(x); // Foo 형식의 인수는 Bar 형식의 매개변수에 할당될 수 없습니다.
}
```

문맥상으로 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당 가능하다면, 오류를 제거하는 방법은 두 가지이다.

``` tsx
function f1() {
    const x: any = expressionReturningFoo(); // 이렇게 하지 말자
    processBar(x);
}

function f2() {
    const x = expressionReturningFoo();
    processBar(x as any); // 이게 낫다.
}
```

두 가지 해결책 중 f2의 방법이 더 권장된다. 그 이유는 any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.

비슷한 관점에서, 타입스크립트가 함수의 반환 타입을 추론할 수 있는 경우에도 함수의 반환 타입을 명시하는 것이 좋다. 함수의 반환 타입을 명시하면 any 타입이 함수의 바깥으로 영향을 미치는 것을 방지할 수 있다.

f1과 f2 함수를 다시 한번 살펴보자. f1은 오류를 제거하기 위해 x를 any 타입으로 선언했다. 한편 f2는 오류를 제거하기 위해 x가 사용되는 곳에 as any 단언문을 사용했다. 여기서 @ts-ignore를 사용하면 any를 사용하지 않고 오류를 제거할 수 있다.

``` tsx
function f1() {
    const x = expressionReturningFoo();
    @ts-ignore
    processBar(x);
    return x;
}
```

@ts-ignore를 사용한 다음 줄의 오류가 무시된다. 그러나 근본적인 원인을 해결한 것이 아니기 때문에 다른 곳에서 더 큰 문제가 발생할 수도 있다.
타입 체커가 알려 주는 오류는 문제가 될 가능성이 높은 부분이므로 근본적인 원인을 찾아 적극적으로 대체하는 것이 바람직하다.

이번에는 객체와 관련한 any의 사용법을 살펴보자. 어떤 큰 객체 안의 한 개 속성이 타입 오류를 가지는 상황을 예로 들어보겠다.

``` tsx
const config: Config {
    a: 1,
    b: 2,
    c: {
        key: value
        // foo 속성이 Foo 타입에 필요하지만 Bar 타입에는 없습니다.
    }
}
```

단순히 생각하면 config 객체 전체를 as any로 선언해서 오류를 제거할 수 있지만, 객체 전체를 any로 단언하면 다른 속성들 역시 타입 체크가 되지 않는 부작용이 생긴다. 그러므로 최소한의 범위에만 any를 사용하는 것이 좋다.

``` tsx
const config: Config {
    a: 1,
    b: 2, // 나머지 속성은 여전히 체크된다.
    c: {
        key: value as any
    }
}
```

## 요약
-  의도치 않은 타입 안전성의 손실을 피하기 위해서 any의 사용 범위를 최소한으로 좁혀야 한다. 
- 함수의 반환 타입이 any인 경우 타입 안정성이 나빠진다. 따라서 any 타입을 반환하면 절대 인된다.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore를 사용하는 것이 좋다.