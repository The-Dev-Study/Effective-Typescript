# 아이템 22 타입 좁히기
타입 넓히기의 반대는 타입 좁히기이다. 타입 좁히기는 타입스크립트가 넓은 타입으로부터 좁은 타입으로 진행하는 과정을 말한다.

가장 일반적인 예시는 null 체크이다.

## 타입을 좁히는 여러 방법
- 분기문에서 예외를 던지거나 함수를 반환하여 타입 좁히기
- instanceof를 사용해서 타입을 좁히기

```typescript
fucntion contains(text: string, search: string|RegExp) {
    if (search instanceofRegExp) {
        search // 타입이 RegExp
    }
    search // 타입이 string
}
```

- 속성 체크로 타입 좁히기

``` typescript
interface A {a: number};
interface B {b: number}
function pickAB(ab: A | B) {
    if ('a' in ab) {
        ab // 타입이 a
    } else {
        ab // 타입이 B
    }
    ab // 타입이 A| B
}
```

- 내장 함수로 타입 좁히기
``` typescript 
function contains(text: string, terms: string|string[]) {
    const termList = Array.isArray(terms) ? terms : [terms];
    termList // 타입이 string[]
}
```

- 태그 붙이기
``` typescript
interface UploadEvent { type: 'upload'; filename: string; contents: string;};
interface DownloadEvent { type: 'download'; filename: string;};

type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
    switch (e.type) {
        case 'download':
            e // 타입이 DownloadEvent
            break;
        case 'upload':
            e // 타입이 UploadEvent
            break;
    }
}
```

이 패턴은 태그된 유니온(tagged union) 또는 구별된 유니온 (discriminated union)이라고 불리며, 타입스크립트 어디에서나 찾아볼 수 있다.

- 사용자 정의 타입 가드
반환 
<br> 타입의 el is HTMLInputElement는 함수의 반환이 true인 경우, 타입 체커에게 매겨 변수의 타입을 좁힐 수 있다고 알려준다.
``` typescript 
function isInputElement(el: HTMLElement): el is HTMLInputElement {
    return 'value' in el;
}

function getElementContent(el: HTMLElemnt) {
    if (isInputElement(el)) {
        el; // 타입이 HTMLInputElement
    }
    el; // 타입이 HTMLElement
}
```

## 요약
- 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야 한다.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있다.