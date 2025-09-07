# 아이템 37 공식 명칭에는 상표를 붙이기

구조적 타이핑의 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.

``` tsx
interface Vector2D {
    x: number;
    y: number;
}

function calculateNorm(p: Vector2D) {
    return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x: 3, y: 4}); // 정상 결과는 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D); // 정상! 결과는 동일하게 5
```

calculateNorm 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭을 사용하면 된다. 공식명칭을 사용하는 것은, 타입이 아니라 값의 관점에서 Vector2D라고 말하는 것이다. 공식 명칭 개념을 타입스크립트에서 흉내 내려면 상표를 붙이면 된다.

``` tsx
interface Vector2D {
    _brand: '2d';
    x: number;
    y: number;
}

function vec2D(x: number, y: number): Vector2D {
    return {x, y, _brand: '2d'}
}

function calculateNorm(p: Vector2D) {
    return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4)); // 정상 결과는 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D); // _brand 속성이 형식에 없습니다.
```

상표를 사용해서 calculateNorm 함수가 Vector2D 타입만 받는 것을 보장한다.
그러나 vec3D에 _brand: '2d'라고 추가하는 것 같은 악의적인 사용을 막을 수는 없다.
다만 단순한 실수를 방지하기에는 충분하다.

상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 같은 동일한 효과를 얻을 수 있다.

예를 들어 절대 경로를 사용해 파일 시스템에 접근하는 함수를 가정해보겠다.
런타임에는 절대 경로로 시작하는지 체크하기 쉽지만, 타입 시스템에서는 절대 경로를 파악하기 어렵기 때문에 상표 기법을 사용한다.

``` tsx
type AbsolutePath = string & {_brand: 'abs'};
function listAbsolutePah(path: AbsolutePath) {
    // ... 
}
function isAbsolutePath(path: string): path is AbsolutePath {
    return path.startsWith('/');
}
```

string이면서 _brand 속성을 가지는 객체를 만들 수는 없다. AbsolutePath는 온전히 타입 시스템의 영역이다. 만약 path 값이 절대 경로와 상대 경로 둘다 될 수 있다면 타입을 정제해주는 타입 가드를 사용해 오류를 방지할 수 있다.


``` tsx
function f(path: string) {
    is (isAbsolutePath(path)) {
        listAbsolutePath(path);
    }
    listAbsolutePath(path); // 'string' 형식의 인수는 'AbsolutePath' 형식의 매개변수에 할당될 수 없습니다.
}
```

상표 기법은 타입 시스템 내에서 표현할 수 없는 수많은 속성들을 모델링하는데 사용되기도 한다. 예를 들어, 목록에서 한 요소를 찾기 위해 이진 검색을 하는 경우를 보겠다.


``` tsx
function binarySearch<T>(xs: T[], x: T): boolean {
    let low = 0, high = xs.length - 1;
    while (high >= low) {
        const mid = low + Math.floor((high - low) / 2);
        const v = xs[mid];
        if (v === x) return true;
        [low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
    }
    return false;
}
```

이진 검색은 이미 정렬된 상태를 가정하기 때문에, 목록이 이미 정렬되어 있다면 문제가 없다.
하지만 목록이 정렬되어 있지 않다면 잘못된 결과가 나오는데, 타입스크립트 시스템에서는 목록이 정렬되어 있다는 의도를 표현하기 어렵다. 따라서 다음 예제처럼 상표 기법을 사용해보겠다.


``` tsx
type SortedList<T> = T[] & {_brand: 'sorted'};


function isSorted<T>(xs: T[]): xs is SortedList<T> {
    for (let i = 1; i < xs.length; i++) {
        if (xs[i] < xs[i -  1]) {
            return false
        }
    }
    return true;
}

function binarySearch<T>(xs: SortedList<T>, x: T): boolean { 
    // ...
}
```
binarySearch를 호출하려면 정렬되었다는 상표가 붙은 SortedList 타입의 값을 사용하거나 isSorted를 호출하여 정렬되었음을 증명해야 한다.

number 타입에도 상표를 붙일 수 있다. 예를 들어 단위를 붙여보겠다.

``` tsx
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000);
const oneMin = seconds(60);
```

number 타입에 상표를 붙여도 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하기에는 무리가 있다.
``` tsx
const tenKm = oneKm * 10; // 타입이 number
const v = oneKm / oneMin; // 타입이 number
```

그러나 코드에 여러 단위가 혼합된 많은 수의 숫자가 들어 있는 경우, 숫자의 단위를 문서화하는 괜찮은 방법일 수 있다.

## 요약
- 타입스크립트는 구조적 타이핑을 사용하기 때문에, 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야 한다.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.