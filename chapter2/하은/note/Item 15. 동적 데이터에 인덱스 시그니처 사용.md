# 인덱스 시그니처

## 인덱스 시그니처란

## `Record<Keys, Type>` 사용하기

`Record<Keys, Type>`는 **객체 타입**을 만드는 유틸리티 타입이다.

-   `Keys`: 객체의 키들
-   `Type`: 모든 키에 대응하는 값의 타입

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```
