# 아이템 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

### 함수의 반환값과 관련된 unknown
먼저 함수의 반환값과 관련된 unknown을 알아보자. YAML 파서인 parseYAML 함수를 작성한다고 가정해보자 JSON.parse의 반환 타입과 동일하게 parseYAML 메서드의 반환 타입을 any로 만들어보자

``` tsx
function parseYAML(yaml: string): any {
    // ...
}
```

아이템 38에서 설명했듯이 (any 타입은 가능한 한 좁은 범위에서만 사용하기) 함수의 반환 타입으로 any를 사용하는 것은 좋지 않은 설계이다.
대신 parseYAML을 호출한 곳에서 반환값을 원하는 타입으로 할당하는 것이 이상적이다.

``` tsx
interface Book {
    name: string;
    author: string;
}

const book: Book = parseYAML(`
    name: book name,
    author: kim wonjeong
`);

alert(book.title); // 런타임에 undefined 경고
book('read'); // 런타임에 TypeError 예외 발생
```

대신 parseYAML이 unknown 타입을 반환하게 만드는 것이 더 안전하다.

``` tsx
function safeParseYAML(yaml: string): unknown {
    return parseYAML(yaml);
}

const book = safeParseYAML(`
    name: The Tenant of Wildfell Hall
    author: Anne Brontë
`);
alert(book.title);
//    ~~~~ 개체가 unknown 형식입니다.
book('read');
//~~~~~~~~~~ 개체가 unknown 형식입니다.
```

unknown 타입을 이해하기 위해서는 할당 가능성의 관점에서 any를 생각해 볼 필요가 있다. any가 강력하면서도 위험한 이유는 다음 두 가지 특징으로 부터 비롯된다.
- 어떠한 타입이든 any 타입에 할당 가능하다.
- any 타입은 어떠한 타입으로도 할당 가능하다.

(타입을 값의 집합으로 생각하기) 아이템7의 관점에서, 한 집합은 다른 모든 집합의 부분 집합이면서 동시에 상위 집합이 될 수 없기 때문에, 분명히 any는 타입 시스템과 상충되는 면을 가지고 있다. 이런 점이 any의 강력함의 원천이면서 동시에 문제를 일으키는 원인이 된다. 타입 체커는 집합 기반이기 때문에 any를 사용하면 타입 체커가 무용지물이 된다는 것을 주의해야 한다.

unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다.
unknown 타입은 앞에서 언급한 any의 첫 번째 속성(어떠한 타입이든 unknown에 할당 가능)을 만족하지만, 두 번째 속성(unknown은 오직 unknown과 any에만 할당 가능)은 만족하지 않는다. 반면 never 타입은 unknown과 정반대이다. 첫 번째 속성(어떤 타입도 never에 할당할 수 없음)은 만족하지 않지만, 두 번째 속성(어떠한 타입으로도 할당 가능)은 만족한다.

한편 unknown 타입인 채로 값을 사용하면 오류가 발생한다. unknown인 값에 함수 호출을 하거나 연산을 하려고 해도 마찬가지다. unknown 상태로 사용하려고 하면 오류가 발생하기 때문에, 적절한 타입으로 변환하도록 강제할 수 있다.

``` tsx
const book = safeParseYAML(`
    name: Book Name
    author: Charlotte Bronte
`) as Book;
alert(book.title); // book 형식에 title 속성이 없습니다.
book('read'); // 이 식은 호출할 수 없습니다.
```

함수의 반환 타입은 unknown 그대로 값을 사용할 수 없기 때문에 Book으로 타입 단언을 해야한다. 애초에 반환값이 Book이라고 기대하며 함수를 호출하기 때문에 단언문은 문제가 되지 않는다. 그리고 Book 타입 기준으로 타입 체크가 되기 때문에 unknown 타입 기준으로 오류를 표시했던 예제보다 오류의 정보가 더 정확하다.

### 변수 선언과 관련된 unknown
다음으로 변수 선언관 관련된 unknown을 알아보자. 어떠한 값이 있지만 그 타입을 모르는 경우에 unknown을 사용한다. 앞서 예로 든 parseYAML 외에 다른 예시들도 있다. 예를 들어, GeoJSON 사양에서 Feature의 properties 속성은 JSON 직렬화가 가능한 모든 것을 담는 잡동사니 주머니 같은 존재이다. 그래서 타입을 예상할 수 없기 때문에 unknown을 사용한다.

``` tsx
interface Feature {
    id?: string | number;
    geometry: Geometry;
    properties: unknown;
}
```
타입 단언문이 unknown에서 원하는 타입으로 변환하는 유일한 방법은 아니다. instannceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다.

``` tsx
function processValue(val: unknown) {
    if (val instanceof Date) {
        val // 타입이 Date
    }
}
```
또한 사용자 정의 타입 가드도 unknown에서 원하는 타입으로 변환할 수 있다.

``` tsx
function isBook(val: unknown): val is Book {
    return (
        typeof(val) === 'object' && val !== null && 'name' in val && 'author' in val
    );
}

function processValue(val: unknown) {
    if (isBook(val)) {
        val // 타입이 Book
    }
}
```

unknown 타입의 범위를 좁히기 위해서는 상당히 많은 노력이 필요하다. in 연산자에서 오류를 피하기 위해 먼저 val이 객체임을 확인해야 하고, typeof null === 'object'이므로 별도로 val이 null이 아님을 확인해야 한다.

가끔 unknown 대신 제네릭 매개변수가 사용되는 경우도 있다. 제네릭을 사용하기 위해 다음 코드처럼 safeParseYAML 함수를 선언할 수 있다.

``` tsx
function safeParseYAML<T>(yaml: string): T {
    return parseYAML(yaml);
}
```

그러나 앞의 코드는 일반적으로 타입스크립트에서 좋지 않은 스타일이다.

제네릭을 사용한 스타일은 타입 단언문과 달라 보이지만 기능적으로는 동일하다. 제네릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.

### 단언문과 관련된 unknown
다음으로 단언문과 관련된 unknown을 알아보자. 이중 단언문에서 any 대신 unknown을 사용할 수도 있다.

``` tsx
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

barAny와 barUnk는 기능적으로 동일하지만, 나중에 두 개의 단언문을 분리하는 리팩터링을 한다면 unknown 형태가 더 안전하다. any의 경우는 분리되는 순간 그 영향력이 전염병처럼 퍼지게 된다. 그러나 unknown의 경우는 분리되는 즉시 오류를 발생하게 하므로 더 안전하다.

### unknown과 유사하지만 조금 다른 타입들
마지막으로 unknown과 유사하지만 조금 다른 타입들도 알아보자.
이번 아이템에서 unknown에 대해서 설명한 것과 비슷한 방식으로 object 또는 {}를 사용하는 코드들이 존재한다. object 또는 {}를 사용하는 방법 역시 unknown만큼 범위가 넓은 타입이지만 unknown보다는 범위가 약간 좁다.
- {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 모든 비기본형 타입으로 이루어진다. 여기에는 true또는 12또는 "foo"가 포함되지 않지만 객체와 배열은 포함된다.

unknown 타입이 도입되기 전에는 {}가 더 일반적으로 사용되었지만 최근에는 {}를 사용하는 경우가 꽤 드물다. 정말로 null과 undefined가 불가능하다고 판단되는 경우에만 {}을 사용하면 된다.

## 요약
- unknown은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 unknown을 사용하면 된다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.
- {}, object, unknown의 차이점을 이해해야 한다.