# 아이템 28 유효한 상태만 표현하는 타입을 지향하기

타입을 작성할 때에는 분기 조건을 명확히 분리해야 한다.

```ts
interface State {
    pageText: string;
    isLoading: boolean;
    error?: string;
}
```

-   `isLoading`이 `true`면서 `error`값이 존재할 수 있는 상황
-   **무효한 상태**가 없도록 재설계가 필요함

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

interface State {
    pageText: string;
    requests: { [page: string]: RequestState };
}
```

-   네트워크 요청을 ` pending``,error `, `success` 각각 하나의 타입으로 설계하고 유니언 타입으로 사용함
