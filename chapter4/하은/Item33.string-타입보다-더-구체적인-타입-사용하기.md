# 아이템 33. string 타입보다 더 구체적인 타입 사용하기

> `string`타입의 범위는 매우 넓기 때문에, 더 좁은 타입이 적절하지 않은지 검토해봐야 한다.

```ts
interface Album {
    artist: string;
    title: string;
    releaseDate: string; // YYYY-MM-DD
    recordingType: string; // 'live' 또는 'studio'
}
```

-   `recordingType`에 'studio'가 아닌 'Studio'가 들어오더라도 이를 막을 방법이 없음
-   `releaseDate`에 `YYYY-MM-DD` 형식이 아닌 'August 17th 1959'가 들어오더라도 검토할 수 없음
-   아무리 주석이 있더라도 실제 저장될 수 있는 값의 집합과 일치하지 않는 타입이 지정되어 있으면 무의미

```ts
type RecordingType = 'studio' | 'live';

interface Album {
    artist: string;
    title: string;
    releaseDate: Date;
    recordingType: RecordingType;
}
```

-   이렇게 바꾸면 타입스크립트가 오류를 더 세밀하게 체크할 수 있다.

## 더 구체적인 타입을 사용하는 것의 장점(180)

### 1. 다른 곳에서 사용할 때에도 타입 정보를 유지할 수 있다.

```ts
function getAlbumsOfType(recordingType: string): Album[] {}
```

-   함수를 사용하는 사람은 `recordingType`이 'studio'또는 'live'중 하나라는 걸 알 수 없다.

```ts
function getAlbumsOfType(recordingType: RecordingType): Album[] {}
```

-   `recordingType`이 'studio'또는 'live'중 하나라는 걸 확신할 수 있다.

### 2. 타입을 명시적으로 정의하고, 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.

```ts
/** 이 녹음은 어떤 환경에서 이루어졌는지? **/
type RecordingType = 'live' | 'studio';
```

### 3. `keyof`연산자를 통한 세밀한 객체 속성 체크

`keyof` 연산자를 통해 더욱 세밀한 객체 속성 체크가 가능해진다.

-   특정 필드 값 뽑고 싶은 경우에 다음과 같이 코드를 작성할 수 있다.

```ts
function pluck(records: any[], key: string): any[] {
    return records.map((r) => r[key]);
}
```

-   반환값에 `any`를 사용하는 건 좋지 않은 타입 설계이다.

```ts
function pluck<T>(records: T[], key: keyof T): T[keyof T][] {
    return records.map((r) => r[key]);
}
```

-   이렇게 하면 `key`가 `T`타입의 키임을 보장할 수 있다.
-   그러나 `T[keyof T][]`의 타입은 `(string | Date)][]`가 된다.
    -   `Album[releaseDate]` 타입은 `'studio' | 'live'`타입이지만, 상위 집합과 하위 타입의 합집합은 **상위 타입**이 되므로 그냥 `string`만 남는 거임

```ts
interface Album {
    artist: string;
    title: string;
    releaseDate: Date;
    recordingType: RecordingType;
}

const albums: Album[] =[...]

function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    return records.map((r) => r[key]);
}

pluck(albums, 'artist');
```

-   K가 구체적으로 지정된 타입 파라미터이기 때문에, 호출 시점에 key 값이 구체적인 하나의 키로 추론되고, 반환 타입이 해당 키에 맞는 구체적인 값 타입 배열이 된다.

## 요약

-   문자열을 남발하여 선언된 코드를 피하자.
-   string타입보다 구체적인 타입을 쓸 수 있다면 쓰도록하자.
-   객체의 속성 이름을 함수의 매개변수로 받을 때에는 string 보다 keyof T를 사용한다.
