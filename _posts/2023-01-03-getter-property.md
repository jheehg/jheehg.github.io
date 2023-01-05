---
layout: post
title: getter property가 있는 객체 복사하기
description: getter property가 있는 객체 복사하기 관련 글입니다.
summary: getter property가 있는 객체 복사하기 관련 글입니다.
comments: true
tags: [javascript]
---

<br>

#### 문제 상황:

1.  React 프로젝트에서 object 타입의 상태값의 변경이 필요.
2.  원본 object 의 `getter` property 내에서 `this` 와 다른 property 를 참조하고 있다.
3.  원본 object 를 카피해서 `getter` 를 제외하고 그 외에 property 만 업데이트 하려고 시도했다.
4.  복사한 object 의 `getter` property 는 여전히 원본의 `this` 를 가리키고 있어 값이 업데이트 되지 않는 오류 발생.

<br>

👉 문제 상황 코드

```tsx
const [user, setUser] = useState(() => ({
  userName: name,
  userAge: age,
  get userInfo() {
    if (!this.userName || !this.userAge) {
      return '';
    }
    return `Name: ${this.userName}, Age: ${this.userAge}`;
  },
}));
```

<br>

아래와 같은 spread 방식으로 복사할 경우 getter 까지 복사가 될까?<br>
결과는 원본 object의 값을 그대로 쓰고 있었다.<br>
userName, userAge 값은 업데이트 되지만 getter property 인 userInfo 가 리턴하는 값은 변경되지 않는다.

```ts
useEffect(() => {
  setUser({
    ...user,
    userName: name,
    userAge: age,
  });
}, [name, age]);
```

<br>

그렇다면 getter property 는 어떻게 복사할까?<br>
구글링으로 열심히 찾아본 결과로는 `Object.getOwnPropertyDescriptor` 메서드를 사용하라는 솔루션을 받았다.

<br>

#### 1. Object.getOwnPropertyDescriptor 사용해서 복사해보기

```ts
useEffect(() => {
  const updateUser = { ...user, userName: name, userAge: age };

  const descriptor = Object.getOwnPropertyDescriptor(user, 'userInfo');
  Object.defineProperty(
    updateUser,
    'userInfo',
    descriptor as PropertyDescriptor
  );

  setUser(updateUser);
}, [name, age]);
```

<br>
다른 방식으로도 해결이 가능할까 시도해보던 중 `Object.getOwnPropertyDescriptor` 메서드를 사용하지 않고
업데이트를 하는 방법을 찾을 수 있었다.

<br>

#### 2. 원본과 공유하지 않고 재할당해서 사용하기

<br>
복사하려는 object 의 내용은 변경될 일이 없기 때문에 useMemo 로 생성해준다.<br>
해당 object 를 새 변수 `updateUser` 에 할당하면 this 바인딩이 `updateUser` 로 되기 때문에 그 이후로 property 값만 바꿔주면 된다.

```ts
const bsUser = useMemo(
  () => ({
    userName: name,
    userAge: age,
    get userInfo() {
      if (!this.userName || !this.userAge) {
        return '';
      }
      return `Name: ${this.userName}, Age: ${this.userAge}`;
    },
  }),
  []
);

const [user, setUser] = useState(() => bsUser);

// code

useEffect(() => {
  const updateUser = bsUser;
  updateUser.userName = name;
  updateUser.userAge = age;

  setUser(updateUser);
}, [name, age]);
```

<br>

아래 참고한 링크 내용에 객체의 모든 property 의 얕은 복사, 깊은 복사에 대한 설명이 잘 나와있다.
<br>

<https://zellwk.com/blog/copy-properties-of-one-object-to-another-object/>
