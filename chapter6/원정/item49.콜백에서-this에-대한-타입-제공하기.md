# 아이템 49 콜백에서 this에 대한 타입 제공하기

자바스크립트에서 this 키워드는 매우 혼란스러운 기능이다. let이나 const로 선언된 변수가 렉시컬 스코프(어휘적 범위, Lexical Environment라고도 함) 반면, this는 다이나믹 스코프이다. 다이나믹 스코프의 값은 '정의된' 방식이 아니라 '호출된'방식에 따라 달라진다.
this는 전형적으로 객체의 현재 인스턴스를 참조하는 클래스에서 가장 많이 쓰인다.

``` tsx
class C {
    vals = [1, 2, 3];
    logSquares() {
        for (const val of this.vals) {
            console.log(val * val);
        }
    }
}

const c = new C();
c.logSquares();
```

코드를 실행하면 다음처럼 출력된다.
1
4
9

이제 logSquares를 외부 변수에 넣고 호출하면 어떻게 되는지 보자

``` tsx
const c = new C();
const method = c.logSquares;
method();
```
이 코드는 런타임에 다음과 같은 오류가 발생한다.
> Uncaught TypeError: undefined의 'vals' 속성을 읽을 수 없습니다.

c.logSquares()가 실제로는 두 가지 작업을 수행하기 때문에 문제가 발생한다. C.prototype.logSquares를 호출하고, 또한 this의 값을 c로 바인딩한다. 앞의 코드에서는 logSquares의 참조 변수를 사용함으로써 두 가지 작업을 분리했고, this의 값은 undefined로 설정된다.

자바스크립트에는 this 바인딩을 온전히 제어할 수 있는 방법이 있다.
``` tsx
const c = new C();
const method = c.logSquares;
method.call(c); 
```
this가 반드시 C의 인스턴스에 바인딩되어야 하는 것은 아니며, 어떤 것이든 바인딩할 수 있다. 그러므로 라이브러리들은 API의 일부에서 this의 값을 사용할 수 있게 한다. 심지어 DOM에서도 this를 바인딩할 수 있다. 이벤트 핸들러를 예로 들 수 있다.
``` tsx
document.querySelector('input')!.addEventListener('change', function(e) {
    console.log(this); // 이벤트가 발생한 input 엘리먼트를 출력한다.
})
```

this 바인딩은 종종 콜백 함수에서 쓰인다. 예를 들어 클래스 내에 onClick 핸들러를 정의한다면 다음처럼 할 수 있다.
``` tsx
class ResetButton {
    render() {
        return makeButton({text: 'Rest', onClick: this.onClick});
    }
    onClick() {
        alert(`Reset ${this}`);
    }
}
```

그러나 ResetButton에서 onClick을 호출하면, this 바인딩 문제로 인해 "Reset 이 정의되지 않았다"라는 경고가 뜬다. 일반적인 해결책은 생성자에서 메서드에 this를 바인딩시키는 것이다.

``` tsx
class ResetButton {
    constructor() {
        this.onClick = this.onClick.bind(this);
    }
    render() {
        return makeButton({text: 'Rest', onClick: this.onClick});
    }
    onClick() {
        alert(`Reset ${this}`);
    }
}
```
onClick() { ... }은 ResetButton.prototype의 속성을 정의한다. 그러므로 ResetButton의 모든 인스턴스에서 공유된다. 그러나 생성자에서 this.onClick = ...으로 바인딩하면, onClick 속성에 this가 바인딩되어 해당 인스턴스에 생성된다. 속성 탐색 순서에서 onClick 인스턴스 속성은 onClick 프로토타입 속성보다 앞에 놓이므로, render() 메서드의 this.onClick은 바인딩된 함수를 참조하게 된다.

``` tsx
class ResetButton {
    render() {
        return makeButton({text: 'Rest', onClick: this.onClick});
    }
    onClick = () => {
        alert(`Reset ${this}`); // "this"가 항상 인스턴스를 참조한다.
    }
}
```

onClick을 화살표 함수로 바꿨다. 화살표 함수로 바꾸면, ResetButton이 생성될 때마다 제대로 바인딩 된 this를 가지는 새 함수를 생성하게 된다. 

``` js
class ResetButton {
    constructor() {
        var _this = this;
        this.onClick = function () {
            alert("Reset " + _this);
        };
    }
    render() {
        return makeButton({ text: 'Reset', onClick: this.onClick });
    }
}
```

this 바인딩은 자바스크립트의 동작이기 때문에, 타입스크립트 역시 this 바인딩을 그대로 모델링하게 된다. 만약 작성 중인 라이브러리에 this를 사용하는 콜백 함수가 있다면, this 바인딩 문제를 고려해야 한다.

이 문제는 콜백 함수의 매개변수에 this를 추가하고, 콜백 함수를 call로 호출해서 해결할 수 있다.

``` tsx
function addKeyListener (
    el: HTMLElement,
    fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
    el.addEventListener('keydown', e => {
        fn.call(el, e);
    });
}
```

콜백 함수의 첫 번째 매개변수에 있는 this는 특별하게 처리된다.
콜백 함수의 매개변수에 this를 추가하면 this 바인딩이 체크되기 때문에 실수를 방지할 수 있다.

``` tsx
function addKeyListener (
    el: HTMLElement,
    fn: (this: HTMLElement, e: KeyboardEvent) => void
) {
    el.addEventListener('keydown', e => {
        fn.(e);
        // ~~~~ 'void' 형식의 'this' 컨텍스트르르 메서드의 'HTMLElement' 형식 'this'에 할당할 수 없습니다.
    });
}
```
또한 라이브러리 사용자의 콜백 함수에서 this를 참조할 수 있고 완전한 타입 안전성도 얻을 수 있다.
만약 라이브러리 사용자가 콜백을 화살표 함수로 작성하고 this를 참조하려고 하면 타입스크립트가 문제를 잡아낸다.

``` tsx
class Foo {
    registerHandler(el: HTMLElement) {
        addKeyListener(el, e => {
            this.innerHTML;
            // ~~~~~~~~~~~ 'Foo' 유형에 'innerHTML' 속성이 없습니다.
        });
    }
}
```

this의 사용법을 반드시 기억해야 한다. 콜백 함수에서 this 값을 사용해야 한다면 this는 API의 일부가 되는 것이기 때문에 반드시 타입 선언에 포함해야 한다.

## 요약
- this 바인딩이 동작하는 원리를 이해해야 한다.
- 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시해야 한다.
