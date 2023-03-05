---
layout: post
title: react-query 중요한 기본 개념들 정리
description: react-query 기본 개념들에 대한 글입니다.
summary: react-query 기본 개념들에 대한 글입니다.
comments: true
categories: [FE]
tags: [react-query, react]
---

<br>

# 기본 옵션들

### 👉 StaleTime

useQuery 는 기본적으로 캐시된 데이터를 오래된 상태라고 간주한다.<br>
(오래된 상태라고 간주하면 새로운 데이터를 요청하게 되므로)<br>
이 기본적으로 적용된 옵션을 막으려면 전역적으로 또는 개별적으로 useQuery 를 호출할 때<br>
staleTime 옵션을 길게 적용하면 쿼리를 refetch 가 자주 실행되지 않도록 할 수 있다. 기본값은 0 이다.

Stale query 는 아래 환경에서 다시 refetch 가 일어나는데 각 환경마다 조건을 변경해주면 된다.

- refetchOnMount: 쿼리 마운트의 새 인스턴스가 생성될 때
- refetchOnWindowFocus: 윈도우 창이 다시 포커스될 때
- refetchOnReconnect: 네트워크가 재연결될 때
- refetchInterval: 쿼리의 refetch interval 이 다시 주어질 때

<br>

### 👉 cacheTime

쿼리 결과가 더이상 active 상태가 아닌 useQuery 인스턴스 또는 query observer 들은 inactive 로 분류되어 캐시에 남아있게 된다.
기본적으로 inactive 된 쿼리들은 5분 이후에 가바지 콜렉터 대상이 되는데 이를 변경하려면
cacheTIme 옵션을 변경한다. 기본값은 5분으로 설정되어 있음.

<strike>\* fetch 된 이후에는 staleTime 영향을 먼저 받고 주어진 stale 타임아웃이 끝날 경우 캐시된 데이터에서 찾게 된다. </strike>
<br>
✳️ **cache-time 과 stale-time 관계**

- stale-time 이 종료되어 stale 상태가 되면 서버에서 fetch 해서 새 데이터를 가져오기 전까지, 캐시된 데이터를 보여준다. 즉, cache-time 과 상관없이 **stale-time 이 종료되면 다시 fetch 를 한다.**
- cache-time 은 데이터가 stale 상태가 되기 전까지 캐시된 데이터를 가져올 수 있는 duration 을 설정한 것이다.
- 그래서 **stale-time < cache-time** 으로 설정하는 것이 좋다.
- stale-time 은 요청하는 데이터가 **얼마나 자주 변경되는지**에 따라 설정한다.

<https://www.codemzy.com/blog/react-query-cachetime-staletime>

<br>

### 👉 retry / retryDelay

쿼리의 오류가 잡혀서 화면에 보이지기 전까지 3번의 retry를 하게 되는데 이를 조정하려면 retry 옵션 또는 retryDelay 를 변경하면 된다.

<br>

### 👉 config.isDataEqual

쿼리 결과는 기본적으로 데이터가 변경된게 없는지 구조적으로 감지하고 만약 변경된게 없다면 데이터는 변경되지 않는 값을 참조하게 되고 이는 useMemo, useCallback 으로 값을 유지하고자 할 때 많은 도움이 되도록 한다.<br>이 기능은 JSON 호환타입일 때만 적용되고 나머지 타입에 대해서는 항상 변경된 값으로 간주한다.
<br>JSON 호환 타입 외에도 모든 타입에 같은 기능을 적용하고 싶다면 config.isDataEqual 을 사용하면 된다.

<br>

# caching 데이터를 사용하기 위해 알아야 할 점

### 👉 queryKeys 설정

```tsx
// string-only
useQuery('todos', ...)
// array keys
useQuery(['todo', 5], ...)
```

<br>

```tsx
// array key 는 순서가 다르면 다른 키로 간주한다.
useQuery(['todos', status, page], ...)
useQuery(['todos', page, status], ...)

// object 은 순서 상관없이 같은 키로 간주한다. 아래 두 개는 같은 키.
// query string 은 사용자의 입력 순서에 따라 달라질 수 있으므로 object 로 키 값을 받도록 한다.
useQuery(['todos', { status, page }], ...)
useQuery(['todos', { page, status, other: undefined }], ...)
```

<br>

### 👉 caching Lifecycle

- 쿼리 인스턴스 생성
- background refetching
- 쿼리 inative 상태로 변경
- Garbage Collection

<br>

아래 과정 참고:

1. useQuery('todos', fetchTodos) 의 인스턴스가 마운트되었다.<br>
   해당 쿼리키로 쿼리가 만들어진 적이 없으므로 네트워크에 데이터 fetching 요청을 하게 된다.<br>
   -- 'todos'와 fetchTodos 를 유니크한 식별자로 사용하면서 이를 캐싱하게 된다. <br>
   -- staleTime 의 기본값은 0 이므로 쿼리가 실행되자마자 stale 상태로 간주하게 된다. <br><br>

2. useQuery('todos', fetchTodos) 의 두번째 인스턴스가 마운트되었다.<br>
   -- 첫번째 인스턴스로 인해 쿼리가 캐시되어 있는 상태이므로 데이터는 캐시된 데이터를 가져오게 된다.<br><br>

3. 새 인스턴스가 화면에 나타났으므로 background refetch 는 각 쿼리에 의해 trigger 된다.<br>
   -- fetch 가 성공했다면 두 인스턴스 모두 새 데이터로 업데이트 된다.<br><br>

4. useQuery('todos', fetchTodos) 의 각 인스턴스는 언마운트되고 더이상 사용되지 않는다.<br>
   -- 각 쿼리의 인스턴스는 더이상 active 상태가 아니므로 캐시 타임아웃(5분)이 쿼리에 설정되게 된다.<br><br>

5. 캐시 타임아웃이 다 진행되기 전에 또 다른 useQuery('todos', fetchTodos) 마운트의 인스턴스가 생성되면 fetchTodos 함수가 새 데이터와 함께 쿼리가 실행되는 동안에 현재 사용가능한 캐시된 데이터를 즉시 리턴하게 된다.<br><br>

6. 가장 마지막 인스턴스가 언마운트된다.<br><br>

7. useQuery('todos', fetchTodos) 쿼리가 5분간 사용되지 않을 경우 데이터가 가비지 콜렉터에 의해 제거되게 된다.

<br><br><br>

아래 공식문서를 참고해서 정리했습니다. <br>
개인적인 학습 목적으로 번역한 것이므로 잘못된 내용이 있을 수 있습니다.<br>
<https://react-query-v3.tanstack.com/guides/important-defaults>
