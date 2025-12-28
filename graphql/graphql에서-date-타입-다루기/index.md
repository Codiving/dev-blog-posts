---
title: "GraphQL에서 Date 타입 다루기"
date: "2025-04-15"
keywords:
  [
    "GraphQL",
    "GraphQL Date",
    "GraphQL Date type",
    "GraphQL middleware",
    "GraphQL Date string",
    "GraphQL Date 다루기",
  ]
description: GraphQL에서 Date 타입 다루는 방법.
summary: GraphQL에서는 Date 타입을 기본으로 제공하지 않기 때문에, 서버에서 Date 값을 받아도 실제로는 string으로 처리되는 문제가 발생합니다. 특히 TypeScript 환경에서는 이로 인해 타입 혼란과 런타임 오류가 발생할 수 있습니다. 이 문제를 깔끔하게 해결하는 방법을 소개합니다.
thumbnail: ./thumb.png
---

# GraphQL에서 Date 타입 다루기

`GraphQL`에서는 `Date` 타입을 기본으로 제공하지 않기 때문에, 서버에서 `Date` 값을 받아도 실제로는 `string`으로 처리되는 문제가 발생합니다. 특히 `TypeScript` 환경에서는 이로 인해 타입 혼란과 런타임 오류가 발생할 수 있습니다. 이 문제를 깔끔하게 해결하는 방법을 소개합니다.

## GraphQL에서 `Date` 타입 문제

```graphql title="GraphQL"
scalar Date
```

`GraphQL`에는 `Date`가 내장 타입이 아니므로, **커스텀 스칼라**로 정의해야 합니다. 이 때문에 클라이언트와 서버 간에 날짜 값이 `string` 형태로 전달되고, 타입스크립트에서는 이를 `Date` 타입으로 간주해 혼란이 생깁니다.

```ts title="TS"
const date = useQuery(GET_USER_BIRTH_DATE);
console.log(typeof date);
```

타입스크립트 상으로는 `Date`처럼 보이지만 실제로는 `string`이며, 그대로 `Date` 메서드를 사용하면 오류가 발생합니다.

```ts title="TS"
const date = useQuery(GET_USER_BIRTH_DATE);
console.log(new Date(date));
```

매번 `new Date()`로 변환하여 사용하면 되는데 이는 번거롭고 비효율적입니다.

## GraphQL에서 `Date` 타입 변환 미들웨어 구현

서버로부터 응답받은 데이터를 자동으로 `Date` 객체로 변환하는 미들웨어를 구현하면 문제를 해결할 수 있습니다. 이렇게 하면 클라이언트에서는 별도 처리 없이 `Date` 메서드를 사용할 수 있습니다.

### 변환이 필요한 쿼리와 필드 선언

먼저, 어떤 쿼리의 어떤 필드를 `Date`로 변환할지 정의합니다.

```ts title="TS"
const NEEDED_DATE_CONVERT = {
  GetUserBirthDate: ["getUserBirthDate.birthDate"],
  GetComment: ["getComment.info.createdAt", "getComment.info.updatedAt"],
  PaginateUser: ["paginateUser.docs.${index}.birthDate"],
};

const QUERY_LIST = Object.keys(NEEDED_DATE_CONVERT);
```

배열 형태의 응답(docs)은 `${index}`를 사용해 모든 인덱스를 처리할 수 있도록 패턴을 정의합니다.

### 변환 미들웨어 구현

데이터를 **평탄화(flatten)** 한 후, 지정한 필드를 `Date`로 변환하고 다시 원래 구조로 **복원(unflatten)** 합니다. 해당 작업에서는 `flat` 라이브러리를 사용하기 때문에 설치가 필요합니다.

<br />

미들웨어 코드는 간단합니다.

```ts title="TS"
import { flatten, unflatten } from "flat";

const convertDateMiddleware = new ApolloLink((operation, forward) => {
  if (DATE_OPERATION_LIST.includes(operation.operationName)) {
    return forward(operation).map((response) => {
      if (response.data) {
        const flattenData: Record<string, any> = flatten(response.data);
        const values = DATE_CONVERT_QUERY[operation.operationName];

        values.forEach((pattern) => {
          // 패턴에 ${index}가 포함되어 있으면 처리
          if (pattern.includes("${index}")) {
            // 정규 표현식으로 index 부분이 숫자인 경우 찾기
            const regex = new RegExp(pattern.replace("${index}", "(\\d+)"));

            Object.keys(flattenData).forEach((flatKey) => {
              if (regex.test(flatKey) && flattenData[flatKey]) {
                flattenData[flatKey] = new Date(flattenData[flatKey]);
              }
            });
          } else {
            // 일반적인 키 변환
            Object.keys(flattenData).forEach((flatKey) => {
              if (flatKey === pattern && flattenData[flatKey]) {
                flattenData[flatKey] = new Date(flattenData[flatKey]);
              }
            });
          }
        });

        const unflattenData = unflatten(flattenData);

        if (unflattenData) {
          response.data = unflattenData;
        }
      }
      return response;
    });
  }

  return forward(operation);
});
```

`${index}`를 **정규 표현식**으로 처리해 배열 내 항목까지 자동 변환하도록 구현했습니다.

## 결론

이 미들웨어를 사용하면, 서버에서 `string`으로 전달된 날짜 값을 클라이언트에서 자동으로 `Date` 객체로 변환해 사용할 수 있습니다. 이제 `useQuery`나 `useLazyQuery`를 사용할 때 별도로 날짜를 파싱하지 않아도 되므로, 코드가 훨씬 깔끔하고 안전해집니다.

<br />더 나은 방법이 있다면 의견을 남겨주세요.
