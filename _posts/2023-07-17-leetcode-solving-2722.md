---
layout: post
title:  리트코드 문제풀이 (Join Two Arrays by ID)
description: 리트코드 문제풀기 (Join Two Arrays by ID)
summary: 리트코드 문제풀기 (Join Two Arrays by ID)
comments: true
categories: [JavaScript]
tags: [leetcode]
---

<br>
### 문제 내용
arr1, arr2 두 개의 배열을 받아서 하나의 joinedArray 을 리턴한다.<br>
각 배열안의 객체는 정수값을 가지는 id 필드가 반드시 포함되어 있다.<br>
joinedArray 는 기본적으로 arr1 와 arr2 의 id 필드를 이용해 머지한 배열이다. <br>
joinedArray 배열의 길이는 반드시 두 배열의 id 값이 유니크한 배열의 길이를 가져야만 한다.<br>
리턴되는 배열은 id 가 오름차순으로 정렬되어 있어야 한다.<br><br>

- 서로 겹치지 않은 id 값을 가지는 객체가 있다면 key-value 쌍에 대한 수정 없이 리턴해야 한다.
- 만약 같은 id 값을 가진다고 하면 하나의 객체로 머지해야 하는데 arr2 의 값을 arr1 값에 오버라이딩 해야 한다.

<br>

```
Example 1:

Input:
arr1 = [
    {"id": 1, "x": 1},
    {"id": 2, "x": 9}
],
arr2 = [{"id": 3, "x": 5}]

Output:
[
    {"id": 1, "x": 1},
    {"id": 2, "x": 9},
    {"id": 3, "x": 5}
]
```

```
Example 2:

Input:
arr1 = [
    {"id": 1, "x": 2, "y": 3},
    {"id": 2, "x": 3, "y": 6}
],
arr2 = [
    {"id": 2, "x": 10, "y": 20},
    {"id": 3, "x": 0, "y": 0}
]

Output:
[
    {"id": 1, "x": 2, "y": 3},
    {"id": 2, "x": 10, "y": 20},
    {"id": 3, "x": 0, "y": 0}
]
```

<br><br>

### 초기 문제 접근

1. arr1 를 탐색하면서 arr2 의 id 값과 매칭되는 값이 있는지 확인한다.
2. 만약 매칭되는 값이 있다면 두 객체를 머지한다.
3. 머지된 객체는 arr2 에서 splice 를 통해 제거한다.
4. arr1 탐색이 끝난 후 arr2 에 남은 객체가 있다면 합쳐준다.
5. 마지막으로 sort() 한 후 결과값을 리턴한다.

<br>

```js
const join = function (arr1, arr2) {
  for (let i = 0; i < arr1.length; i++) {
    const targetIndex = arr2.findIndex((obj) => arr1[i]['id'] === obj['id']);
    if (targetIndex > -1) {
      arr1[i] = { ...arr1[i], ...arr2[targetIndex] };
      arr2.splice(targetIndex, 1);
    }
  }

  if (arr2.length > 0) {
    arr1.push(...arr2);
  }

  arr1.sort((a, b) => a['id'] - b['id']);
  return arr1;
};
```

<br><br>

### 런타임 개선이 필요하다 😢

sort() 를 한 게 문제였을까 결과로 제출한 문제풀이의 런타임이 처참했다. <br>
정렬을 안하고 id 를 순차적으로 넣을 수 있는 방법이 있을까 이것저것 루프 안에서 시도해보다가 <br>
더 꼬여버리고 난 후 다른 사람 풀이를 참고하기로 했다.<br><br>

1️⃣ array 를 사용해 인덱스를 기록하기

```js
const join = function (arr1, arr2) {
  let result = [];
  for (let i = 0; i < arr1.length; i++) {
    result[arr1[i].id] = arr1[i];
  }

  for (let i = 0; i < arr2.length; i++) {
    if (result[arr2[i].id]) {
      result[arr2[i].id] = { ...result[arr2[i].id], ...arr2[i] };
    } else {
      result[arr2[i].id] = arr2[i];
    }
  }

  return Object.values(result);
};
```

배열로 저장을 해볼까 하다가 시도를 하지 않았는데 id 가 0 부터 1씩 차례로 증가하면서 들어올거라는 보장이 없어서였다. <br>
근데 `Object.values(result)` 는 어차피 `undefined` 인 값들은 버리고 값이 존재하는 것만
리턴해준다. (+ undefined 제외하고 필터링하는 방법도 있겠네 (..😵)) <br><br>
아래 코드 참고👇 <br>

```js
const arr = [, , { id: 2 }, , { id: 4 }];
Object.values(arr); // [{id: 2}, {id: 4}]
```

<br><br>

2️⃣ 2개 포인터 사용

```js
const join = function (arr1, arr2) {
  let result = [];
  let i = 0;
  let j = 0;
  while (i < arr1.length && j < arr2.length) {
    if (arr1[i]['id'] < arr2[j]['id']) {
      result.push(arr1[i++]);
    } else if (arr1[i]['id'] > arr2[j]['id']) {
      result.push(arr2[j++]);
    } else {
      result.push({ ...arr1[i++], ...arr2[j++] });
    }
  }
  if (i < arr1.length) {
    result.push(...arr1.slice(i));
  } else if (j < arr2.length) {
    result.push(...arr2.slice(j));
  }

  return result;
};
```

<br><br>

### 정리

**내 풀이** <br>
for 루프 안에서 findIndex 함수를 사용할 경우 시간 복잡도는 O(n \* m)<br><br>
**첫번째 풀이** <br>
시간 복잡도 O(n + m)<br><br>
**두번째 풀이** <br>
시간 복잡도 O(n + m) 이고, 객체를 직접 push 하여 결과값을 리턴하므로 제일 효율적인 방법이라고 한다.<br><br>

### 결론

루프 안에서 실행되는 findIndex 함수도 루프라는 사실을 간과하지 말자. <br>
오히려 sort() 하는 것보다 성능에 더 안좋은 영향을 준다.
