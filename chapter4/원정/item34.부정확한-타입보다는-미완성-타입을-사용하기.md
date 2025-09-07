# 아이템 34 부정확한 타입보다는 미완성 타입 사용하기
타입 선언을 작성하다 보면 코드의 동작을 더 구체적으로 또는 덜 구체적으로 모델링하는 상황을 맞닥뜨리게 된다. 일반적으로 타입이 구체적일수록 버그를 더 많이 잡고 타입스크립트가 제공하는 도구를 활용할 수 있게 된다.

그러나 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야 한다.
아이템 31에서 보았던 GeoJSON 형식의 타입 선언을 작성한다고 가정해 보겠다. GeoJSON 정보는 각각 다른 형태의 좌표 배열을 가지는 몇 가지 타입 중 하나가 될 수 있다.

``` tsx
interface Point {
    type: 'Point';
    coordinates: number[];
}

interface LineString {
    type: 'LineString';
    coordinates: number[][];
}

interface Polygon {
    type: 'Polygon';
    coordinates: number[][][];
}

type Geometry = Point | LineString | Polygon // 다른 것들도 추가될 수 있다.
```

큰 문제는 없지만 좌표에 쓰이는 number[]가 약간 추상적이다. 여기서 number[]는 경도와 위도를 나타내므로 튜플 타입으로 선언하는 게 낫다.

``` tsx 
type GeoPosition = [number, number];
interface Point {
    type: 'Point';
    coordinates: GeoPosition;
}
```

타입을 구체적으로 개선했기 때문에 더 나은 코드가 된 것 같지만, 그렇지 않다.

코드에는 위도와 경도만을 무시했지만, GeoJSON의 위치 정보에는 세 번째 요소인 고도가 있을 수 있고 또 다른 정보가 있을 수 있다.

현재의 타입 선언을 그대로 사용하려면 사용자들은 타입 단언문을 도입하거나 as any를 추가해서 타입 체커를 완전히 무시해야 한다.

## 요약
- 어설프게 타입을 추가하지 말아야 합니다.
- 정확하게 타입을 모델링할 수 없다면, 부정확하게 모델링하지 말아야 합니다. 또한 any와 unknown을 구별해서 사용해야 합니다.
- 타입 정보를 구체적으로 만들수록 오류 메시지와 자동 완성 기능에 주의를 기울여야 한다.