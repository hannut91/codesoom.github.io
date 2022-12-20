---
layout  : wiki
title   : Suspense는 무엇인가요?
date    : 2022-12-15 11:23:00 +0900
updated : 2022-12-15 11:23:00 +0900
author  : 우종혁
tag     : 
toc     : true
public  : true
parent  : react
latex   : false
---

* TOC
{:toc}

## Suspense란?
Suspense를 사용하면 컴포넌트가 렌더링 하기 전에 다른 작업이 먼저 이루어지도록 대기합니다.
현재 Suspense는 단 하나의 사용 사례 [React.lazy](#reactlazy)를 사용하여 컴포넌트를 동적으로 불러오기만 지원합니다.
```jsx
// 이 컴포넌트는 동적으로 불러옵니다
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // Displays <Spinner> until OtherComponent loads
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```
`children` : 렌더링 하려는 실제 UI입니다. 자식이 렌더링 하는 동안 일시 중지되면 Suspense의 `fallback`이 렌더링 됩니다.

`fallback` : 로드가 완료되지 않은 경우 실제 UI 대신 렌더링 할 대체 UI입니다. 모든 유효한 React 노드가 허용됩니다. Suspense는 자식이 렌더링 하는 동안 일시 중지되면 자동으로 fallback으로 전환되고 데이터가 준비되면 다시 자식으로 돌아갑니다. 렌더링 중에 fallback이 일시 중단되면 가장 가까운 상위 Suspense 경계가 활성화됩니다.

`lazy`한 컴포넌트는 `Suspense` 트리 내의 깊숙한 곳에 위치할 수 있다는 것을 유의해야 합니다. 즉, `Suspense`가 모든 컴포넌트를 감쌀 필요는 없습니다.
가장 좋은 사용법은 로딩 처리를 보여주고 싶은 지점에 `Suspense`를 작성하는 것이지만, [Code Spliting](#codespliting)을 하고자 하는 지점 어디서든지 `lazy()`를 써야합니다.
### React.lazy
`React.lazy()`를 사용하면 동적으로 불러오는 컴포넌트를 정의할 수 있습니다.
그러면 번들의 크기를 줄이고, 초기 렌더링에서 사용되지 않는 컴포넌트를 불러오는 작업을 지연시킬 수 있습니다.
`lazy`한 컴포넌트를 렌더링 하려면 렌더링 트리 상위에 `<React.Suspense>` 컴포넌트가 존재해야 합니다. 이를 활용하여 로딩에 대한 처리를 나타낼 수 있습니다.

## Code Spliting
코드 분할은 앱을 지연 로딩하게 도와주고 앱 사용자에게 획기적인 성능 향상을 하게 합니다. 앱의 코드 양을 줄이지 않고도 사용자가 필요하지 않은 코드를 불러오지 않게 하며 앱의 초기화 로딩에 필요한 비용을 줄여줍니다.
`React.lazy`함수를 사용하면 동적 import를 사용해서 컴포넌트를 렌더링 할 수 있습니다.
Before
```jsx
import OhterComponent from './OtehrComponent';
```
After
```jsx
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```
`MyComponent`가 처음 렌더링 될 때 `OtherComponent`를 포함한 번들을 자동으로 불러옵니다.

lazy 컴포넌트는 `Suspense` 컴포넌트 하위에서 렌더링 되어야 하며, `Suspense`는 lazy 컴포넌트가 로드되길 기다리는 동안 로딩 화면과 같은 예비 컨텐츠를 보여줄 수 있게 해줍니다.
```jsx
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
`fallback` prop은 컴포넌트가 로드될 때까지 기다리는 동안 렌더링 하려는 React 엘리먼트를 보여줍니다. `Suspense` 컴포넌트는 lazy 컴포넌트를 감쌉니다.
하나의 `Suspense`컴포넌트로 여러 lazy 컴포넌트를 감쌀 수도 있습니다.

만약 네트워크 장애 같은 이유로 다른 모듈을 로드에 실패할 경우 어떻게 대응할 수 있을까요?
*Error boundaries* 를 이용하면 됩니다.
Error Boundary를 만들고 lazy 컴포넌트를 감싸면 네트워크 장애가 발생했을 때 에러를 표시할 수 있습니다.


## 참고
- [React Suspense공식문서](https://ko.reactjs.org/docs/react-api.html#suspense)
- [React.lazy 공식문서](https://ko.reactjs.org/docs/react-api.html#reactlazy)
- [React(Beta) 공식문서](https://beta.reactjs.org/apis/react/Suspense)