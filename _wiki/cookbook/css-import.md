---
layout  : wiki
title   : 테스트할 때 css문 import문을 해석할 수 없다
date    : 2022-10-08 12:03:00 +0900
updated : 2022-10-08 12:03:00 +0900
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

jest로 테스트로 실행할 때 css파일 import문에서 module을 찾을 수 없다는 에러가 발생합니다.

```bash
Test suite failed to run

    Cannot find module 'swiper/css' from 'src/components/home/Reviews.jsx'

    Require stack:
      src/components/home/Reviews.jsx
      src/components/home/Reviews.test.jsx

      11 | import { medias } from '../../designSystem';
      12 |
    > 13 | import 'swiper/css';
         | ^
      14 | import 'swiper/css/navigation';
```

## 원인

자바스크립트에서는 css파일을 import할 수 없습니다. 그러다보니 module을 찾을 수 없다는 에러가 발생합니다.  webpack에서 되는 이유는 `css-loader` 처럼 자바스크립트에서 css파일을 임포트할 수 있도록 js로 변환해주는 것을 사용하기 때문입니다.

## 해결방법

import문이 테스트에 영향을 주지 않기 때문에, mocking해서 처리를 해야 합니다. jest에서는 설정에서 `moduleNameMapper` 라는 옵션으로 모듈이름과 사용할 모킹 모듈로 매핑할 수 있습니다.

```jsx
module.exports = {
  // ... 생략

	moduleNameMapper: {
    'swiper/css': '<rootDir>/dummy.js'
  },

  // ... 생략
}

// dummy.js
const nothing = {};

export default nothing;
```

그러면 테스트를 실행할 때 `swiper/css` 파일을 import하는 것이 아니라, `dummy.js` 를 import합니다. `dummy.js` 는 아무것도 없는 그냥 JavaScript파일입니다. css파일을 기능을 테스트하는데 아무런 영향을 주지 않으므로, 아무것도 안하는 JavaScript파일로 대체하도록 합니다.

만약 `dummy.js` 처럼 가짜 파일을 만드는게 싫다면, 이러한 가짜 파일을 미리 만들어놓은 걸 사용할 수 있습니다.

```bash
npm install  --save-dev identity-obj-proxy
```

라이브러리를 설치한 후 `jest.config.js` 에 다음과 같이 설정합니다.

```jsx
module.exports = {
  // ... 생략

	moduleNameMapper: {
    'swiper/css': 'identity-obj-proxy'
  },

  // ... 생략
}
```

`identity-obj-proxy` 라이브러리의 실제 코드를 보면 이 파일도 아무일도 안하고, 주어진 값을 그냥 반환하는 코드인 것을 확인하실 수 있습니다.

```jsx
// https://github.com/keyz/identity-obj-proxy/blob/master/src/index.js

// ... 생략

idObj = new Proxy({}, {
  get: function getter(target, key) {
    if (key === '__esModule') {
      return false;
    }
    return key;
  }
});

module.exports = idObj;

// other-file.js
import idObj from 'identity-obj-proxy';

console.log(idObj.foo); // 'foo'
console.log(idObj.bar); // 'bar'
console.log(idObj[1]); // '1'
```

## 참고

- [https://jestjs.io/docs/configuration#modulenamemapper-objectstring-string--arraystring](https://jestjs.io/docs/configuration#modulenamemapper-objectstring-string--arraystring)
- [https://github.com/keyz/identity-obj-proxy](https://github.com/keyz/identity-obj-proxy)
