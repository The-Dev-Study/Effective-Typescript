# 아이템 54 객체를 순회하는 노하우

다음 예제는 정상적으로 실행되지만, 편집기에서는 오류가 발생한다. 이유가 뭘까?
``` tsx
const obj = {
    one: 'uno',
    two: 'dos',
    three: 'tres'
}

for (const k in obj) {
    const v = obj[k]
}
```

코드를 수정해 가며 원인을 찾다 보면 obj 객체를 순회하는 루프 내의 상수 k와 관련된 오류라는 것을 알 수 있다.

k의 타입은 string인 반면, obj 객체에는 one, two, three 세 개의 키만 존재한다. k와 obj 객체의 키 타입이 서로 다르게 추론되어 오류가 발생한 것이다.
k의 타입을 더 구체적으로 명시해 주면 오류는 사라진다.

첫 번째 예제의 k 타입이 string으로 추론된 원인은 무엇일까?
이해를 돕기 위해 인터페이스와 함수가 가미된 다른 예제를 보자.

``` tsx
interface ABC {
    a: string;
    b: string;
    c: number;
}

function foo(abc: ABC) { 
    for (const k in abc) {
        const v = abc[k];
                // ~~~ 'ABC' 타입에 인덱스 시그니처가 없기 때문에 엘리먼트는 암시적으로 'any'가 됩니다.
    }
}
```

첫 번째 예제와 동일한 오류이다. 그러므로 (let k: keyof ABC)와 같은 선언으로 오류를 제거할 수 있다. 오류의 내용이 잘못된 것처럼 보이지만, 실제 오류가 맞고 또한 타입스크립트가 정확히 오류를 표시했다. 제대로 된 오류인 이유를 예로 들어 설정해보겠다.

``` tsx
const x = {a: 'a', b: 'b', c: 2, d: new Date()};
foo(x);
```

foo 함수는 a, b, c 속성 외에 d를 가지는 x 객체로 호출이 가능하다. foo 함수는 ABC 타입에 할당 가능한 어떠한 값이든 매개변수로 허용하기 때문이다.

즉, ABC 타입에 할당 가능한 객체에는 a, b, c외에 다른 속성이 존재할 수 있기 때문에, 타입스크립트는 string 타입으로 선택해야 한다.
또한 keyof 키워드를 사용한 방법은 또 다른 문제점을 내포하고 있다.
``` tsx
function foo(abc: ABC) {
    let k: keyof ABC;
    for (k in abc) {
        const v = abc[k];
    }
}
```

k가 'a' | 'b' | 'c' 타입으로 한정되어 문제가 된 것 처럼 v도 string | number 타입으로 한정되어 범위가 너무 좁아 문제가 된다. d: new Date()가 있는 이전 예제처럼, d 속성은 Date 타입뿐만 아니라 어떠한 타입이든 될 수 있기 때문에 v가 string | number 타입으로 추론된 것은 잘못이며 런타임의 동작을 예상하기 어렵다.

골치 아픈 타입 문제 없이, 단지 객체의 키와 값을 순회하고 싶다면 어떻게 해야 할까?
Object.entries를 사용하면 된다.
``` tsx
function foo(abc: ABC) {
    for (const [k, v] of Object.entries(abc) {
        k // string 타입
        v // any 타입
    })
}
```

Object.entries를 사용한 루프가 직관적이지는 않지만, 복잡한 기교 없이 사용할 수 있다.

한편, 객체를 다룰 때는 항상 '프로토타입 오염' 가능성을 염두에 두어야 한다. for-in 구문을 사용하면, 객체의 정의에 없는 속성이 갑자기 등장할 수 있다.

``` tsx
Object.prototype.z = 3; // 제발 이렇게 하지 말자
const obj = {x: 1, y: 2};
for (const k in obj) { console.log(k); }
```

실제 작업에서는 Object.prototype에 순회 가능한 속성을 절대로 추가하면 안된다. for-in 루프에서 k가 string 키를 가지게 된다면 프로토타입 오염의 가능성을 의심해 봐야 한다.

객체를 순회하며 키와 값을 얻으려면, (let k: type keyof T) 같은 keyof 선언이나 Object.entries를 사용하면 된다. keyof 선언은 상수이거나 추가적인 키 없이 정확한 타입을 원하는 경우에 적절하다. Object.entries는 더욱 일반적으로 쓰이지만, 키와 값의 타입을 다루기 까다롭다.

## 요약
- 객체를 순회할 때 어떤 타입인지 정확히 파악하고 있다면 let k: keyof T와 for-in 루프를 사용하자. 함수의 매개변수로 쓰이는 객체에는 추가적인 키가 존재할 수 있다는 점을 명심하자.
- 객체를 순회하며 키와 값을 얻는 가장 일반적인 방법은 Object.entries를 사용하는 것이다.