# 아이템 39. any를 구체적으로 변형해서 사용하기

`any`타입은 <ins>자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입</ins>이다.

> 일반적인 상황에서 `any`보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 있다. 이왕이면 구체적인 타입을 찾아 타입 안전성을 높이자.

## 배열 타입을 정확히 모를 때

```ts
function getLengthBad(array: any) {
    return array.length;
}
```

-   `array`의 타입이 배열인지 알 수 없다.

```ts
function getLength(array: any[]) {
    return array.length;
}
```

-   `array`의 타입이 배열인지 알 수 있다.
-   함수 내부의 `array.length` 타입이 `number`로 체크된다.
-   반환 타입 또한 `number`로 추론된다.
-   함수가 호출될 때 매개변수의 타입이 배열인지 체크된다.

## 객체의 값을 정확히 모를 때

### (1) `{ [key: string]: any }` 사용하기

```ts
function hasTwelveLetterKey(o: { [key: string]: any }) {
    for (const key in o) {
        if (key.length === 12) {
            return true;
        }
    }
    return false;
}
```

-   함수의 매개변수가 객체이지만, 정확히 그 값을 알수 없다면 `{[key: string]:any}`로 선언한다.

### (2) `object` 타입 사용하기

`object` 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다.

```ts

```
