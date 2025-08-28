# 이펙티브 타입스크립트 아이템 25: 타입 추론과 명시적 타입 선언

> **핵심 개념:** 타입스크립트의 강력한 타입 추론 기능을 활용하며, 필요한 순간에만 명시적 타입 선언을 사용하여 코드의 안전성과 가독성을 확보하는 방법

## 1. 타입스크립트의 타입 추론 원리

타입스크립트는 코드의 문맥을 분석하여 변수, 함수 반환값, 표현식의 타입등을 자동으로 추론한다.

```ts
let message = 'hello'; // string으로 추론
const msg = 'message'; // string literal로 추론

function add(a: number, b: number) {
    return a + b; // number로 추론
}
```

## 2. 명시적 타입 선언이 불필요한 경우

> 🤔타입스크립트는 타입을 위한 언어니까, 변수를 선언할 때 무조건 타입을 명시해야 하지 않나?

타입스크립트를 사용할 때 타입 구문이 불필요한 경우가 많다. 정확한 타입 추론이 가능하다면, 오히려 일일이 명시적 타입 구문을 작성하는게 **비생산적**일 수 있다.

### 2.1 지역 변수와 구조분해 할당

```ts
interface Product {
    id: number;
    name: string;
    price: number;
}

function logProduct(product: Product) {
    // 타입 명시 불필요 - product 타입으로부터 추론 가능
    const { id, name, price } = product;

    // 지역 변수도 초기값으로부터 타입 추론
    const discountedPrice = price * 0.9; // number로 추론

    console.log(id, name, discountedPrice);
}
```

-   `id`, `name`, `price`의 타입은 `product`의 타입을 바탕으로 추론할 수 있다. (구조적 타이핑)

### 2.2 기본값이 있는 매개변수

```ts
// base 매개변수는 기본값 10으로부터 number 타입 추론
function parseNumber(str: string, base = 10) {
    return parseInt(str, base);
}

// 객체 기본값도 마찬가지
function createUser(name: string, options = { isAdmin: false, theme: 'light' }) {
    // options의 타입이 자동으로 추론됨
    return { name, ...options };
}
```

-   기본값이 있는 매개변수의 경우 기본값을 바탕으로 타입 정보를 얻을 수 있다.

## 3. 명시적 타입 선언이 필요한 경우

### 3.1 기본값이 없는 매개변수

```ts
function logProduct(product) {
    const { id, name, price } = product;
}
```

-   함수의 매개변수에 기본값이 없는 경우, 함수의 외부로부터 오는 데이터의 타입을 알 수 없다.
-   매개변수 product의 타입은 추론 정보가 부족하기 때문에 정확히 타입을 지정해줘야 한다.

### 3.2 객체 리터럴의 타입 명시

```ts
interface Product {
    id: number;
    name: string;
    price: number;
}

// 타입을 명시하면 잉여 속성 체크를 활성화할 수 있다.
const elmo: Product = {
    name: '곰돌이',
    id: 1,
    price: 10000,
};

// 타입을 명시하지 않은 경우
const elmo2 = {
    name: '곰돌이',
    id: 1,
    price: 10000,
    category: 'toy', // 오류 없음, 하지만 나중에 Product 타입으로 사용할 때 문제 발생 가능
};
```

정의에 타입을 명시함으로써 다음과 같은 이점을 챙길 수 있다.

-   **잉여 속성 체크:**: 의도치 않은 속성을 추가하는 걸 방지할 수 있다.
-   **할당 시점에서 오류 발견:** 변수 사용 시점이 아닌 실제 실수가 발생한 지점에서 즉시 오류를 표시할 수 있다.
-   **타입 안전성 향상:** 객체 구조의 정확성을 보장할 수 있다.

### 3.3 함수의 반환타입

함수의 반환 타입을 명시하면 함수를 호출하는 곳이 아닌 함수 반환값 자체에 오류를 표시할 수 있다.(실제 오류 발생 지점!) 또한 반환 타입을 명시함으로써 미리 틀을 정해두고 개발을 진행할 수 있다는 장점도 있다.

```ts
function fetchUserData(id: string): Promise<User> {
    return fetch(`/api/users/${id}`)
        .then((response) => response.json())
        .then((data) => {
            // 만약 여기서 잘못된 형태의 객체를 반환한다면
            // 함수 내부에서 오류를 발견할 수 있음
            return data as User;
        });
}

// 복잡한 비즈니스 로직
function calculateDiscount(user: User, product: Product): DiscountInfo {
    // 복잡한 계산 로직...
    // 반환 타입을 명시함으로써 함수의 의도를 명확히 하고
    // return 문에서 타입 오류를 조기에 발견할 수 있다.
    return {
        amount: discountAmount,
        percentage: discountPercentage,
        // validUntil: new Date() // 이 속성을 빼먹었다면 여기서 오류 발생
    };
}
```

-   **오류 위치의 명확화:** 함수를 사용하는 곳이 아닌, 함수 내부의 return 문에서 오류 발생
-   **함수 설계의 명확성:** 구현 전에 함수의 인터페이스를 설계하는 TDD와 유사한 접근
-   **명명된 타입 사용:** 복잡한 반환 타입을 의미 있는 이름으로 표현 가능
-   **API 일관성:** 함수의 계약(contract)을 명확히 정의
