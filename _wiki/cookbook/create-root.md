---
layout  : wiki
title   : 리액트 컴포넌트 테스트할 때 createRoot를 사용하라는 경고가 출력된다
date    : 2022-10-08 11:23:00 +0900
updated : 2022-10-08 11:23:00 +0900
author  : 한윤석
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제

jest로 `.jsx` 파일을 테스트할 때 테스팅 라이브러리에서 사용하는 `render` 를 사용하면 다음과 같이 `createRoot` 를 사용하라는 경고 문구가 출력됩니다.

```bash
console.error
  Warning: ReactDOM.render is no longer supported in React 18. 
  Use createRoot instead. Until you switch to the new API, your app will 
  behave as if it's running React 17. 
  Learn more: https://reactjs.org/link/switch-to-createroot
```

## 원인

프로젝트에 설치된 리액트의 버전은 18인데, `@testing-library/react` 버전을 낮은 것을 사용하고 있습니다. 낮은 버전에서 redner할 때 `createRoot` 대신에 `ReactDOM.render` 를 사용하고 있습니다.

## 해결 방법

리액트 18버전에서부터는 `ReactDOM.render` 대신에 `createRoot` 을 사용하는 것을 권장합니다. 왜냐하면 렌더할 때 사용하는 API의 사용법이 변경되었기 때문입니다. 따라서 `@testing-library/react` 의 버전을 업데이트 해야 합니다.

```bash
npm install --save-dev @testing-library/react
npm install --save-dev @testing-library/jest-dom
```

[https://github.com/testing-library/react-testing-library/pull/1031](https://github.com/testing-library/react-testing-library/pull/1031)

위의 풀 리퀘스트를 보시면 우리가 테스트 라이브러리에서 사용하는 `render` 함수가 내부에서 `createRoot` 을 사용하는 것으로 변경된 것을 확인할 수 있습니다.

그래서 버전을 업그레이드 하면 경고 문구가 사라집니다.

## 참고

- [Updates to Client Rendering APIs](https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html#updates-to-client-rendering-apis)
- [Replacing render with createRoot](https://github.com/reactwg/react-18/discussions/5)
