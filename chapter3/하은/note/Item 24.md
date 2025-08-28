# 일관성 있는 별칭 사용하기

> **핵심 원칙:** 별칭(alias)은 타입 좁히기를 방해할 수 있으므로, 일관성 있게 사용해야 한다. 같은 값에 대해 다른 이름을 혼용하지 말고, 구조 분해 할당을 적극 활용하자.

## 1. 별칭의 개념과 문제점

### 1.1 별칭이란?

별칭은 기존 값이나 객체에 새로운 이름을 부여하는 것이다.

```ts
const borough = {
    name: 'Brooklyn',
    location: [40.688, -73.979],
};

const loc = borough.location; // loc은 borough.location의 별칭
```

-   `borough.location`에 `loc`이라는 별칭(alias)를 붙이는 코드

### 1.2 별칭의 기본 동작

별칭을 통해 값을 변경하면 원본에 반영된다.

```ts
// 별칭을 통한 값 변경이 원본에 반영!
loc[0] = 40.7; // borough.location[0]도 40.7로 변경됨
console.log(borough.location[0]); // 40.7
```

-   별칭 값을 변경하면 원래 속성값에서도 변경된다.

### 1.3 별칭으로 인한 제어 흐름 분석의 어려움

별칭을 남발하면 제어 흐름 추적 및 분석이 어려워진다.

```ts
interface Polygon {
    exterior: Coordinate[];
    holes: Coordinate[][];
    bbox?: BoundingBox;
}

interface BoundingBox {
    x: [number, number];
    y: [number, number];
}

const polygon: Polygon = getPolygon();

// ❌ 문제가 있는 코드: 별칭과 원본을 혼용
const box = polygon.bbox; // 별칭 생성

if (polygon.bbox) {
    // polygon.bbox는 BoundingBox로 타입 좁히기됨
    console.log(polygon.bbox.x[0]); // ✅ 정상 동작

    // 하지만 box는 여전히 BoundingBox | undefined
    console.log(box.x[0]); // ❌ Error: box는 undefined일 수 있음
}
```

> 타입스크립트의 타입 좁히기는 **특정 변수**에 대해서만 적용된다. 따라서 `polygon.bbox`가 `BoundingBox`로 좁혀졌다고 해서, 같은 값을 가리키는 `box`까지 자동으로 좁혀지지는 않는다!

## 2. 일관성 있게 별칭 사용하기

### 2.1 별칭을 타입 좁히기에 활용하기

타입 좁히기에 별칭인 `box`를 쓰면 문제를 해결할 수 있다.

```ts
const polygon: Polygon = getPolygon();

const box = polygon.bbox;

if (box) {
    // box를 직접 체크 → BoundingBox로 좁혀짐
    console.log(box.x[0]);
    console.log(box.y[1]);
}
```

-   근데 `box`와 `bbox`는 동일한 값이지만, 이름이 다름. 이것도 맞춰주는게 좋긴 함!

### 2.2 원본과 별칭을 혼용하지 않기

일관되게 별칭만 사용하든지 원본만 사용하든지 하자.

```ts
const polygon: Polygon = getPolygon();
const box = polygon.bbox;

if (box) {
    if (isPointInBox(point, box)) {
        expandBox(box, 10);
        drawBox(box);
    }

    // ❌ 원본과 별칭을 혼용 → 혼란을 야기함
    // if (isPointInBox(point, polygon.bbox)) {
    //     expandBox(box, 10);  // box와 polygon.bbox를 섞어 사용
    // }
}
```

## 3. 구조 분해 할당 활용

### 3.1 구조 분해를 통한 일관된 별칭

구조 분해 할당을 사용하면 원본과 같은 이름을 유지하면서 별칭의 장점을 얻을 수 있다.

```ts
const { bbox } = polygon;

if (bbox) {
    // bbox는 BoundingBox로 타입 좁히기됨
    const { x, y } = bbox; // 중첩 구조 분해

    if (point.x < x[0] || point.x > x[1] || point.y < y[0] || point.y > y[1]) {
        return false;
    }
}
```

## 4. 별칭 사용 시 주의사항

### 4.1 함수 호출과 타입 좁히기

```ts
interface ApiResponse {
    data?: UserData;
    error?: string;
}

function processResponse(response: ApiResponse) {
    const { data } = response;

    if (data) {
        // data는 UserData로 타입 좁히기됨
        console.log(data.name);

        // ⚠️: 함수 호출 후 타입 좁히기가 무효화될 수 있음
        someAsyncFunction(); // 이 함수가 response를 수정할 수 있음

        // 여기서 data는 여전히 UserData로 인식되지만,
        // 실제로는 undefined가 될 수 있는 위험이 있음
        console.log(data.email); // 런타임 오류 가능성
    }
}
```

### 4.2 지역 변수의 재할당

```ts
function calculateDistance(polygon: Polygon) {
    let { bbox } = polygon;

    if (bbox) {
        // bbox는 BoundingBox로 좁혀짐
        console.log(bbox.x[0]);

        // ⚠️ 지역 변수 재할당
        if (someCondition) {
            bbox = getDefaultBoundingBox(); // 다른 객체로 재할당
        }

        // 이제 bbox는 원본과 다른 객체를 가리킬 수 있음
        console.log(bbox.y[0]);
    }
}
```

-   지역변수에 객체 구조분해 할당 이후에 필드값을 수정하면 다른 주소를 가리킬 수도 있다.
