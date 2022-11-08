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

공부방 이메일 인증하기 기능을 작업 중이었습니다.

사용자가 마이페이지에서 이메일 인증하기를 누르면 SendGrid를 이용해 사용자에게 이메일을 전송하고, 메일 본문의 인증하기 버튼을 누르면 코드숨 사이트 내에 있는 이메일 인증 확인 페이지에서
이메일이 성공적으로 인증되었는지 확인하는 흐름의 작업이었습니다.

로컬에서 정상적으로 기능이 동작하는 것을 확인한 후, 실제 서버에 배포했으나 이메일 인증 확인 페이지에서 API를 정상적으로 호출하지 못하는 것을 확인했습니다.

## 원인

원인은 바로 코드 내의 useEffect 내의 코드 때문이었습니다.

```js
  useEffect(() => {
    return () => {
      example(token);
    };
  }, []);
```

API 호출을 단순히 useEffect 내에 한 게 아니라 보다시피
clean-up 내에 선언한 것을 확인할 수 있죠.

그런데, 왜 개발 모드에서는 정상 작동되고 프로덕션 모드에서는 작동하지 않았던 걸까요?

바로 Strict Mode 때문이었습니다. 정확히는, React.StrictMode 때문이었습니다.
StrictMode는 애플리케이션 내의 잠재적인 문제를 알아내기 위한 도구입니다.
하위 컴포넌트들에 대한 부가적인 검사와 경고를 활성화합니다.

```js
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```

흔히 <App/>같은 상위 컴포넌트에서 볼 수 있습니다. React.StrictMode 컴포넌트로 감싸진 컴포넌트들의 자손들은 모두 Strict 모드 검사가 이루어지게 됩니다.

아무튼, 왜 React StrictMode 때문에 정상 작동 되어선 안 되는 함수 호출이 되었던 걸까요? 바로, 이중으로 의도적으로 이중으로 호출하기 때문이었습니다.
원래대로 였으면 useEffect 리턴 문에서 실행되지 않았어야할 함수가
두 번 실행되어 useEffect 안의 리턴문의 함수가 실행되는 것처럼 작동되었던 것입니다.

## 해결 방법

함수호출을 return 문에서 삭제해주고 useEffect 내에 넣어주는 것으로 간단하게 해결할 수 있는 문제였습니다.

참고로, useEffect clean-up은 메모리 누수 방지를 위해 사용합니다.

## 참고

-[https://ko.reactjs.org/docs/strict-mode.html](https://ko.reactjs.org/docs/strict-mode.html)
