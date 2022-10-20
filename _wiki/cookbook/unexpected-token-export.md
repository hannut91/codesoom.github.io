---
layout  : wiki
title   : 테스트에서 외부 라이브러리를 사용할 때 `Unexpected token 'export'` 에러가 난다
date    : 2022-10-08 13:13:00 +0900
updated : 2022-10-08 13:13:00 +0900
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

jest로 컴포넌트를 테스트하려고 하는데, 사용하고 있는 외부 라이브러리에서 올바른 값을 읽지 못해서 테스트가 실패하고 있다.

## 예시

```bash
({"Object.<anonymous>":function(module,exports,require,__dirname,__filename,global,jest){export * from './swiper-react.js';
                                                                                             ^^^^^^
SyntaxError: Unexpected token 'export'

> 1 | import { Swiper, SwiperSlide } from 'swiper/react';
```

## 원인

Jest는 Node.js로 실행됩니다. 그래서 ES Module들을 CommonJS Module로 transpile되어야 합니다. 그런데 Jest의 `transformIgnorePatterns` 의 기본 옵션은 node_modules의 파일들을 transpile하지 않습니다. 그래서 CommonJS 모듈이 아닌 라이브러리를 사용할 수 없습니다.

## 해결과정

ES Module인 라이브러리 파일을 CommonJS Module로 transpile 되도록 하기 위해서 `transformIgnorePatterns` 의 옵션을 수정해야 합니다.

```jsx
module.exports = {
  // ... 생략

  transformIgnorePatterns: [
    'node_modules/(?!swiper|ssr-window|dom7)',
  ],

  // ... 생략
};
```

정규식에 맞는 모듈들이 무시되는 것이므로, transpile되어야 하는 파일들이 포함되지 않는 것을 무시하도록 위와 같이 설정하면 됩니다.

## 참고

- [https://jestjs.io/docs/configuration#transformignorepatterns-arraystring](https://jestjs.io/docs/configuration#transformignorepatterns-arraystring)
- [https://jestjs.io/docs/code-transformation](https://jestjs.io/docs/code-transformation)
