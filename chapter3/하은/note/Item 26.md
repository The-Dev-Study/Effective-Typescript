# 타입 추론에 문맥이 어떻게 사용되는지 이해하기

## 문자열 리터럴에서의 추론

```ts
type Language = 'TypeScript' | 'JavaScript';
let language1 = 'JavaScript'; // string 타입으로 추론
let language2: Language = 'JavaScript'; // 명시적으로 타입 선언
const language3 = 'JavaScript'; // string literal 타입으로 추론

function setLanguage(language: Language) {
    console.log(`선택한 언어는 ${language}`);
}

setLanguage('JavaScript'); //됨 - 문맥과 값을 분리하지 않았기 때문에 잘 추론됨
setLanguage(language1); //안됨
setLanguage(language2); //됨
setLanguage(language3); //됨
```

## 튜플에서의 추론

튜플에서도 문맥과 값을 분리하면 변수를 선언할 당시에는 함수의 문맥을 고려할 수 없기 때문에 단순히 저장되는 값을 바탕으로 타입을 추론한다.

```ts
function panTo(where: [number, number]) {}

panTo([10, 20]); //panTo의 매개변수 타입을 바탕으로 [number, number]로 추론

const loc = [10, 20]; // number[]로 추론
panTo(loc); // 에러남
```

배열의 경우 const로 선언한다고 해도 이 오류를 고칠 수 없다.

### 상수 문맥 제공하기

`as const`를 사용하면 참조만 변하지 않는 얕은 상수가 아니라 내부까지 깊은 상수로 만들 수 있다.

```ts
const loc = [10, 20] as const; // readonly [10, 20]로 추론
panTo(loc); // 그래도 에러남 ㅋㅋ -> 함수에서 매개변수를 readonly [number, number]로 바꿔주면 됨
```

-   근데 이렇게 하면 만약에 loc을 잘못 선언해서 readonly [10, 20, 30]으로 정의한 경우 정의하는 부분이 아니라 함수 호출 부분에서 에러가 난다는 단점이 있음

그냥 내가 봤을 때 변수에 타입 지정해주는게 나을 것 같음 ㅇㅇ

### 객체 사용시 추론

그냥 비슷한 내용임

### 콜백을 매개변수로 넘기는 경우

콜백을 매개변수로 넘기는 경우에도 따로 생성하고 참조값을 넘기는 것과~는 다름

```ts
function callWithRandomNumbers(fn: (n1: number, n2: number) => void) {
    fn(Math.random(), Math.random());
}

// callWithRandomNumbers의 매개변수 타입을 바탕으로 a와 b의 타입을 number로 추론
callWithRandomNumbers((a, b) => {
    console.log(a + b);
});

// a와 b를 any 타입으로 추론함
const myFunc = (a, b) => {
    console.log(a + b);
};
callWithRandomNumbers(myFunc);
```

이 경우 매개변수에 타입 구문을 추가해서 해결이 가능하다.

```ts
const myFunc = (a: number, b: number) => {
    console.log(a + b);
};
```
