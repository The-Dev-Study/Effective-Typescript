# 아이템 33 string 타입보다 더 구체적인 타입 사용하기

string의 타입 범위는 매우 넓다. string타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지 않을지 검토해 보아야 한다.

음악 컬렉션을 만들기 위해 앨범의 타입을 정의한다고 가정해보자.

``` tsx
interface Album {
    artist: string;
    title: string;
    releaseDate: string;
    recordingType: string;
}
```

string 타입이 남발된 모습이다. 게다가 주석에 타입 정보를 적어 둔 걸 보면 현재 인터페이스가 잘못되었다는 것을 알 수 있다.
Album에 엉뚱한 값을 설정할 수 있다.

string 타입의 범위가 매우 넓기 때문에 제대로 된 Album 객체를 사용하더라도 매개변수 순섯가 잘못된 것이 오류로 드러나지 않는다.

앞의 예제처럼 string 타입이 남용된 코드를 "문자열을 남발하여 선언되었다."고 표현하기도 한다. 앞의 오류를 방지하기 위해 타입의 범위를 좁히는 방법을 생각해보자

``` tsx
type RecordingType = 'studio' | 'live';

interface Album {
    artist: string;
    title: string;
    releaseDate: Date;
    recordingType: RecordingType;
}
```

앞의 코드처럼 바꾸면 타입스크립트는 오류를 더 세밀하게 체크한다.

이러한 방식에는 세가지 장점이 있다.

첫 번째 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다. 예를 들어, 특정 레코딩 타입의 앨범을 찾는 함수를 작성한다면 다음처럼 정의할 수 있다.
``` tsx
function getAlbumsOfType(recordingType: string): Album[] {
    // ...
}
```

getAlbumsOfType 함수를 호출하는 곳에서 recordingType의 값이 string 타입이어야 한다는 것 외에는 다른 정보가 없다.
주석으로 써놓은 "studio" 또는 "live"는 Album의 정의에 숨어 있고, 함수를 사용하는 ㅏ람은 recordingType이 studio 또는 live여야 한다는 것을 알 수 없다.


두 번째, 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여넣을 수 있다.
``` tsx
// 이 녹음은 어떤 환경에서 이루어졌는지?
type RecordingType = 'live' | 'studio';
```

getAlbumsOfType이 받는 매개변수를 string 대신 RecordingType으로 바꾸면 함수를 사용하는 곳에서 RecordingType의 설명을 볼 수 있다.

세 번째, keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.
함수의 매개변수에 string을 잘못 사용하는 일은 흔하다. 어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 생각해보자 

``` tsx
function pluck(records, key) {
    return records.map(r => r[key]);
}
```

pluck 함수의 시그니처를 다음처럼 작성할 수 있다.

``` tsx
function pluck(records: any[], key: string): any[] {
    return records.map(r => r[key]);
}
```

타입 체크가 되긴 하지만 any 타입이 있어서 정밀하지 못하다. 특히 반환 값에 any를 사용하는 것은 매우 좋지 않은 설계이다. 먼저 타입 시그니처를 개선하는 첫 단계로 제너릭 타입을 도입해보자

``` tsx
function pluck<T>(records: T[], key: string): any[] {
    return records.map(r => r[key]);  // 암시적으로 any가 있습니다.
}
```

이제 타입스크립트는 key의 타입이 string이기 때문에 범위가 너무 넓다는 오류를 발생시킨다. Album의 배열을 매개변수로 전달하면 기존의 string 타입의 넓은 범위와 반대로 key는 단 네 개의 값만이 유효하다.
다음 예시는 keyof Album 타입으로 얻는 결과이다.

``` tsx
type K = keyof Album;
// 타입이 "artist" | "title" | "releaseDate" | "recordingType"
```

그러므로 string을 keyof T로 바꾸면 된다.

``` tsx
function pluck<T>(records: T[], key: keyof T) {
    return records.map(r => r[key]);
}
```

이 코드는 타입 체커를 통과한다. 또한 타입스크립트가 반환 타입을 추론할 수 있게 해준다. pluck 함수에 마우스를 올려보면 추론된 타입을 알 수 있다.
``` tsx
function pluck<T>(records: T[], key: string): T[keyof T][] {}
```
T[keyof T] T객체 내의 가능한 모든 값의 타입이다. 그런데 key의 값으로 하나의 문자열을 넣게 되면 그 범위가 너무 넓어서 적절한 타입이라고 보기 어렵다.
예를 들어보자

``` tsx
const releaseDates = pluck(albums, 'releaseDate'); // 타입이 (string | Date)[]
```

releaseDates의 타입은 (string | Date)[]가 아니라 Date[]이어야 한다. keyof T는 string에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓다. 따라서 범위를 더 좁히기 위해, keyof T의 부분 집합으로 두번째 제너릭 매개변수를 도입해야 한다.

``` tsx
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    return records.map(r => r[key]);
}
```

매개변수 타입이 정밀해진 덕분에 언어 서비스는 Album의 키에 자동 완성 기능을 제공할 수 있게 해준다.
string은 any와 비슷한 문제를 가지고 있다. 따라서 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감추어 버린다. 

## 요약
- 문자열을 남발하여 선언된 코드를 피하자
- 변수의 범위를 보다 정확하게 표현하고 싶다면 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 된다.
- 객체 속성 이름을 함수 매개변수로 받을 때는 string보다 keyof T를 사용하는 것이 좋다.