# 아이템 56 정보를 감추는 목적으로 private 사용하지 않기

자바스크립트는 클래스에 비공개 속성을 만들 수 없다. 많은 이가 비공개 속성을 나타내기 위해 언더스코어(_)를 접두사로 붙이던 것이 관례로 인정될 뿐이었다.
``` js
class Foo {
    _private = 'secret123';
}
```

그러나 속성에 언더스코어를 붙이는 것은 단순히 비공개라고 표시한 것뿐이다. 따라서 일반적인 속성과 동일하게 클래스 외부로 공개되어 있다는 점을 주의해야 한다.

``` js
const f = new Foo();
f._private; // 'secret123'
```

타입스크립트에는 public, protected, private 접근 제어자를 사용해서 공개 규칙을 강제할 수 있는 것으로 오해할 수 있다.

``` tsx
class Diary {
    private secret = 'cheated on my English test';
}

const diary = new Diary();
diary.secret
    //~~~~~~ 'secret' 속성은 private이며
    //       'Diary' 클래스 내에서만 접근할 수 있습니다.
```
그러나 public, protected, private 같은 접근 제어자는 타입스크립트 키워드이기 때문에 컴파일 후에는 제거된다.

타입스크립트의 접근 제어자들은 단지 컴파일 시점에만 오류를 표시해 줄 뿐이며, 언더스코어 관례와 마찬가지로 런타임에는 아무런 효력이 없다. 심지어 단언문을 사용하면 타입스크립트 상태에서도 private 속성에 접근할 수 있다.

``` tsx
class Diary {
    private secret = 'cheated on my English test';
}

const diary = new Diary();
(diary as any).secret //정상
```

즉 정보를 감추기 위해 private을 사용하면 안된다.
자바스크립트에서 정보를 숨기기 위해 가장 효과적인 방법은 클로저를 사용하는 것이다. 다음 코드처럼 생성자에서 클로저를 만들어 낼 수 있다.

``` tsx
declare function hash(text: string): number;

class PasswordChecker {
    checkPassword: (password: string) => boolean;
    constructor(passwordHash: number) {
        this.checkPassword = (password: string) => {
            return hash(password) === passwordHash
        }
    }
}

const checker = new PasswordChecker(hash('s3cret'));
checker.checkPassword('s3cret'); // 결과는 true
```

앞의 코드를 살펴보면 PasswordChecker의 생성자 외부에서 passwordHash 변수에 접근할 수 없기 때문에 정보를 숨기는 목적을 달성했다. 그런데 몇가지 주의사항이 있따. passwordHash를 생성자 외부에서 접근할 수 없기 때문에, passwordHash에 접근해야 하는 메서드 역시 생성자 내부에 정의되어야 한다. 그리고 메서드 정의가 생성자 내부에 존재하게 되면, 인스턴스를 생성할 때마다 각 메서드의 복사본이 생성되기 때문에 메모리를 낭비하게 된다는 것을 기억해야 한다. 또한 동일한 클래스로부터 생성된 인스턴스라고 하더라도 서로의 비공개 데이터에 접근하는 것이 불가능하기 때문에 철저하게 비공개면서 동시에 불편함이 따른다.

또 하나의 선택지로, 현재 표준화가 진행 중인 비공개 필드 기능을 사용할 수도 있다. 비공개 필드 기능은 접두사로 #를 붙여서 타입 체크와 런타임 모두에서 비공개로 만드는 역할을 한다.

``` tsx
class PasswordChecker {
    #passwordHash: number;

    constructor(passwordHash: number) {
        this.#passwordHash = passwordHash;
    }

    checkPassword(password: string) {
        return hash(password) === this.#passwordHash;
    }
}

const checker = nw PasswordChecker(hash('s3cret'));
checker.checkPassword('secret'); // 결과는 false
checker.checkPassword('s3cret'); // 결과는 true
```

#passwordHash 속성은 클래스 외부에서 접근할 수 없다. 그러나 클로저 기법과 다르게 클래스 메서드나 동일한 클래스의 개별 인스턴스끼리는 접근이 가능하다. 비공개 필드를 지원하지 않는 자바스크립트 버전으로 컴파일하게 되면, WeapMap을 사용한 구현으로 대체된다. 어쨌든 구현방식과 무관하게 데이터는 동일하게 비공개이다. 
만약 설계 관점의 캡슐화가 아닌 '보안'에 대해 걱정하고 있다면 내장된 프로토타입과 함수에 대한 변조 같은 문제를 알고 있어야 한다.

## 요약
- public, protected, private 접근 제어자는 타입 시스템에서만 강제될 뿐이다. 런타임에는 소용이 없으며 단언문을 통해 우회할 수 있다. 접근 제어자로 데이터를 감추려고 해서는 안된다.
- 확실히 데이터를 감추고 싶다면 클로저를 사용해야 한다.