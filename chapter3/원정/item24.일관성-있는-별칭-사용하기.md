# 아이템 24 일관성 있는 별칭 사용하기

``` typescript
const borough = {name: 'Brooklyn', location: [40.688, -73.979]};
const loc = borough.location;
```

borough.location 배열에 loc이라는 별칭을 만들었다. 별칭의 값을 변경하면 원래 속성값에서도 변경된다.

``` bash
> loc[0] = 0;
> borough.location
[0, -73.979]
```

## 요약
- 별칭은 타입스크립트가 타입을 좁히는 것을 방해한다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
- 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋다.
- 함수 호출이 객체 속성의 타입 정제를 무효화할 수 있다는 점을 주의해야 한다.