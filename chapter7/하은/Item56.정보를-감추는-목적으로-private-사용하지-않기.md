# 아이템 56 정보를 감추는 목적으로 private 사용하지 않기

자바스크립트는 클래스에 비공개 속성을 만들 수 없다. `_`를 접두사로 붙여 비공개 속성임을 나타내는게 관례

> `_`를 붙였더라도 클래스 외부로 공개돼 있음

`public`, `protected`, `private` 접근 제어자는 컴파일 시점에 오류를 표시해주지만 컴파일 후에는 제거된다.

따라서 정보를 감추기 위해서 `private`를 사용하면 안된다.

## 클로저를 사용해 정보 숨기기

```ts
declare function hash(text: string): number;
class PasswordChecker {
    checkPassword: (password: string) => boolean;
    constructor(passwordHash: number) {
        //클로저 생성
        this.checkPassword = (password: string) => {
            return hash(password) === passwordHash; // passwordHash를 기억하지만, 외부에 공개되지 않음
        };
    }
}

const checker = new PaaswordChecker(hash('s3cret'));
checker.checkPassword('s3cret');
```

-   인스턴스가 생성될 때마다 메서드의 복사본이 생기므로 메모리를 낭비한다는 단점이 있음
-   인스턴스끼리 비공개 데이터에 접근하는게 불가능해서 불편함이 존재함

## `#`붙이기

접두사 `#`을 붙이면 타입체크와 런타임에서 비공개로 만들 수 있다.

```ts
class PasswordChecker {
    #passwordHash: number;

    constructor(passwordHash: number) {
        this.#passwordHash = passwordHash;
    }

    checkPassword(password: string): boolean {
        return hash(password) === this.#passwordHash;
    }
}
```

-   외부에서 접근이 불가능하다는 건 동일하지만, 클로저와 달리 동일한 클래스의 개별 인스턴스끼리 접근이 가능함
