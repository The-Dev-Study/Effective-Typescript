# 아이템 32. 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

-   유니온 타입의 속성보다는 인터페이스의 유니온 타입이 적절하지 않은지 고려해보기
-   타입 좁히기 시에 같이 좁혀져야 하는 속성들이 있다면 이걸 모아서 하나의 타입으로 감싸고 쉽게 좁힐 수 있도록 하는게 좋은 것 같음

```ts
interface Layer {
    layout: FillLayout | LineLayout | PointLayout;
    paint: FillPaint | LinePaint | PointPaint;
}
```

```ts
interface FillLayer {
    type: 'fill';
    layout: FillLayout;
    paint: FillPaint;
}

interface LineLayer {
    type: 'line';
    layout: LineLayout;
    paint: LinePaint;
}

interface PointLayer {
    type: 'point';
    layout: PointLayout;
    paint: PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;
```

-   각 속성을 따로 따로 좁혀줄 필요가 없다.
-   따라서 속성이 잘못된 조합으로 섞이는 버그를 사전에 방지할 수 있다.

```ts
interface Person {
    name: string;
    placeOfBirth?: string;
    dateOfBirth?: Date;
}
```

-   이것도 `placeOfBirth`와 `dateOfBirth`는 둘 다 있거나 둘다 없거나 인데 코드만 봐서는 그 관계가 드러나지 않는다.

```ts
interface Person {
    name: string;
    birth?: {
        place: string;
        date: Date;
    };
}
```

-   이렇게 하면 둘 다 있거나 둘 다 없어야 함이 명확하다.
-   `birth` 속성 하나만 체크해주면 된다.
