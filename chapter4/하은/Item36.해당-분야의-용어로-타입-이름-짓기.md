# 아이템 36. 해당 분야의 용어로 타입 이름 짓기

> -   엄선된 변수명은 의도를 명확히 하고, 코드와 타입의 추상화 수준을 높여준다.
> -   가독성과 추상화 수준을 높이기 위해서는 해당 분야의 용어를 사용해야 한다.

## 네이밍에 문제가 있는 코드

```ts
interface Animal {
    name: string; // 동물의 학명인지 일반 명칭인지 알 수 없음 ❗
    endangered: boolean; // 만약 이미 멸종한 동물이라면 어떻게 표현할지 애매함❗
    habitat: string; // string 타입이어서 범위가 너무 넓고, 필드명이 모호함 ❗
}

const leopard: Animal = {
    name: 'Snow Leopard', // 동물 이름: 스노우 레오파드인데 변수명과 다름❗
    endangered: false,
    habitat: 'tundra',
};
```

-   위 코드는 필드명이 모호하다는 문제를 갖고 있다.

## 개선된 코드

```ts
// 동물 정보를 정의하는 인터페이스
interface Animal {
    commonName: string; // 동물의 일반 이름
    genus: string; // 속(genus)명
    species: string; // 종(species)명
    status: ConservationStatus; // 보전 상태
    climates: KoppenClimate[]; // 서식하는 쾨펜 기후대 목록
}

// 국제 자연 보호 연맹(IUCN) 보전 상태 코드
type ConservationStatus =
    | 'EX' // Extinct (절멸)
    | 'EW' // Extinct in the Wild (야생 절멸)
    | 'CR' // Critically Endangered (위급)
    | 'EN' // Endangered (위기)
    | 'VU' // Vulnerable (취약)
    | 'NT' // Near Threatened (준위협)
    | 'LC'; // Least Concern (관심 필요 없음)

// 쾨펜 기후 구분법(세계 기후대 코드)
type KoppenClimate =
    // 열대(A) 기후
    | 'Af'
    | 'Am'
    | 'As'
    | 'Aw'
    // 건조(B) 기후
    | 'BSh'
    | 'BSk'
    | 'BWh'
    | 'BWk'
    // 온대(C) 기후
    | 'Cfa'
    | 'Cfb'
    | 'Cfc'
    | 'Csa'
    | 'Csb'
    | 'Csc'
    | 'Cwa'
    | 'Cwb'
    | 'Cwc'
    // 한대(D) 기후
    | 'Dfa'
    | 'Dfb'
    | 'Dfc'
    | 'Dfd'
    | 'Dsa'
    | 'Dsb'
    | 'Dsc'
    | 'Dwa'
    | 'Dwb'
    | 'Dwc'
    | 'Dwd';
// 각 기호에 대한 자세한 설명은 쾨펜 기후 구분법 용어 참고
```

-   데이터를 훨씬 명확하게 표현하게 됨
-   정보를 찾기 위해 사람에 의존할 필요가 사라짐

## 타입 이름 짓는 규칙

-   코드로 표현하고자 하는 모든 분야에 전문 용어가 존재하기 때문에, 해당 용어를 사용해야 한다.
-   동일한 의미를 나타낼 떄는 **같은 용어**를 사용한다. 동의어를 사용하지 않는다.
-   `data`, `info`, `thing`, `item`, `object`, `entity`와 같이 모호하고 의미 없는 이름은 피해야 한다.
-   이름을 지을 때는 포함된 내용이나 계산 방식이 아닌 데이터 자체가 무엇인지를 고려해야 한다.
