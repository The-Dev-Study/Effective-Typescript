# Summary 5장 요약과 느낀점

## 5장에 들어가며
타입스크립트의 타입은 선택적이고 점진적이기 때문에 정적이면서도 동적인 특성을 동시에 가진다.
따라서 타입스크립트는 프로그램의 일부분에만 타입 시스템을 적용할 수 있다.

프로그램의 일부분에만 타입 시스템을 적용할 수 있다는 특성 덕분에 점진적인 마이그레이션이 가능하다. 마이그레이션을 할 때 코드의 일부분에 타입 체크를 비활성화시켜주는 any가 중요한 역할을 한다.

any를 현명하게 사용하는 방법을 익혀야만 효과적인 타입스크립트 코드를 작성할 수 있다.

## any 타입은 가능한 한 좁은 범위에서만 사용하기
먼저, 함수와 관련된 any의 사용법을 알아보자
``` tsx
function processBar(b: Bar) { }

function f() {
    const x = expressionReturningFoo();
    processBar(x); // Foo 형식의 인수는 Bar 형식의 매개변수에 할당될 수 없습니다.
}
```
문맥상으로 x라는 변수가 동시에 Foo 타입과 Bar 타입에 할당 가능하다면, 오류를 제거하는 방법은 두 가지이다.

두 가지 해결책 중 f2의 방법이 더 권장된다. 그 이유는 any 타입이 processBar 함수의 매개변수에서만 사용된 표현식이므로 다른 코드에는 영향을 미치지 않기 때문이다.

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

### 요약
-  의도치 않은 타입 안전성의 손실을 피하기 위해서 any의 사용 범위를 최소한으로 좁혀야 한다. 
- 함수의 반환 타입이 any인 경우 타입 안정성이 나빠진다. 따라서 any 타입을 반환하면 절대 인된다.
- 강제로 타입 오류를 제거하려면 any 대신 @ts-ignore를 사용하는 것이 좋다.

## any를 구체적으로 변형해서 사용하기
any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입이다. 반대로 말하면, 일반적인 상황에서는 any보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 더 구체적인 타입을 찾아 타입 안정성을 높이도록 해야 한다.

예를 들어, any 타입의 값을 그대로 정규식이나 함수에 넣는 것은 권장되지 않는다.

``` tsx
function getLengthBad(array: any) { // 이렇게 하지 말자.
    return array.length;
}

function getLength(array: any[]) {
    return array.length;
}
```

앞의 예제에서 any를 사용하는 getLengthBad보다 any[]를 사용하는 getLength가 더 좋은 함수이다. 이유는 세가지다.
- 함수 내의 array.length 타입이 체크된다.
- 함수의 반환 타입이 any 대신 number로 추론된다.
- 함수 호출될 때 매개변수가 배열인지 체크된다.

함수의 매개변수를 구체화할 때, 배열의 배열 형태라면 any[][]처럼 선언하면 된다. 그리고 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 {[key: string]: any}처럼 선언하면 된다.

앞의 예제처럼 함수의 매개변수가 객체지만 값을 알 수 없다면 {[key: string]: any} 대신 모든 비기본형 타입을 포함하는 object 타입을 사용할 수도 있다. object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 {[key: string]: any}와 약간 다르다.

### 요약
- any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야 한다.
- any보다 더 정확하게 모델링할 수 있도록 any[] 또는 {[id: string]: any} 또는 () => any처럼 구체적인 형태를 사용해야 한다.

## 함수 안으로 타입 단언문 감추기
함수의 모든 부분을 안전한 타입으로 구현하는 것이 이상적이지만, 불필요한 예외 상황까지 고려해 가며 타입 정보를 힘들게 구성할 필요는 없다.

함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 명확히 명시하는 정도로 끝내는 게 낫다. 프로젝트 전반에 위험한 타입 단언문이 드러나 있는 것보다, 제대로 타입이 정의된 함수 안으로 타입 단언문을 감추는 것이 더 좋은 설계이다.

### 요약
- 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 한다.

## any의 진화를 이해하기
타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다. 그 후에 정제될 수 있지만, 새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입과 관련해서 예외인 경우가 존재한다.

``` tsx
function range(start: number, limit: number) {
    const out = [];
    for (let i = start; i < limit; i++) {
        out.push(i);
    }
    return out; // 반환 타입이 number[]로 추론됨.
}
```

위 코드를 자세히 살펴보면 한 가지 이상한 점을 발견할 수 있다. out의 타입이 처음에는 any 타입 배열인 []로 초기화되었는데, 마지막에는 number[]로 추론되고 있다.

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

``` tsx
const result = [] // 타입이 any[]
result.push('a');
result // 타입이 string[]
result.push(1);
result // 타입이 (string | number)[]
```

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

타입을 안전하게 지키기 위해서는 암시적 any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계이다.

### 요약
- 일반적인 타입들은 정제되기만 하는 반면, 암시적 any와 any[] 타입은 진화할 수 있다.
- any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

## 모르는 타입의 값에는 any 대신 unknown을 사용하기

#### 함수의 반환값과 관련된 unknown
YAML 파서인 parseYAML 함수를 작성한다고 가정해보자 JSON.parse의 반환 타입과 동일하게 parseYAML 메서드의 반환 타입을 any로 만들어보자

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

unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다.

unknown 타입인 채로 값을 사용하면 오류가 발생한다. unknown인 값에 함수 호출을 하거나 연산을 하려고 해도 마찬가지다. unknown 상태로 사용하려고 하면 오류가 발생하기 때문에, 적절한 타입으로 변환하도록 강제할 수 있다.

``` tsx
const book = safeParseYAML(`
    name: Book Name
    author: Charlotte Bronte
`) as Book;
alert(book.title); // book 형식에 title 속성이 없습니다.
book('read'); // 이 식은 호출할 수 없습니다.
```
함수의 반환 타입은 unknown 그대로 값을 사용할 수 없기 때문에 Book으로 타입 단언을 해야한다. 애초에 반환값이 Book이라고 기대하며 함수를 호출하기 때문에 단언문은 문제가 되지 않는다. 그리고 Book 타입 기준으로 타입 체크가 되기 때문에 unknown 타입 기준으로 오류를 표시했던 예제보다 오류의 정보가 더 정확하다.

#### 변수 선언과 관련된 unknown
어떠한 값이 있지만 그 타입을 모르는 경우에 unknown을 사용한다. 
예를 들어, GeoJSON 사양에서 Feature의 properties 속성은 JSON 직렬화가 가능한 모든 것을 담는 잡동사니 주머니 같은 존재이다. 그래서 타입을 예상할 수 없기 때문에 unknown을 사용한다.

``` tsx
interface Feature {
    id?: string | number;
    geometry: Geometry;
    properties: unknown;
}
```
타입 단언문이 unknown에서 원하는 타입으로 변환하는 유일한 방법은 아니다. instannceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다. 또한 사용자 정의 타입 가드도 unknown에서 원하는 타입으로 변환할 수 있다.

가끔 unknown 대신 제네릭 매개변수가 사용되는 경우도 있다. 제네릭을 사용하기 위해 다음 코드처럼 safeParseYAML 함수를 선언할 수 있다.

``` tsx
function safeParseYAML<T>(yaml: string): T {
    return parseYAML(yaml);
}
```
그러나 앞의 코드는 일반적으로 타입스크립트에서 좋지 않은 스타일이다.
제네릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.

#### 단언문과 관련된 unknown
다음으로 단언문과 관련된 unknown을 알아보자. 이중 단언문에서 any 대신 unknown을 사용할 수도 있다.

``` tsx
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

barAny와 barUnk는 기능적으로 동일하지만, 나중에 두 개의 단언문을 분리하는 리팩터링을 한다면 unknown 형태가 더 안전하다. any의 경우는 분리되는 순간 그 영향력이 전염병처럼 퍼지게 된다. 그러나 unknown의 경우는 분리되는 즉시 오류를 발생하게 하므로 더 안전하다.

#### unknown과 유사하지만 조금 다른 타입들
object 또는 {}를 사용하는 방법 역시 unknown만큼 범위가 넓은 타입이지만 unknown보다는 범위가 약간 좁다.
- {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 모든 비기본형 타입으로 이루어진다. 여기에는 true또는 12또는 "foo"가 포함되지 않지만 객체와 배열은 포함된다.

unknown 타입이 도입되기 전에는 {}가 더 일반적으로 사용되었지만 최근에는 {}를 사용하는 경우가 꽤 드물다. 정말로 null과 undefined가 불가능하다고 판단되는 경우에만 {}을 사용하면 된다.

### 요약
- unknown은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 unknown을 사용하면 된다.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.
- {}, object, unknown의 차이점을 이해해야 한다.

## 몽키 패치 보다는 안전한 타입을 사용하기
자바스크립트의 가장 유명한 특징 중 하나는 객체와 클래스에 임의의 속성을 추가할 수 있을 만큼 유연하다는 것이다. 

사실 객체에 임의의 속성을 추가하는 것은 일반적으로 좋은 설계가 아니다.
예를 들어 window 또는 DOM 노드에 데이터를 추가한다고 가정해보자
그러면 그 데이터는 기본적으로 전역 변수가 된다. 전역 변수를 사용하면 은연중에 프로그램 내에서 서로 멀리 떨어진 부분들 간에 의존성을 만들게 된다.

그러면 함수를 호출할 때 마다 부작용을 고려해야만 한다.
타입스크립트까지 더하면 또 다른 문제가 발생한다. 타입 체커는 Document와 HTMLElement의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.

이 오류를 해결하는 가장 간단한 방법은 any 단언문을 사용하는 것이다.
``` tsx
(document as any).monkey = 'Tamarin'; // 정상
```

타입 체커는 통과하지만 단점이 있다. any를 사용함으로써 타입 안전성을 상실하고 언어 서비스를 사용할 수 없게 된다는 것이다.

최선의 해결책은 document 또는 DOM으로부터 데이터를 분리하는 것이다.
분리할 수 없는 경우 두가지 차선책이 존재한다.

첫번째 interface 특수 기능 중 하나인 보강을 사용하는 것
``` tsx
interface Document {
    monkey: string
}

document.monkey = 'Tamarin'; // 정상
```

보강을 사용한 방법이 any보다 나은 점은 다음과 같다.

- 타입이 더 안전하다. 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
- 속성에 주석을 붙일 수 있다.
- 속성에 자동완성을 사용할 수 있다.
- 몽치 패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.

그리고 모듈의 관점에서 제대로 동작하게 하려면 global 선언을 추가해야 한다.
``` tsx
export {};
declare global {
    interface Document {
        monkey: string;
    }
}
document.monkey = 'Tamarin';
```

보강을 사용할 때 주의해야 할 점은 모듈 영역과 관련이 있다. 보강은 전역적으로 적용되기 때문에, 코드이 다른 부분이나 라이브러리로부터 분리할 수 없다. 그리고 애플리케이션이 실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없다. 특히 웹 페이지 내의 HTML 엘리먼트를 조작할 때, 어떤 엘리먼트는 속성이 있고 어떤 엘리먼트는 속성이 없는 경우 문제가 된다.

두 번째, 더 구체적인 타입 단언문을 사용하는 것이다.
``` tsx
interface MonkeyDocument extends Document {
    monkey: string;
}

(document as MonkeyDocument).monkey = 'Macaque';
```
MonkeyDocument는 Document를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전하다. 또한 Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있다. 따라서 몽키 패치된 속성을 참조하는 경우에만 단언문을 사용하거나 새로운 변수를 도입하면 된다.

### 요약
- 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리해 사용해야 한다.
- 내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나를 사용해야 한다.
- 보강 모듈 영역 문제를 이해해야 한다.

## 타입 커버리지를 추적하여 타입 안정성 유지하기

noImplicitAny를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 any 타입과 관련된 문제들로부터 안전하다고 할 수 없다. any 타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우가 있다.

- 명시적 any 타입
아이템 38과 아이템 39의 내용에 따라 any 타입의 범위를 좁히고 구체적으로 만들어도 여전히 any 타입이다. 특히 any[]와 {[key: string]: any} 같은 타입은 인덱스를 생성하면 단순 any가 되고 코드 전반에 영향을 미친다.

- 서드파티 타입 선언
이 경우는 @types 선언 파일로부터 any 타입이 전파되기 때문에 특별히 조심해야 한다. noImplicitAny를 설정하고 절대 any를 사용하지 않았다 하더라도 여전히 any 타입은 코드 전반에 영향을 미친다.

any 타입은 타입 안전성과 생산성에 부정적 영향을 미칠 수 있으므로 프로젝트에서 any의 개수를 추적하는 것이 좋다. npm의 type-cover-age 패키지를 활용하여 any를 추적할 수 있는 몇 가지 방법이 있다.

any가 등장하는 몇 가지 문제와 그 해결책을 살펴보겠다.
표 형태의 데이터에서 어떤 종류의 열 정보를 만들어 내는 함수를 만든다고 가정해보자.

``` tsx
function getColumnInfo(name: string): any {
    return utils.buildColumInfo(appState.dataSchema, name); // any를 반환합니다.
}
```
utils.buildColumInfo 호출은 any를 반환한다. 그래서 geColumnInfo 함수의 반환에는 주석과 함께 명시적으로 : any 구문을 추가했다.

이후에 타입 정보를 추가하기 위해 ColumnInfo를 반환하도록 개선해도 getColumnInfo 함수의 반환문에 있는 any까지 제거해야 문제가 해결된다.


서드파티 라이브러리로부터 비롯되는 any 타입은 몇 가지 형태로 등장할 수 있지만 가장 극단적인 예는 전체 모듈에 any 타입을 부여하는 것이다.

``` tsx
declare module 'my-module';
```

앞의 선언으로 인해 my-module에서 어떤 것이든 오류 없이 임포트할 수 있다. 임포트한 모든 심벌은 any 타입이고, 임포트한 값이 사용되는 곳마다 any 타입을 양산하게 된다.

일반적인 모듈의 사용법과 동일하기 때문에, 타입 정보가 모두 제거됐다는 것을 간과할 수 있다. 또는 동료가 모든 타입 정보를 날려 버렸지만, 알아채지 못하는 경우일 수도 있다. 그렇기 때문에 가끔 해당 모듈을 점검해야 한다.

선언된 타입과 실제 반환된 타입이 맞지 않는다면 어쩔 수 없이 any 단언문을 사용해야 한다.

### 요약
- noImplicitAny가 설정되어 있어도, 명시적 any 또는 서드파티 타입 선언을 통해 any 타입은 코드 내에 여전히 존재할 수 있다는 점을 주의해야 한다.
- 작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다.