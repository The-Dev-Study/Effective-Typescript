# 아이템 51 의존성 분리를 위해 미러 타입 사용하기

CSV 파일을 파싱하는 라이브러리를 작성한다고 가정해 보겠다. parseCSV API는 간단하다. CSV 파일의 내용을 매개변수로 받고 열 이름을 값으로 매핑하는 객체들을 생성하여 배열로 반환한다. 그리고 NodeJS 사용자를 위해 매겨변수에 Buffer 타입을 허용한다.

``` tsx
function parseCSV(contents: string | Buffer): {[column: string]: string}[] {
    if (typeof contents === 'object') {
        // 버퍼인 경우
        return parseCSV(contents.toString('utf8'));
    }
}
```

앞에서 작성한 CSV 파싱 라이브러리를 공개하면 타입 선언도 포함하게 된다. 그리고 타입 선언이 @types/node에 의존하기 때문에 @types/node는 devDependencies로 포함해야 한다. 그러나 @types/node를 devDependencies로 포함하면 다음 두 그룹의 라이브러리 사용자들에게 문제가 생긴다.
- @types와 무관한 자바스크립트 개발자
- NodeJS와 무관한 타입스크립트 웹 개발자

두 그룹의 사용자들은 각자가 사용하지 않는 모듈이 포함되어 있기 때문에 혼란스러울 거다. Buffer는 NodeJS 개발자에게만 필요하다. 그리고 @types/node는 NodeJS와 타입스크립트를 동시에 사용하는 개발자만 관련된다.

각자가 필요한 모듈만 사용할 수 있도록 구조적 타이핑을 적용할 수 있다. @types/node에 있는 Buffer 선언을 사용하지 않고, 필요한 메서드와 속성만 별도로 작성할 수 있다. 앞선 예제의 경우 인코딩 정보를 매개변수로 받는 toString 메서드를 가지는 인터페이스를 별도로 만들어 사용하면 된다.

``` tsx
interface CsvBuffer {
    toString(encoding: string): string;
}

function parseCSV(content: string | CsvBuffer): {[column: string]: string}[] {
    // ...
}
```

CsvBuffer는 Buffer 인터페이스보다 훨씬 짧으면서도 실제로 필요한 부분만을 떼어 내어 명시했다. 또한 해당 타입이 Buffer와 호환되기 때문에 NodeJS 프로젝트에서는 실제 Buffer 인스턴스로 parseCSV를 호출하는 것이 가능하다.

``` tsx
parseCSV(new Buffer("column 1, column2\n val1, val2, "utf-8")); //정상
```

만약 작성 중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것 (미러링)을 고려해 보는 것도 좋다.

NodeJS 기반 타입스크립트 사용자에게는 변화가 없지만, 웹 기반이나 자바스크립트 등 다른 모든 사용자에게는 더 나은 사양을 제공할 수 있다.

그러나 프로젝트 의존성이 다양해지고 필수 의존성이 추가됨에 따라 미러링 기법을 적용하기가 어려워진다. 다른 라이브러리의 타입 선언의 대부분을 추출해야 한다면, 차라리 명시적으로 @types 의존성을 추가하는 게 낫다.

## 요약
- 필수가 아닌 의존성을 분리할 때는 구조적 타이핑을 사용하면 된다.
- 공개한 라이브러리를 사용하는 자바스크립트 사용자가 @types 의존성을 가지지 않게 해야 한다. 그리고 웹 개발자가 NodeJS 관련 의존성을 가지지 않게 해야 한다.
