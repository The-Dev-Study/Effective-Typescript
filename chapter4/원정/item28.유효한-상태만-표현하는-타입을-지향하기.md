# 아이템 28 유효한 상태만 표현하는 타입을 지향하기
``` tsx
interface State {
    pageText: string;
    isLoading: boolean;
    error?: string;
}

fuction renderPage(state: State) {
    if (state.error) {
        reutrn `Error! Unable to load ${currentPage}: ${state.error}`;
    } else if (state.isLoading) {
        return `Loading ${currentPage}...`;
    }
    return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```
isLoading이 true이고 동시에 error 값이 존재하면 로딩 중인 상태인지 오류가 발생한 상태인지 명확히 구분할 수 없다.

``` tsx
async function changePage(state: State, newPage: string) {
    state.isLoading = true;
    try {
        const response = await fetch(getUrlForPage(newPage));
        if (!response.ok) {
            throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
        }
        const text = await response.text();
        state.isLoading = false;
        state.pageText = text;
    } catch(e) {
        state.error = '' + e;
    }
}
```
changePage에는 많은 문제점이 있다.
- 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져있다.
- state.error를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메세지 대신 과거의 오류 메세지를 보여 주게 된다.
- 페이지 로딩 중에 사용자가 페이지를 바꿔 버리면 어떤 일이 벌어질지 예상하기 어렵다. 새 페이지에 오류가 뜨거나, 응답이 오는 순서에 따라 두 번째 페이지가 아닌 첫 번째 페이지로 전환될 수도 있다.

문제는 상태 값의 두 가지 속성이 동시에 정보가 부족하거나, 두 가지 속성이 충돌(오류이면서 동시에 로딩 중일 수 있다.)할 수 있따는 것이다. State 타입은 isLoading이 true이면서 동시에 error 값이 설정되는 무효한 상태를 허용한다. 무효한 상태가 존재하면 render()와 changePage() 둘 다 제대로 구현할 수 없게 된다.

``` tsx
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
    currentPage: string;
    requests: {[page: string]: RequestState};
}
```
여기서는 네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온(또는 구별된 유니온)이 사용되었다. 코드의 길이가 길어지긴 했지만, 무효한 상태를 허용하지 않도록 개선되었다. 그 결과로 개선된 renderPage와 changePage 함수는 쉽게 구현할 수 있다.

``` tsx
function renderPage(state: State) {
    const { currentPage } = state;
    const requestState = state.requests[currentPage];
    switch (requestState.state) {
        case 'pending':
            return `Loding ${currentPage}...`;
        case 'error':
            return `Error! Unable to load ${currentPage}: ${requestState.error}`;
        case 'ok':
            return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
    }
}

async function changePage(state: State, newPage: string) {
    state.requests[newPage] = { state: 'pending' };
    state.currentPage = newPage;
    try {
        const response = await fetch(getUrlForPage(newPage));
        if (!response.ok) {
            throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
        }
        const pageText = await response.text();
        state.requests[newPage] = {state: 'ok', pageText};
    } catch(e) {
        state.requests[newPage] = {state: 'error', error: '' + e};
    }
}
```
## 요약
- 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽고 오류를 유발하게 된다.
- 유효한 상태만 표현하는 타입을 지향해야 한다.