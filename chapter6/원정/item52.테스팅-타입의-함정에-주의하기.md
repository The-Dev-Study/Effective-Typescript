# 아이템 52 테스팅 타입의 함정에 주의하기

타입 선언을 테스트하기는 매우 어렵다. 그래서 타입 선언에 대한 테스트 코드를 작성할 때 타입스크립트가 제공하는 도구를 사용하여 단언문으로 때우기 십상이지만, 이런 방법에는 몇 가지 문제가 있다. 궁극적으로는 dtslint 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하는 것이 더 안전하고 간단하다.

유틸리티 라이브러리에서 제공하는 map함수의 타입을 선언을 작성한다고 가정해보자.

``` tsx
declare function map<U, V>(array: U[], fun: (u: U) => V): V[];
```

타입 선언이 예상한 타입으로 결과를 내는지 체크할 수 있는 한 가지 방법은 함수를 호출하는 테스트 파일을 작성하는 것이다.

> map(['2017','2018','2019'], v => Number(v));

이 코드는 오류 체크를 수행하지만 허점이 존재한다. 예를 들어 map의 첫 번째 매개변수에 배열이 아닌 단일 값이 있었다면 매개변수의 타입에 대한 오류는 잡을 수 있다. 그러나 반환값에 대한 체크가 누락되어 있기 때문에 완전한 테스트라고 할 수 없다.

앞의 코드와 동일한 스타일로 square라는 함수의 런타임 동작을 테스트한다면 다음과 같은 테스트 코드가 된다.

``` tsx
test('square a number', () => {
    square(1);
    square(2);
})
```

이 테스트 코드는 square 함수의 '실행'에서 오류가 발생하지 않는지만 체크한다. 그러나 반환값에 대해서는 체크하지 않기 때문에, 실제로는 실행의 결과에 대한 테스트는 하지 않은 게 된다.


타입 선언 파일을 테스팅할 때는 이 테스트 코드 처럼 단순히 하수를 실행하는 방식을 일반적으로 적용하게 되는데, 그 이유는 라이브러리 구현체의 기존 테스트 코드를 복사하면 간단히 만들 수 있기 때무이다. 함수를 실행만 하는 테스트 코드가 의미 없는 것은 아니지만, 실제로 반환 타입을 체크하는 것이 훨씬 좋은 테스트 코드이다.

반환값을 특정 타입의 변수에 할당하여 간단히 반환 타입을 체크할 수 있는 방법을 알아보자

``` tsx
const lengths: number[] = map(['john', 'paul'], name => name.length);
```

이 코드는 불필요한 타입 선언에 해당하나 테스트 코드 관점에서는 중요한 역할을 하고 있다. 

그러나 테스팅을 위해 할당을 사용하는 방법에는 두 가지 근본적인 문제가 있다.

첫 번째, 불필요한 변수를 만들어야 한다. 반환값을 할당하는 변수는 샘플 코드처럼 쓰일 수도 있지만, 일부 린팅 규칙(미사용 변수 경고)을 비활성해야 한다.

일반적인 해결책은 변수를 도입하는 대신 헬퍼 함수를 정의하는 것이다.

``` tsx
function assertType<T>(x: T) {}

assertType<number[]>(map['join', 'paul'], name => name.length);
```

이 코드는 불필요한 변수 문제를 해결하지만, 또 다른 문제저미 남아 있다.
두 번째, 두 타입이 동일한지 체크하는 것이 아니라 할당 가능성을 체크하고 있다.

``` tsx
const n = 12;
assertType<number>(n); // 정상
```

n 심벌을 조사해 보면 실제로 숫자 리터럴 타입인 12임을 볼 수 있다. 12는 number의 서브타입이고 할당 가능성 체크를 통과한다.

그러나 객체의 타입을 체크하는 경우를 살펴보면 문제를 발견하게 된다.

``` tsx
const beatles = ['john', 'paul', 'george', 'ringo'];
assertType<{name: string}[]>(
    map(beatles, name => ({
        name,
        inYellowSubmarine: name === 'ringo'
    }))
); // 정상
```

map은 {name: string, inYellowSubmarine: boolean} 객체의 배열을 반환한다. 반환된 배열을 { name: string}[]에 할당 가능하지만, inYellowSubmarine 속성에 대한 부분이 체크되지 않았다. 상황에 따라 타입이 정확한지 체크 할 수도 있고, 할당이 가능한지 체크할 수도 있다.

게다가 assertType에 함수를 넣어 보면, 이상한 결과가 나타난다.
``` tsx
const add = (a: number, n: number) => a + b;
assertType<(a: number, b: number) => number>(add); // 정상

const double = (x: number) => 2 * x;
assertType<(a: number, b: number) => number>(double) // 정상??
```

double 함수의 체크가 성공하는 이유는, 타입스크립트의 함수는 매개변수가 더 적은 함수 타입에 할당 가능하기 때문이다.

또 다른 예제를 들어보자
``` tsx
const g: (x: string) => any = () => 12; // 정상
```

앞의 코드는 선언된 것보다 적은 함수를 할당하는 것이 아무런 문제가 없다는 것을 보여 준다. 타입스크립트에서는 이러한 동작을 모델링하도록 설계되어 있다. 예를 들어, 로대시의 map 함수의 콜백은 세 가지 매개변수를 받는드.

콜백 함수는 세 가지 매개변수 name, index, array 중에서 한 두 개만 보통 사용한다. 매개 변수 세개를 모두 사용하는 경우는 매우 드물다. 만약 매개변수의 개수가 맞지 않는 경우를 타입 체크에서 허용하지 않으면, 매우 많은 곳의 자바스크립트 코드에서 콜백 함수의 타입과 관련된 오류들이 발생하게 될거다.

다시 assertType 문제로 돌아와 보자. 제대로 된 assertType 사용 방법은 무엇일까? 다음 코드처럼 Parameters와 ReturnType 제네릭 타입을 이용해 함수의 매개변수 타입과 반환 타입만 분리하여 테스트할 수 있다.

``` tsx
const double = (x: number) => 2 * x;
let p: Parameters<typeof double> = null!;


assertType<[number, number]>(p);
                        //   ~ '[number]' 형식의 인수는 '[number, number]' 형식에 할당될 수 없다.
let r: ReturnType<typeof double> = null!;
assertType<number>(r); // 정상
```

한편, this가 등장하는 콜백 함수의 경우는 또 다른 문제가 있다. map은 콜백 함수에서 this의 값을 사용할 때가 있으며 타입스크립트는 이러한 동작을 모델링할 수 있으므로 타입 선언에 반영해야 하며 테스트도 해야 한다.

앞서 아이템 52에 등장했던 map에 대한 테스트는 모두 블랙박스 스타일이었다. map의 매개변수로 배열을 넣어 함수를 실행하고 반환 타입을 테스트했지만, 중간 단계의 세부 사항은 테스트하지 않았다. 세부 사항을 테스트하기 위해서 콜백 함수 내부에서 매개변수들의 타입과 this를 직접 체크해 보겠다.

``` tsx
const beatles = ['john', 'paul', 'george', 'ringo'];
assertType<number[]>(map(
    beatles,
    function(name, i, array) {
    // ~~~~~ '(name: any, i: any, array: any) => any' 형식의 인수는 '(u: string) => any' 형식의 매개변수에 할당될 수 없다.
        assertType<string>(name);
        assertType<number>(i);
        assertType<string[]>(array);
        assertType<string[]>(this);
                        //   ~~~~~ 'this'에는 암시적으로 'any' 형식이 포함된다.
        return name.length;
    }
))
```

이 코드는 map의 콜백 함수에서 몇 가지 문제가 발생했다. 한편 이번 예제의 콜백 함수는 화살표 함수가 아니기 때문에 this의 타입을 테스트할 수 있음을 주의하기 바란다.

다음 코드의 선언을 사용하면 타입 체크를 통과한다.
``` tsx
declare function map<U, V>(
    array: U[],
    fn: (this: U[], u: U, i: number, array: U[]) => V
): V[];
```

그러나 여전히 중요한 마지막 문제가 남아 있다. 다음 모듈 선언은 까다로운 테스트를 통과할 수 있는 완전한 타입 선언 파일이지만, 결과적으로 좋지 않은 설계가 된다.

> declare module 'overbar';

이 선언은 전체 모듈에 any 타입을 할당한다. 따라서 테스트는 전부 통과하겠지만, 모든 타입 안전성을 포기하게 된다. 더 나쁜 점은, 해당 모듈에 속하는 모든 함수의 호출하다 암시적으로 any 타입을 반환하기 때문에 타입 안전성을 지속적으로 무너뜨리게 된다는 것이다.


타입 체커와 독립적으로 동작하는 도구를 사용해서 타입 선언을 테스트하는 방법이 권장된다.

DefinitelyTyped의 타입 선언을 위한 도구는 dtslint이다. dtslint는 특별한 형태의 주석을 통해 동작한다. dtslint 사용하면 beatles 관련 예제의 테스트를 다음처럼 작성할 수 있다.

``` tsx
const beatles = ['john', 'paul', 'george', 'ringo'];

map(beatles, function(
    name, // $ExpectedType string
    i,    // $ExpectedType number
    array // $ExpectedType string[]
) {
    this // $ExpectType string[]
    return name.length; 
}); // $ExpectType number[]
```

dtslint는 할당 가능성을 체크하는 대신 각 심벌의 타입을 추출하여 글자 자체가 같은지 비교한다. 

그러나 글자 자체가 같은지 비교하는 방식에는 단점이 있다. number|string과 string|number는 같은 타입이지만 글자 차체로 보면 다르기 때문에 다른 타입으로 인식된다.

타입 선언을 테스트한다는 것은 어렵지만 반드시 해야 하는 작업이다. 앞에서 소개한 일반적인 기법의 문제점을 인식하고, 문제점을 방지하기 위해 dtslint 같은 도구를 사용하도록 하자

## 요약
- 타입을 테스트할 때는 함수 타입의 동일성과 할당 가능성의 차이점을 알고 있어야 하자.
- 콜백이 있는 함수를 테스트할 때, 콜백 매개변수의 추론된 타입을 체크해야 한다. 또한 this가 API의 일부라면 역시 테스트해야 한다.
- 타입 관련 테스트에서 any를 주의해야 한다. 더 엄격한 테스트를 위해 dtslint 같은 도구를 사용하는 것이 좋다.