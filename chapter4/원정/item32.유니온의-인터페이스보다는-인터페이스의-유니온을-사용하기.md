# 아이템 32 유니온의 인터페이스 보다는 인터페이스의 유니온을 사용하기

유니온 타입의 속성을 가지는 인터페이스를 작성 중이라면, 혹시 인터페이스의 유니온 타입을 사용하는게 더 알맞지 않을지 검토해 봐야 한다.

백터를 그리는 프로그램을 작성 중이고 특정한 기하학적 타입을 가지는 계층의 인터페이스를 정의한다고 가정해보자

``` tsx
interface Layer {
    layout: FillLayout | LineLayout | PointLayout;
    point: FilPaint | LinePaint | PointPaint;
}
```

layout 속성은 모양이 그려지는 방법과 위치를 제어하는 반면, paint 속성은 스타일을 제어한다.
layout이 LineLayout이면서 paint 속성이 FillPaint 타입인 것은 말이 되지 않는다.

이러한 조합을 허용한다면 라이브러리에서는 오류가 발생하기 십상이고 인터페이스를 다루기도 어려울 것이다.

더 나은 방법으로 모델링하려면 각각 타입의 계층을 분리된 인터페이스로 둬야 한다.

``` tsx
interface FillLayer { 
    layout: FillLayout;
    paint: FillPaint;
}

interface LineLayer {
    layout: LineLayout;
    paint: LinePaint;
}

interface PointLayer {
    //...
}

type Layer = FillLayer | LineLayer | PointLayer;
```

이런 형태로 Layer를 정의하면 layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다.

이러한 패턴의 가장 일반적인 예시는 태그된 유니온(구분된 유니온)이다. Layer의 경우 속성 중의 하나는 문자열 리터럴 타입의 유니온이 된다.

``` tsx 
interface Layer {
    type: 'fill' | 'line' | 'point';
    layout: FillLayout | LineLayout | PointLayout;
    paint: FillPaint | LinePaint | PointPaint;
}
```

type fill과 함께 LineLayout과 PointLayout 타입이 쓰이는 것은 말이 되지 않는다. 이러한 경우를 방지하기 위해 Layer를 인터페이스의 유니온으로 변환해보자.

``` tsx
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
    type: 'paint';
    //...
}

type Layer = FillLayer | LineLayer | PointLayer;
```

type 속성은 태그이며 런타임에 어떤 타입의 Layer가 사용되는지 판단하는 데 쓰인다. 타입스크립트는 태그를 참고하여 Layer의 타입의 범위를 좁힐 수도 있다.

``` tsx
function drawLayer(layer: Layer) {
    if (layer.type === 'fill') {
        const {paint} = layer; // 타입이 FillPaint
        const {layout} = layer; // 타입이 FillLayout
    } else if (layer.type === 'line') {
        const {paint} = layer; // 타입이 linePaint
        const {layout} = layer; // 타입이 lineLayout
    } else {
        const {paint} = layer; // 타입이 PointPaint
        const {layout} = layer; // 타입이 PointLayout
    }
}
```

각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는 데 도움이 된다. 다만 타입 분기 후 Layer가 포함된 동일한 코드가 반복되는 것이 어수선해 보인다.
태그된 유니온은 타입스크립트 타입 체커와 잘 맞기 때문에 타입스크립트 코드 어디에서나 찾을 수 있다.

어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면 보통은 그렇게 하는 것이 좋다. 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 undefined인 경우에도 태그된 유니온 패턴이 잘 맞는다

``` tsx
interface Person {
    name: string;
    // 다음은 둘 다 동시에 있거나 없다.
    placeOfBirth?: string;
    dateOfBirth?: string;
}
```

타입 정보를 담고 있는 주석은 문제가 될 소지가 매우 높다.
placeOfBirth와 dateOfBirth 필드는 실제로 관련되어 있지만 타입 정보에는 어떠한 관계도 표현되지 않았다.
두 개의 속성을 하나의 객체로 모으는 것이 더 나은 설계이다.
이 방법은 null 값을 경계로 두는 방법과 비슷하다.

``` tsx
interface Person {
    name: string;
    birth?: {
        place: string,
        date: Date;
    }
}
```

이제 place만 있고 date가 없는 경우에는 오류가 발생한다.

Person 객체를 매개변수로 받는 함수는 birth 하나만 체크하면 된다.
``` tsx
function eulogize(p: Person) {
    console.log(p.name);
    const {birth} = p;
    if (birth) {
        console.log(birth.date, birth.place);
    }
}
```

타입 구조를 손 댈 수 없는 상황이면 앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링할 수 있다.
``` tsx
interface Name { 
    name: string;
}

interface PersonWithBirth extends Name {
    placeOfBirth: string;
    dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```
이제 중첩된 객체에서도 동일한 효과를 볼 수 있다.

``` tsx
function eulogize(p: Person) {
    if ('placeOfBirth' in p) {
        p // 타입이 PersonWithBirth
        const {dateOfBirth} = p // 타입이 Date
    }
}
```

## 요약
- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋다.
- 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다.