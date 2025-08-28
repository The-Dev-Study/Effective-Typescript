# 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

타입스크립트는 타입을 추론할 때 단순히 값만 고려하지 않는다. 값이 존재하는 곳의 문맥까지도 살핀다. 그런데 문맥을 고려해 타입을 추론하면 가끔 이상한 결과가 나온다. 

``` typescript
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) {}

setLanguage('JavaScript'); // 정상

let language = 'JavaScript';
setLanguage(language);
// 오류!
```

인라인 형태에서 타입스크립트는 함수 선언을 통해 매개변수가 Language 타입이어야 한다는 것을 알고 있다. 그러나 이 값을 변수로 분리해내면, 타입스크립트는 할당 시점에 타입을 추론한다. 이번 경우에는 string으로 추론했고 Language 타입으로 할당이 불가능하므로 오류가 발생했다.

이런 문제를 해결하는 두 가지 방법이 있다.

### 타입 선언에서 languae의 가능한 값을 제한하기
``` typescript
let language: Language = 'JavaScript';
setLanguage(language); // 정상
```

### language를 상수로 만들기

``` typescript
const language = 'JavaScript';
setLanguage(language); // 정상
```

const를 사용하여 타입 체커에서 language는 변경할 수 없다고 알려준다. 따라서 타입스크립트는 language에 대해 더 정확한 타입인 문자열 리터럴 JavaScript로 추론할 수 있다.

## 튜플 사용 시 주의점
문자열 리터럴 타입과 마찬가지로 튜플 타입에서도 문제가 발생한다.

``` typescript
function panTo(where: [number, number]) {}

panTo([10, 20]); // 정상

const loc = [10, 20];
panTo(loc); // 오류!
```
첫번째 경우는 할당이 가능하지만 두번째 경우는 타입스크립트가 loc의 타입을 number[]로 추론한다.

``` typescript
const loc: [number, number] = [10, 20];
panTo(loc); // 정상
```
상수 문맥을 제공함으로써 문제를 해결할 수 있다.

const는 단지 값이 가리키는 참조가 변하지 않는 얕은(shallow) 상수인 반면, as const는 그 값이 내부까지(deeply) 상수라는 사실을 타입스크립트에게 알려준다.

그러나 as const를 사용한 추론은 너무 과하게 정확하기 때문에 기존 함수의 시그니처를 수정하지 않으면 동작하지 않는다.

``` typescript
function panTo(where: readonly [number, number]){}
const loc = [10, 20] as const;
panTo(loc); // 정상
```

as const는 문맥 손실과 관련한 문제를 깔끔하게 해결할 수 있지만, 한 가지 문제가 있다.

타입 정의에 실수가 있다면 (예를 들어, 튜플에 세번째 요소를 추가한다면) 오류는 타입 정의가 아니라 호출되는 곳에서 발생한다. 이는 근본적인 원인을 파악하기 어렵게 한다.

## 객체 사용 시 주의점
문맥에서 값을 분리하는 문제는 문자열 리터럴이나 튜플을 포함하는 큰 객체에서 상수를 뽑아낼 때도 발생한다.

``` typescript
type Language = 'JavaScript' | 'TypeScript' | 'Python';

interface GovernedLanguage {
    language: Language;
    organization: string;
}

function complain(language: GovernedLanguage) {}

complain({
    language: 'TypeScript',
    organization: 'Microsoft'
}); // 정상

const ts = {
    language: 'TypeScript',
    organization: 'Microsoft'
};

complain(ts);
```

ts 객체에서 language의 타입은 string 으로 추론된다. 이 문제느느 타입 선언을 추가하거나 상수 단언을 사용해 해결한다.

## 콜백 사용 시 주의점
콜백을 다른 함수로 전달할 때, 타입스크립트는 콜백의 매개변수 타입을 추론하기 위해 문맥을 사용한다.

``` typescript
function callWithRandomNumbers(fn (n1: number, n2: number) => void) {
    fn(Math.random(), Math.random());
}

callWithRandomNumbers((a, b) => {
    a; // 타입이 number
    b; // 타입이 number

    console.log(a+b);
});
```

callWithRandom의 타입 선언으로 인해 a와 b의 타입이 number로 추론된다. 그러나 콜백을 상수로 뽑아내면 문맥이 소실된다.

이런 경우는 매개변수에 타입 구문을 추가해서 해결할 수 있다.

## 요약
- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 한다.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야 한다.
- 변수가 정말로 상수라면 상수 단언을 사용해야 한다.