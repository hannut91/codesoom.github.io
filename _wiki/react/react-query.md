---
layout  : wiki
title   : React-Query를 왜 사용하나요?
date    : 2022-10-23 11:23:00 +0900
updated : 2022-10-23 11:23:00 +0900
author  : 안예린
tag     : 
toc     : true
public  : true
parent  : react
latex   : false
---

* TOC
{:toc}

React를 사용하면서 상태관리를 한다면 여러 상태관리 라이브러리를 들어보셨을 것입니다. 특히 Redux라는 상태 관리 매니저를 많이 사용하게 되는데요. Redux를 이용한다면 프로젝트의 전역 상태관리에 부족함이 없을 것입니다. 하지만 서버에서 데이터를 클라이언트에 가져오게 된다면 Redux만으로는 비동기로 데이터를 처리하기 어렵습니다. 비동기 상태처리를 하려면 부가적으로 Redux-Thunk나 Redux-saga같은 미들웨어를 사용해야만 합니다. 게다가 Redux로 전역의 상태관리와 서버 데이터 관리까지 해주게 된다면, 서버 데이터를 위한 코드가 과도해져 관리가 어려워질 것입니다. 코드숨 예약 사이트에서는 Client 데이터와 Server 데이터를 분리하기 위해 리액트 쿼리를 선택했습니다.

## React-Query란?

[공식문서](https://tanstack.com/query/v4/docs/overview)

공식문서 글을 그대로 인용하자면 `React 애플리케이션에서 서버 상태를 가져오고, 캐싱하고, 동기화하고 업데이트하는 작업을 쉽게 만든다`고 적혀있습니다. 말 그대로, 서버 상태를 관리하기 위한 라이브러리입니다.

React Query의 상태를 알고있어야합니다.

- fetching
데이터 요청 중인 상태입니다. 

- fresh
데이터가 만료되지 않은 상태입니다.

- stale
데이터가 만료된 상태입니다.

- inactive
사용하지 않는 상태입니다. 

## 리액트 쿼리의 장점

리액트 쿼리에는 무한 스크롤, 페이징 처리 최적화 등 많은 기능적 장점이 있으나 '데이터 캐싱' 장점 때문에 선택하게 되었습니다.
fetching한 데이터에 update가 있으면 자동으로 다시 fetching을 해주는 점도, 데이터가 stale해지면 자동으로 refetching한 점도 매력적이었습니다. 자연스럽게 상태 관리가 쉬워지게 됩니다. 프로젝트 구조또한 단순해진다는 의미입니다. 

선택한 이유 중 또 다른 하나는, 데이터 패칭 시에 로딩과 에러 처리를 한 곳에서 처리가 가능해진다는 장점 때문입니다.
로딩, 성공, 에러 상태를 따로 전달할 필요없이 onSuccess와 onError로 처리할수 있습니다.

## 사용하는법

### useQuery

```js
function Todos() {
  const { isLoading, isError, data, error } = useQuery(['todos'], fetchTodoList)

  if (isLoading) {
    return <span>Loading...</span>
  }

  if (isError) {
    return <span>Error: {error.message}</span>
  }

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  )
}

```

- `useQuery`는 서버에서 데이터를 가져오기 위해 모든 Promise 기반 메서드(GET 및 POST 메서드 포함)와 함께 사용할 수 있습니다.
- 대부분 의 쿼리의 경우 일반적으로 isLoading상태를 확인한 다음 상태 를 확인한 isError다음 마지막으로 데이터를 사용할 수 있다고 가정하고 성공적인 상태를 렌더링하는 것으로 충분합니다.

### useMutation
데이터를 생성/업데이트/삭제하는데 사용합니다.

```js
function App() {
  const mutation = useMutation(newTodo => {
    return axios.post('/todos', newTodo)
  })

  return (
    <div>
      {mutation.isLoading ? (
        'Adding todo...'
      ) : (
        <>
          {mutation.isError ? (
            <div>An error occurred: {mutation.error.message}</div>
          ) : null}

          {mutation.isSuccess ? <div>Todo added!</div> : null}

          <button
            onClick={() => {
              mutation.mutate({ id: new Date(), title: 'Do Laundry' })
            }}
          >
            Create Todo
          </button>
        </>
      )}
    </div>
  )
}
```

```js
useMutation(addTodo, {
  onSuccess: (data, variables, context) => {

  },
  onError: (error, variables, context) => {

  },
})
```

성공 상태와 에러 상태를 손쉽게 설정할 수 있습니다.

## 참고

- [TanStack Query 공식문서](https://tanstack.com/query/v4/docs/overview)