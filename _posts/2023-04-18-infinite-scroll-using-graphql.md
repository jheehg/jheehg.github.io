---
layout: post
title: 간단한 무한 스크롤 만들어보기 with graphQL
description: 간단한 무한 스크롤 만들어보기 with graphQL
summary: 간단한 무한 스크롤 만들어보기 with graphQL
comments: true
categories: [FE]
tags: [react, graphql, apollo]
---

### 간단한 설명

- 스크롤이 가장 밑에 닿으면 fetch 가 새로운 파라미터와 함께 재호출된다.
- 더이상 결과가 존재하지 않으면 no more data 텍스트가 보여지고 fetch 는 진행하지 않는다.

<br/>

### 사용한 소스

```
:: API (thecatapi)
https://api.thecatapi.com/v1/images/search

:: 요청 파라미터
`limit`: 한번에 불러오는 이미지 수 (1 ~ 100)
`page`: 요청할 페이지
```

<br><br>

## Server

#### typeDefs

```js
const typeDefs = `#graphql
type Image {
  id: ID!
  height: Int!
  width: Int!
  url: String!
}
type Query {
  images(limit: Int, page: Int): [Image!]!
}
`;
```

<br>

#### resolvers

- 첫번째 파라미터는 리졸버 체인 상에서 부모 필드를 의미하는데 여기선 사용하지 않음.
- 두번째 파라미터는 Query 를 실행할때 받은 arguments 이다.

```js
import fetch from ‘node-fetch’;

const resolvers = {
  Query: {
    async images(_, { limit, page }) {
      const r = await fetch(
        `https://api.thecatapi.com/v1/images/search?limit=${limit}&page=${page}`
      );
      const json = await r.json();
      return json;
    },
  },
};
```

<br>

#### 서버 인스턴스 생성 및 실행

```js
import { ApolloServer } from ‘@apollo/server’;
import { startStandaloneServer } from ‘@apollo/server/standalone’;
const server = new ApolloServer({
  typeDefs,
  resolvers,
});
const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
});
console.log(`🚀  Server ready at ${url}`);
```

<br><br>

## Client

#### Query

Apollo Server Explorer 에서도 쉽게 테스트가 가능해서 Query syntax 오류날 경우 참고하기 좋은 것 같다.

```js
import { gql, useQuery } from ‘@apollo/client’;
const GET_IMAGES = gql`
  query getImages($limit: Int, $page: Int) {
    images(limit: $limit, page: $page) {
      id
      height
      width
      url
    }
  }
`;
```

![apollo-explorer](https://user-images.githubusercontent.com/56527636/232785357-346372ad-ab81-43b4-b6e8-c037be15dbdf.png)

#### UseQuery 호출

위에서 `gql` 로 생성한 쿼리를 첫번째 인자로 넣고 두번째 인자로 input data 인 `limit`, `page` 를 넣어준다.

```js
  import { gql, useQuery } from ‘@apollo/client’;
  const { loading, error, data, fetchMore } = useQuery(GET_IMAGES, {
    variables: {
      limit: 10,
      page: 1,
    },
  });
```

<br>

#### fetchMore 실행

scroll 이벤트의 콜백함수로 `loadMoreImages` 를 사용하는데 이 안에서 `currentPage` 값을 참조하기 때문에 `useEffect` 디펜던시에 넣어줘야 업데이트된 `currentPage` 를 참조할 수 있다.<br>
더이상 가져올 이미지가 없으면 `isLoadingMore` 상태를 **false** 로 변경하고
그렇지 않을 경우 `currentPage` 를 **1** 증가시킨다.

```js
const [isLoadingMore, setIsLoadingMore] = useState(true);
const [currentPage, setCurrentPage] = useState(1);

useEffect(() => {
  window.addEventListener(‘scroll’, loadMoreImages);
  return () => window.removeEventListener(‘scroll’, loadMoreImages);
}, [currentPage]);

const loadMoreImages = () => {
   if (isAtTheBottom()) { // 스크롤 위치 확인
     const nextPage = currentPage + 1;
      fetchMore({
      variables: {
        limit: 10,
        page: nextPage,
      },
   }).then(({ data }) => {
      if (data?.images?.length === 0) {
        setIsLoadingMore(false);
      } else {
        setCurrentPage(nextPage);
      }
   });
 }
};
```

<br>

#### cache 설정

read 함수가 실행이 계속 안되서 고생했는데 keyArgs 필드에 `false` 값을 줘야 한다.

```js
// utils/client.ts
import { ApolloClient, InMemoryCache } from ‘@apollo/client’;
const cache = new InMemoryCache({
  typePolicies: {
    Query: {
      fields: {
        images: {
          keyArgs: false,
          read(existing) {
            return existing && existing.slice(0);
          },
          merge(existing, incoming) {
            // Slicing is necessary because the existing data is
            // immutable, and frozen in development.
            const merged = existing ? existing.slice(0) : [];
            return [...merged, ...incoming];
          },
        },
      },
    },
  },
});

const client = new ApolloClient({
  uri: ‘http://localhost:4000’,
  cache,
});
export default client;
```

<br>

### 💡KeyArgs

If a field accepts arguments, you can specify an array of keyArgs in the field's FieldPolicy. This array indicates which arguments are key arguments that affect the field's return value. **Specifying this array can help reduce the amount of duplicate data in your cache.**
<br>

👉 Key Argument에 명시된 필드에 대해서는 중복을 제외시키는 것 같다. merge 된 데이터를 조회해야 하므로 limit, page 를 필드로 넣으면 정상적으로 캐시 데이터가 조회 되지 않는다.

<br><br>

\+ 캐시 사용 안하고 Query 를 업데이트 하는 방법

```js
 updateQuery: (previousResult, { fetchMoreResult }) => {
   if (!fetchMoreResult || fetchMoreResult.images.length === 0) {
     setIsLoadingMore(false);
     return previousResult;
   }
   setCurrentPage(nextPage);
   return Object.assign({}, previousResult, {
     images: [
       ...previousResult.images,
       ...fetchMoreResult.images,
     ],
   });
 },
```

<br><br>
결과는 요거

![Apr-18-2023 22-21-22](https://user-images.githubusercontent.com/56527636/232790566-bba55a35-4312-44a0-a79d-8352a1c2484d.gif){: width="470"}

<br><br><br>
참고자료는<br>
[Apollo GraphQL](https://www.apollographql.com/docs/apollo-server/){:target="\_blank"} 와 챗지피티! 🥲
