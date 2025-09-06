# 아이템 31. 타입 주변에 null값 배치하기

```ts
function extent(nums: number[]) {
    let result: [number, number] | null = null; //초기값을 null로 설정

    for (const num of nums) {
        if (!result) {
            // 첫 번째 숫자인 경우
            result = [num, num];
        } else {
            result = [Math.min(num, result[0]), Math.max(num, result[1])];
        }
    }

    return result;
}
```

-   `min`과 `max`를 둘 다 반환해야 한다면, 하나의 객체로 묶어서 처리하는게 좋다.
-   `max`와 `min` 각각에 대해서 체크하는 방식은 좋지 않음!
-   `0`이 저장될 수 있는 `Number`변수의 경우 `if`문을 쓸 때 특히 조심해야겠다는 생각을 했다.

## Null과 Null이 아닌 값을 혼용하는 경우 - 클래스

```ts
class UserPosts {
    user: UserInfo | null;
    posts: Post[] | null;
}
```

-   이렇게 타입을 설계하면 `user`와 `posts` 모두 타입 좁히기를 통해 `null`인지 아닌지를 확인해야 함
-   총 4가지 케이스가 존재함
    -   `user=null, post=null`
    -   `user=UserInfo, post=Post[]`
    -   `user=null, post=Post[]`
    -   `user=UserInfo, post=null`
-   속성값의 불확실성이 모든 메서드에 악영향을 끼침 -> null 체크 로직이 많아짐

```ts
class UserPosts {
    user: UserInfo;
    posts: Post[];
}
```

-   실제 필요한 데이터가 모두 준비된 후에 생성자를 통해 객체를 생성하도록 수정한다.
