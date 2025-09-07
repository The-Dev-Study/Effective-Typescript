# 아이템 36 해당 분야의 용어로 타입 이름 짓기

이름 짓기 역시 타입에서 중요한 부분이다. 엄성된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입 추상화 수준을 높여 준다.

동물들의 데이터베이스를 구축한다고 가정해보자. 이를 표현하기 위한 인터페이스는 다음과 같다
``` tsx
interface Animal {
    name: string;
    endangered: boolean;
    habitat: string;
}

const leopard: Animal = {
    name: 'Snow Leopard',
    endangered: false,
    habitat: 'tundra'
}
```

이 코드에는 네 가지 문제가 있다.
- name은 매우 일반적인 이름이다. 동물의 학명인지 일반적인 명칭인지 알 수 없다.
- endangered 속성이 멸종 위기 동물을 표현하기 위해 boolean 타입을 사용한 것이 이상하다. 이미 멸종된 동물을 true로 해야 하는지 알 수 없다. endangered 속성의 의도를 '멸종 위기 또는 멸종'으로 생각했을 수도 있다.
- 서식지를 나타내는 habitat는 너무 범위가 넓은 string 타입 뿐만 아니라 서식지라는 뜻 자체도 불분명하기 때문에 다른 속성들보다 훨씬 모호하다.
- 객체의 변수명이 leopard지만 name의 속성 값은 snow leopard다 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명하다.

이 예제의 문제를 해결하려면, 속성에 대한 정보가 모호하기 때문에 해당 속성을 작성한 사람을 찾아서 의도를 물어봐야 한다.

반면, 다음 코드의 타입 선언은 의미가 분명하다
``` tsx
interface Animal {
    commonName: string,
    genus: string,
    species: string,
    status: ConservationStatus;
    climates: KoppenClimate[];
};

type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = | 'Af' | 'Am' | 'As' | 'Aw' | ...

const snowLeopard: Animal = {
    commonName: 'Snow Leopard',
    genus: 'Panthera',
    species: 'Uncia',
    status: 'VU',
    climates: ['ET', 'EF', 'Dfd'],
};
```

이 코드는 다음 세가지를 개선했다.
- name은 commonName, genus, species등 더 구체적인 용어로 대체했다.
- endangered는 동물 보호 등급에 대한 IUCN의 표준 분류 체계인 ConservationStatus 타입의 status로 변경되었습니다.
- habitat는 기후를 뜻하는 climates로 변경되었으며, 쾨펜 기후 분류를 사용한다.

이번 예제는 데이터를 훨씬 명확하게 표현하고 있다.

타입 속성 변수에 이름을 붙일 때 명심해야할 세가지 규칙이 있다.
- 동일한 의미를 나타낼 때는 같은 용어를 사용해야 한다. 글을 쓸 때나 말을 할 때, 같은 단어를 반복해서 사용하면 지루할 수 있기 때문에 동의어를 사용한다. 동의어를 사용하면 글을 읽을 때는
좋을 수 있지만 코드에서는 좋지 않다. 정말로 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용해야 한다.
- data, info, thing, item, object, entity 같은 모호하고 의미 없는 이름은 피해야 한다. 만약 entity라는 용어가 해당 분야에서 특별한 의미를 가진다면 괜찮지만. 귀찮다고 아무 이름이나 붙여서는 안된다.
- 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지 고려해야 한다. 예를 들어 INodeList보다는 Directory가 더 의미있는 이름이다. 좋은 이름은 추상화의 수준을 높이고 의도치 않은 충돌의 위험성을 줄여 준다.

## 요약
- 가독성을 높이고, 추상화 수준을 높이기 위해 해당 분야의 용어를 사용해야 한다.
- 같은 의미에 다른 이름을 붙이면 안된다.