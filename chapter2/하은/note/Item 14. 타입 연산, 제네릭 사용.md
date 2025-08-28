# 타입 연산과 제네릭 사용

`DRY` 원칙을 타입 정의에서 사용할 수 있다. 이 때 **제네릭 타입**을 사용한다.

```ts
interface State {
    userId: string;
    pageTitle: string;
    recentFiles: string[];
    pageContents: string;
}

interface TopNavState {
    userId: string;
    pageTitle: string;
    recentFiles: string[];
}
```

-   `TopNavState`는 `state`의 `pageContents`를 제외한 모든 필드를 갖는 인터페이스

## 원하는 필드만 뽑기

[인덱스 접근 타입](https://www.typescriptlang.org/ko/docs/handbook/2/indexed-access-types.html)

```ts
type TopNavState = {
    userId: State['userId'];
    pageTitle: State['pageTitle'];
    recentFiles: State['recentFiles'];
};
```

```ts
type TopNavState = { [k in Exclude<keyof State, 'pageContents'>]: State[k] };
```

## 기존 타입에서 선택적 프로퍼티로 수정하기

```ts
interface Options {
    width: number;
    height: number;
    color: string;
    label: string;
}

type OptionsUpdate = { [k in keyof Options]?: Options[k] };
```

## ReturnType<함수의 Type>

> `ReturnType`의 적용 대상이 **타입**임에 유의하자. 값인 `getUserInfo`에 적용된게 아니다.

함수나 메서드의 반환값을 바탕으로 명명된 타입을 만들고 싶으면 `ReturnType<Type>`을 사용하면 된다.

[Return 타입](https://www.typescriptlang.org/ko/docs/handbook/utility-types.html)

```ts
function getUserInfo(userId: string) {
    return {
        userId,
        name,
        age,
    };
}

type getUserInfoResType = ReturnType<typeof getUserInfo>;
```

## 제네릭 타입을 제한하는 법

제네릭 타입으로 들어올 수 있는 타입을 제한하고 싶은 경우, `extends`를 사용한다.

```ts
interface Name {
    first: string;
    last: string;
}

type DancingDuo<T extends Name> = [T, T];

type PersonDuo = DancingDuo<Person>;
// 결과: [Person, Person]
```

## Pick

```ts
type Pick<T, K> = { [k in K]: T[k] };
```
