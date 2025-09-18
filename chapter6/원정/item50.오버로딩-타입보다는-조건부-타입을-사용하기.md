# 아이템 50 오버로딩 타입보다는 조건부 타입을 사용하기


다음 예제의 double 함수에 타입 정보를 추가해보자
``` js
function double(x) {
    return x + x;
}
```

double 함수에는 string 또는 number 타입의 매개변수가 들어올 수 있다. 그러므로 유니온 타입을 추가했다.

``` tsx
function double(x: number | string): number | string;
function double(x: any) { return x + x; }
```

선언이 틀린건 아니지만, 모호한 부분이 있다.

``` tsx
const num = double(12); // string | number
const str = double('x'); // string | number
```

double에 number 타입을 매개변수로 넣으면 number 타입을 반환한다. 그리고 string 타입을 매개변수로 넣으면 string 타입을 반환한다.
그러나 선언문에는 number 타입을 매개변수로 넣고 string 타입을 반환하는 경우도 포함되어 있다.

제네릭을 사용하면 이러한 동작을 모델링할 수 있다.

``` tsx
function double<T extends number | string>(x: T): T;
function double(x: any) { return x + x; }

const number double(12); // 타입이 12
const str = double('x'); // 타입이 'x'
```

타입을 구체적으로 만들어 보려는 시도는 좋았지만 너무 과했다. 이제는 타입이 너무 과하게 구체적이다. 

또 다른 방법은 여러 타입 선언으로 분리하는 것이다. 타입스크립트에서 함수의 구현체는 하나지만, 타입 선언은 몇 개든지 만들 수 있다. 이를 활용하여 double의 타입을 개선할 수 있다.

``` tsx
function double(x: number): number;
function double(x: string): string;
function double(x: any) { return x + x; }

const num = double(12); // 타입이 number
const str = double('x'); // 타입이 string
```

함수 타입이 조금 명확해졌지만 여전히 버그는 남아 있다. string이나 number 타입의 값으로는 잘 동작하지만 유니온 타입 관련해서 문제가 발생한다.

``` tsx
function f(x: number | string) {
    return double(x);
    //            ~ 'string | number' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없다.
}
```

이 예제에서 double 함수의 호출은 정상적이며 string|number 타입이 반환되기를 기대한다. 한편 타입스크립트는 오버로딩 타입 중에서 일치하는 타입을 찾을 때까지 순차적으로 검색한다. 그래서 오버로딩 타입의 마지막 선언까지 검색했을 때, string|number 타입은 string에 할당할 수 없기 때문에 오류가 발생한다.

세 번째 오버로딩 타입을 추가하여 문제를 해결할 수도 있지만, 가장 좋은 해결책은 조건부 타입을 사용하는 것이다.
조건부 타입은 타입 공간의 if 구문과 같다.

``` tsx
function double<T extends number | string>(x: T): T extends string ? string : number;
function double(x: any) { return x + x; }
```
이 코드는 제네릭을 사용했던 예제와 유사하지만 반환 타입이 더 정교하다.
조건부 타입은 자바스크립트의 삼항 연산자처럼 사용하면 된다.

- T가 string의 부분 집합이면, 반환 타입이 string이다.
- 그 외의 경우에는 반환 타입이 number다

조건부 타입이라면 앞선 모든 예제가 동작한다.

유니온에 조건부 타입을 적용하면, 조건부 타입의 유니온으로 분리되기 때문에 number|string의 경우에도 동작한다. 

오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 더 정확해진다.

## 요약
- 오버로딩 타입보다 조건부 타입을 사용하는 것이 좋다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원할 수 있다.