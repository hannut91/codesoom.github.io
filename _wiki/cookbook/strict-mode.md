---
layout  : wiki
title   : React.StrictMode 때문에 정상작동하는 줄 알았는데 아니었다 
date    : 2022-11-08 18:54:00 +0900
updated : 2022-11-08 18:54:00 +0900
author  : 안예린
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제 

useEffect로 API를 호출하는 로직을 작성한 후, 로컬에서 정상적으로 기능이 동작하는 것을 확인하고 배포했으나 실제 프로덕션 환경에서는 API를 정상적으로 호출하지 못하는 것을 확인했습니다.

## 원인

원인은 바로 코드 내의 useEffect 내의 코드 때문이었습니다. 호출 성공 여부를 쉽게 확인하기 위해 실제 api 호출 함수 대신 콘솔 로그로 보여드리겠습니다.

```js
  useEffect(() => {
    return () => {
      console.log('api 호출 함수');
    };
  }, []);
```

API 호출을 단순히 useEffect 내에 한 게 아니라 clean-up 내에 선언한 것을 확인할 수 있죠. 리턴 문 안에 함수를 호출했으니 해당 함수는 호출되지 않으리라는 것을 예상할 수 있습니다.

<img width="664" alt="image" src="https://user-images.githubusercontent.com/73337811/201307834-a1da8add-749d-4e61-ba1f-baacc39b486a.png">

하지만 해당 콘솔은 개발모드에서는 한 번 찍혔습니다. 실제 배포 환경에서는 콘솔이 찍히지 않았는데 말이죠. 왜 개발 모드에서는 콘솔이 한 번 찍히고 프로덕션 모드에서는 찍히지 않는 것일까요?

바로 Strict Mode 때문이었습니다. 정확히는, React.StrictMode 때문이었습니다.
StrictMode는 애플리케이션 내의 잠재적인 문제를 알아내기 위한 도구입니다. 

```js
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <ComponentOne />
        <ComponentTwo />
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

위처럼 흔히 <App/>같은 상위 컴포넌트에서 볼 수 있습니다. React.StrictMode 컴포넌트로 감싸진 컴포넌트들의 하위 컴포넌트들은 모두 Strict 모드 검사가 이루어지게 됩니다.

아무튼, 왜 정상 작동 되어선 안 되는 함수 호출이 React StrictMode 때문에 되었던 걸까요?
렌더링 단계 생명주기 메서드들은 여러 번 호출 될 수 있기 때문에 메모리 누수같은 부작용을 가져올 수 있습니다. React.StrictMode는 예상치 못한 부작용을 찾기 위해서 몇몇 함수들을 의도적으로 이중 호출합니다.

- 클래스 컴포넌트의 constructor, render 그리고 shouldComponentUpdate 메서드
- 클래스 컴포넌트의 getDerivedStateFromProps static 메서드
- 함수 컴포넌트 바디
- State updater 함수 (setState의 첫 번째 인자)
- useState, useMemo 그리고 useReducer에 전달되는 함수

 위처럼 useEffect, useState, useMemo, useReducer같은 특정 수명 주기 함수들은 이중 호출 됩니다. 그래서 원래대로 였으면 useEffect 리턴 문에서 실행되지 않았어야할 함수가 두 번 실행되어 useEffect 안의 리턴문의 함수가 실행되는 것처럼 작동되었던 것입니다.

 `<React.StrictMode/>`를 주석 처리 하면 처음 예상대로 콘솔이 찍히지 않습니다.

## 해결 방법

```
  useEffect(() => {

  console.log('함수 호출')
    
  }, []);
  ```
<img width="664" alt="image" src="https://user-images.githubusercontent.com/73337811/201307231-d172bf75-290c-463a-9380-733a05ed8bda.png">

React Strict Mode가 적용된 상태로 useEffect 내의 리턴문을 걷어내니 정상적으로 렌더가 두 번 되어 콘솔 또한 두 번 찍히는 걸 확인할 수 있습니다. 

## 참고

-[https://ko.reactjs.org/docs/strict-mode.html](https://ko.reactjs.org/docs/strict-mode.html)
