# [이펙티브 타입스크립트] 4장 요약본

## 28. 유효한 상태만 표현하는 타입을 지향하기

> 타입 설계 시 각 **상태를 명확하게 분리**해야 한다.

### 코드

```ts
interface State {
    pageText: string;
    isLoading: boolean;
    error?: string;
}
```

-   단순히 `isLoading: boolean`과 `error?: string`같은 조합은 **무효한 상태**가 발생할 수 있어 위험하다.

```ts
interface RequestPending {
    state: 'pending';
}

interface RequestError {
    state: 'error';
    error: string;
}

interface RequestSuccess {
    state: 'ok';
    pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;
```

-   네트워크 요청 상태에 따라 분리된 타입들(`pending`, `error`, `ok`)을 정의하고 이 타입들의 유니언 타입으로 `RequestState`를 정의한다.

## 29. 사용할 때는 너그럽게, 생성할 때는 엄격하게

> -   매개변수는 선택적 속성 / 유니온 타입을 잘 사용해서 함수 사용성을 높이는게 좋다.
> -   그러나 반환 타입의 범위는 작게 하는게 좋다.

-   매개변수의 타입 범위가 넓으면 사용성이 높아진다.
-   반환 타입의 범위가 넓으면 사용성이 낮아진다.
-   따라서 사용하기 편하려면 반환타입이 엄격해야 함

## 30. 문서에 타입 정보를 쓰지 않기

> 주석에 타입 정보를 반복해서 적지 말자!

-   타입스크립트에서 타입 선언은 코드상에서의 명확한 타입 정보를 제공하기 때문에, 주석에 타입 정보를 적는 것은 불필요하고 유지보수 시 혼란이 발생할 수 있다.
    -   타입이 바뀌면 주석도 수정해야 한다;🤔
    -   타입 시스템이 이미 주석 역할을 충분히 해주고 있다.
-   변수명에 타입 정보를 포함하지 않는게 좋다.
    -   `ageNum` → `age`
    -   단, 단위가 있는 숫자의 경우 단위가 변수명에 포함되는게 명확하다. (`durationMs`, `temperatureC`)

## 31. 타입 주변에 null값 배치하기

>

### 31. 타입 주변에 null값 배치하기

-   여러 관련 값을 반환할 경우, 각각 따로 반환하거나 개별 체크하지 말고 **하나의 튜플이나 객체로 묶어 반환하자.**
-   값이 없음을 명확히 표현해야 할 때는 **명시적 `null` 타입을 사용해 초기값을 설정하는 것이 좋다.**
    -   예: `let result: [number, number] | null = null;`
-   `number`타입 변수에는 `0`이 포함될 수 있어, 조건문에서 단순 `falsy` 체크를 하면 의도치 않은 동작이 발생할 수 있다.
-   속성 타입에 `UserInfo | null`, `Post[] | null` 같이 null을 혼용하면, 가능한 조합이 많아지고 null 체크 로직이 많아져 복잡도가 증가한다.
-   이를 방지하기 위해 필요한 데이터가 모두 준비된 후, non-null 타입으로 객체를 생성하는 방식을 사용하는게 좋다.

### 느낀 점

`0`이 저장될 수 있는 `Number`변수의 경우 `if`문을 쓸 때 특히 조심해야겠다는 생각을 했다. `&&`연산자를 사용할 때에도 마찬가지로 `{0 && <div>hello</div>}` 이 경우 0이 렌더링 되게 되는데, 0은 아무것도 없는 상태가 아니라는 걸 항상 기억해야겠다!

## 32. 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

> 인터페이스 내에 유니온 타입 속성을 사용하는 것보다는 인터페이스의 유니온 타입을 사용하는게 좋다.

```ts
interface Layer {
    layout: FillLayout | LineLayout | PointLayout;
    paint: FillPaint | LinePaint | PointPaint;
}
```

-   레이어 타입에 속성별로 각각 유니온 타입을 쓰면 잘못된 조합으로 섞일 위험이 있다.

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

-   각 레이어별 인터페이스를 정의하고, 이들을 유니온 타입으로 묶으면 속성값 조합 오류를 방지할 수 있다.
-   관련된 속성들을 함께 그룹화함으로써 타입좁히기 작업이 간결하고 안전해진다.

```ts
interface Person {
    name: string;
    birth?: {
        place: string;
        date: Date;
    };
}
```

-   선택적 속성이 서로 관련되어 같이 있거나 같이 없어야 되는 조건인 경우, 관련 속성을 하나의 객체로 묶어 표현하면 타입 관계가 명확해진다.
-   `birth` 속성 하나만 체크해주면 된다.

## 33. string 타입보다 더 구체적인 타입 사용하기

> `string`타입의 범위는 매우 넓기 때문에, 더 좁은 타입이 적절하지 않은지 검토해봐야 한다.

```ts
type RecordingType = 'studio' | 'live'; // string 타입보다 훨씬 좁은 타입!

interface Album {
    artist: string;
    title: string;
    releaseDate: Date;
    recordingType: RecordingType;
}
```

-   `recordingType`에 `'Studio'`나 '`August 17th 1959'`처럼 잘못된 값이 들어오는 것을 방지할 수 있음.

> 아무리 주석이 있더라도 실제 저장될 수 있는 값의 집합과 일치하지 않는 타입이 지정되어 있으면 무의미하다.

### 추가 장점

1. 타입의 의미를 명확히 하여 코드를 읽는 사람이 타입에 대해 이해하기 쉽다.
2. 함수 매개변수 등에 사용할 때, 타입을 좁게 지정하면 함수 사용자가 올바른 값만 전달하도록 안내할 수 있다.
3. `keyOf`를 사용하여 객체 속성명을 더 안전하게 다룰 수 있다.

## 34. 부정확한 타입보다는 미완성 타입을 사용하기

> 실수가 발생하기 쉬운 잘못된 타입은 타입이 없는 것보다 못하다.

-   타입이 구체적일 수록 버그를 많이 잡고 도구를 사용할 수 있지만, 타입의 정밀도를 높이는 과정에서 실수가 발생하기 쉽다.

## 35. 데이터가 아닌, API와 명세를 보고 타입 만들기

> 예시 데이터가 아닌 명세를 참고해서 타입을 생성하자. 예시 데이터는 모든 데이터를 대표하지 않을 수 있음!

## 36. 해당 분야의 용어로 타입 이름 짓기

> -   엄선된 변수명은 의도를 명확히 하고, 코드와 타입의 추상화 수준을 높여준다.
> -   가독성과 추상화 수준을 높이기 위해서는 해당 분야의 용어를 사용해야 한다.

### 타입 이름 짓는 규칙

-   코드로 표현하고자 하는 모든 분야에 전문 용어가 존재하기 때문에, 해당 용어를 사용해야 한다.
-   동일한 의미를 나타낼 떄는 **같은 용어**를 사용한다. 동의어를 사용하지 않는다.
-   `data`, `info`, `thing`, `item`, `object`, `entity`와 같이 모호하고 의미 없는 이름은 피해야 한다.
-   이름을 지을 때는 포함된 내용이나 계산 방식이 아닌 데이터 자체가 무엇인지를 고려해야 한다.

## 37. 공식 명칭에는 상표 붙이기

### 단점

-   타입에 '태그'를 지정하는 데 사용되는 `_brand` 프로퍼티는 **‘빌드 시점’**에만 존재하는 프로퍼티이다.
-   `_brand` 프로퍼티는 런타임에서는 표시되지 않기 때문에 개발자가 이를 사용하려고 하면 문제가 될 수 있다.
