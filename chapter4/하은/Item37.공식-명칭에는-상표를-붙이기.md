# 아이템 37. 공식 명칭에는 상표 붙이기

`Vector2D`와 `Vector3D`가 있다고 할 때, `Vector2D`만을 파라미터로 받을 수 있는 함수를 만들어야 한다면 `_brand` 속성을 추가한다.

> 이를 **브랜드 타입**이라고 한다.

```ts
interface Vector2D {
    _brand: '2d';
    x: number;
    y: number;
}

function vec2D(x: number, y: number): Vector2D {
    return { x, y, _brand: '2d' };
}

function calculateNorm(p: Vector2D) {
    return Math.sqrt(p.x * p.x + p.y * p.y); // 기존과 동일
}

calculateNorm(vec2D(3, 4)); // 정상, 5를 반환합니다.
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D);
// ~~~~~ '_brand' 속성이 ... 형식에 없습니다.
```

-   `_brand`값이 '2d'가 아니면 `vec2D`ㄱ

## 브랜드 타입 사용해서 단위 시스템 표현하기

```ts
type Meters = number & { _brand: 'meters' };
type Seconds = number & { _brand: 'seconds' };

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); // 타입이 Meters
const oneMin = seconds(60); // 타입이 Seconds
```
