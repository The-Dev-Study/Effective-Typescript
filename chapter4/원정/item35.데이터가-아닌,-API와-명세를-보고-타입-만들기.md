# 아이템 35 데이터가 아닌, API와 명세를 보고 타입 만들기
잘 설계된 타입은 타입스크립트 사용을 즐겁게 해 주는 반면, 잘못 설계된 타입은 비극을 불러온다. 이런 상황에서 타입을 직접 작성하지 않고 자동으로 생성할 수 있다면 매우 유용할 것이다.

파일 형식, API, 명세등 우리가 다루는 타입 중 최소한 몇개는 프로젝트 외부에서 비롯된 것이다. 명세를 참고해 타입을 생성하면 타입스크리트는 사용자가 실수를 줄일 수 있게 도와준다.

아이템 31에서 Feature의 경계 상자를 계산하는 calculateBoundingBox 함수를 사용했다.

``` tsx
function calculateBoundingBox(f: Feature): BoundingBox | null {
    let box: BoundingBox | null = null;

    const helper = (coords: any[]) => {
        // ...
    };

    const { geometry } f;
    if (geometry) {
        helper(geometry.coordinates);
    }

    return box;
}
```

Feature 타입은 명시적으로 정의된 적이 없다. 아이템 31에 등장한 focusOnFeature 함수 예제를 사용하여 작성해볼 수 있지만, 공식 GeoJSON 명세를 사용하는 것이 더 낫다.
다행이도 DefinitelyTyped에는 이미 타입스크립트 타입 선언이 존재한다. 따라서 다음과 같이 익숙한 방법을 사용해 타입을 추가할 수 있다.

``` bash
npm install --save-dev @types/geojson
```


``` tsx
import { Feature } from 'geojson';

function calculateBoundingBox(f: Feature): BoundingBox | null {
    let box: BoundingBox | null = null;

    const helper = (coords: any[]) => {
        // ...
    };

    const { geometry } f;
    if (geometry) {
        helper(geometry.coordinates);
    }

    return box;
}
```
> Geometry 형식에 'coordinates' 속성이 없습니다.
> <br> GeometryCollection 형식에 'coordinates' 속성이 없습니다.

geometry에 coordinates 속성이 있다고 가정한게 문제이다. 이러한 관계는 점, 선, 다각형을 포함한 많은 도형에서는 맞는 개념이지만, GeoJSON은 다양한 도형의 모음인 GeometryCollection일 수도 있다. 다른 도형 타입들과는 다르게 GeometryCollection에는 coordinates 속성이 없다.

geometry가 GeometryCollection 타입인 Feature를 사용해서 calculateBoundingBox를 호출하면 undefined의 0 속성을 읽을 수 없다는 오류가 발생한다.

이 오류를 고치는 방법은 다음 코드처럼 GeometryCollection을 명시적으로 차단하는 것이다.

``` tsx
const {geometry} = f;
if (geometry) {
    if (geometry.type === 'GeometryCollection') {
        throw new Error('GeometryCollection은 지원되지 않습니다.');
    }
    helper(geometry.coordinate); // 정상
}
```

타입스크립트에서는 타입을 체크하는 방법으로 도형의 타입을 정제할 수 있으므로 정제된 타입에 한해서 geometry.coordinates의 참조를 허용하게 된다. 

그러나 GeometryCollection 타입을 차단하기 보다는 모든 타입을 지원하는 것이 더 좋은 방법이기 때문에 조건을 분기해서 헬퍼 함수를 호출하면 모든 타입을 지원할 수 있다.

``` tsx
const geometryHelper = (g: Geometry) => {
    if (geometry.type === 'GeometryCollection') {
        geometry.geometries.forEach(geometryHelper);
    } else {
        helper(geometry.coordinates); // 정상
    }
}

const {geometry} = f;
if (geometry) {
    geometryHelper(geometry);
}
```

그동안 GeoJSON을 사용해온 경험을 바탕으로 GeoJSON의 타입을 직접 작성했을 수 있다. 
하지만 직접 작성한 선언에는 GeometryCollection 같은 예외 상황이 포함되지 않았을 테고 완벽할 수도 없다. 반면 명세를 기반으로 타입을 작성한다면 현재까지 경험한 데이터뿐만 아니라 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있다.

API 호출에도 비슷한 고려 사항들이 적용된다. API의 명세로부터 타입을 생성할 수 있따면 그렇게 하는 것이 좋다. 특히 GraphQL처럼 자체적으로 타입이 정의된 API에서 잘 동작한다.

GraphQL의 장점은 특정 쿼리에 대해 타입스크립트 타입을 생성할 수 있다는 것이다. GeoJSON 예제와 마찬가지로 GraphQL을 사용한 방법도 타입에 null이 가능한지 여부를 정확하게 모델링할 수 있다.
 
다음 예제는 GitHub 저장소에서 오픈 소스 라이선스를 조회하는 쿼리이다.
``` tsx
query getLicense($owner:String!, $name:String!) {
    repository(owner:$owner, name:$name) {
        description
        licenseInfo {
            spdxId
            name
        }
    }
}
```

$owner와 $name은 타입이 정외된 GraphQL 변수이다. 타입 문법이 타입스크립트와 매우 비슷하다. 타입스크립트에서는 string이 된다. 그리고 타입스크립트에서 string 타입은 null이 불가능하지만 GraphQL의 String 타입에서는 null이 가능하다. 타입 뒤에 !는 null이 아님을 명시한다.
GraphQL 쿼리를 타입스크립트 타입으로 변환 해주는 많은 도구가 존재한다. 

자동 생성된 타입 정보는 API를 정확히 사용할 수 있도록 도와준다. 쿼리가 바뀐다면 타입도 자동으로 바뀌며 스키마가 바뀐다면 타입도 자동으로 바뀐다. 타입은 단 하나의 원천 정보인 GraphQL 스카마로부터 생성되기 때문에 타입과 실제 값이 항상 일치한다.

우리는 이미 자동 타입 생성의 이점을 누리고 있다. 브라우저 DOM API에 대한 타입 선언은 공식 인터페이스로부터 생성되었다. 이를 통해 복잡한 시스템을 정확히 모델링하고 타입스크립트가 오류나 코드상의 의도치 않은 실수를 잡을 수 있게 한다.

## 요약
- 코드 구석 구석까지 타입 안정성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.