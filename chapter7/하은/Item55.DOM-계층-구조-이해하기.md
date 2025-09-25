# 아이템 55 DOM 계층 구조 이해하기

## EventTarget의 계층 구조

### EventTarget (최상위)

```ts
// EventTarget의 기본 기능
element.addEventListener('click', handler);
element.removeEventListener('click', handler);
```

-   이벤트 처리의 가장 기본이 되는 계층
-   모든 DOM 요소가 이벤트를 받을 수 있는 이유임

### Node

-   DOM 트리 구조를 나타내는 계층
-   부모-자식 관계, 형제 관계 등을 처리

### Element

-   HTML 태그를 나타내는 계층
-   `classList`, `getAttribute()` 등의 기본적인 HTML 요소 조작 기능

### HTMLElement

-   HTML 전용 요소들의 공통 기능
-   `style`, `hidden`, `click()` 등
-   `HTML___ELEMENT`의 경우 자신만의 고유한 속성을 갖음

```ts
function handleDrag(eDown: Event) {
    const targetEl = eDown.currentTarget;

    // 타입 단언 사용
    // 클래스를 추가하려면 Element 단으로 타입이 좁혀져야 함
    (targetEl as Element).classList.add('dragging');

    // 또는 타입 가드 사용
    if (targetEl instanceof Element) {
        targetEl.classList.add('dragging');
    }
}
```

-   DOM 관련한 타입 단언문은 사용해도 괜찮음!

## Event의 계층 구조

`Event` 타입에도 계층 구조가 있으며, 이중 `Event`가 가장 추상화된 이벤트임.
