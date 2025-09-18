# 아이템 48 API 주석에 TSDoc 사용하기

다음은 인사말을 생성하는 타입스크립트 함수이다.

``` tsx
function greet(name: string, title: string) {
    return `Hello ${title} ${name}`;
}
```

함수의 앞부분에 주석이 있어 함수가 어떤 기능을 하는지 쉽게 알 수는 있다. 그러나 사용자를 위한 문서라면 JSDoc 스타일의 주석으로 만드는 것이 좋다.

``` tsx
/** 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다. */
function greet(name: string, title: string) {
    return `Hello ${title} ${name}`;
}
```

왜냐하면 대부분의 편집기는 함수가 호출되는 곳에서 함수에 붙어 있는 JSDoc 스타일의 주석을 툴팁으로 표시해 주기 때문이다.

타입스크립트 언어 서비스가 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋다. 만약 공개 API에 주석을 붙인다면 JSDoc 형태로 작성해야 한다. 한편 타입스크립트 관점에서는 TSDoc이라고 부르기도 한다.

``` tsx
/**
 * 인사말을 생성한다.
 * @param name 인사할 사람의 이름
 * @param title 그 사람의 칭호
 * @returns 사람이 보기 좋은 형태의 인사말
 */
 function greet(name: string, title: string) {
    return `Hello ${title} ${name}`;
}
```

타입 정의에 TSDoc을 사용할 수도 있다.
``` tsx
/** 특정 시간과 장소에서 수행된 측정 */
interface Measurement {
    /** 어디에서 측정되었나? */
    position: Vector3D;
    /** 언제 측정되었나? */
    time: number;
    /** 측정된 운동량 */
    momentum: Vector3D;
}
```

Measurement 객체의 각 필드에 마우스를 올려 보면 필드별로 설명을 볼 수 있다.
TSDoc 주석은 마크다운 형식으로 꾸며지므로 굵은 글씨, 기울임 글씨, 글머리 기호 목록을 사용할 수 있다.

주석을 수필처럼 장황하게 쓰지 않도록 주의해야 한다. 휼륭한 주석은 간단히 요점만 언급한다.

JSDoc에는 타입 정보를 명시하는 규칙이 있지만, 타입스크립트에서는 타입 정보가 코드에 있기 때문에 TSDoc에서는 타입 정보를 명시하면 안 된다.

## 요약
- 익스포트된 함수, 클래스, 타입에 주석을 달 때는 JSDoc/TSDoc 형태를 사용하자
- @param, @returns 구문과 문서 서식을 위해 마크다운을 사용할 수 있다.
- 주석에 타입 정보를 포함하면 안 된다.