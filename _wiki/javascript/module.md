---
layout  : wiki
title   : 자바스크립트에서 모듈은 어떻게 사용할까?
date    : 2022-11-07 20:44:00 +0900
updated : 2022-11-07 20:44:00 +0900
author  : 한윤석
tag     : 
toc     : true
public  : true
parent  : javascript
latex   : false
---
* TOC
{:toc}

브라우저에서 자바스크립트는 모듈 시스템이 없었습니다. 그래서 하나의 파일에 모든
코드의 내용을 작성했습니다. 초기에는 자바스크립트로 작성하는 내용이 별로 없다 보니 크게 문제가 되는 것은
없었습니다. 그런데 웹 애플리케이션이 대중화되면서 자바스크립트의 코드 양은
증가하게 되었고, 코드를 나눠서 관리하려는 니즈가 생겨났습니다.

## 파일로 분리하기

무조건 하나의 파일에 모두 작성할 필요는 없습니다. `<script>`태그를 나눠서 작성할
수 있습니다. 그러면 별도의 스크립트로 작성하거나 별도의 파일로 작성할 수 있습니다.

```html
<body>
  <div>
    <button id="button">클릭</button>
  </div>

  <script>
    const greetings = "Hi there!";
  </script>
  <script>
    const button = document.getElementById("button");
    button.addEventListener('click', () => {
      alert(greetings);
    });
  </script>
</body>
```

그런데 위와 같이 작성할 경우 몇 가지 문제점이 있습니다.

1. 코드의 순서가 영향을 줄 수 있습니다. 만약 잘못된 순서로 스크립트를 사용하면,
   올바르게 동작하지 않을 수 있습니다.
2. 같은 이름의 변수나 함수를 작성할 수 없습니다.

## 웹팩으로 해결하기

Webpack을 사용하면 별도의 모듈을 만들 수 있습니다.

```js
// src/index.js
import greetings from './greetings';

const button = document.getElementById("button");
button.addEventListener('click', () => {
  alert(greetings);
});

// src/grettings
const greetings = "Hi there!";

export default greetings;
```

위 코드는 번들링 되면 다음과 같이 됩니다.

```js
(() => {
  "use strict";
  document.getElementById("button")
    .addEventListener("click", (() => {
      alert("Hi there!")
    }))
})();
```

웹팩이 `greetings`의 변수를 처리하여 아예 문자열로 합쳐서, 모듈이 같이 동작하는
것처럼 만들어 버렸네요. 이와 같이 내부적으로 웹팩이 코드를 처리하여 하나의
파일로 번들링 해줍니다. 그래서 모듈 시스템을 사용할 수 있게 된 것입니다.

## ECMAScript modules

ES모듈은 자바스크립트에서 공식적으로 제공하는 모듈입니다. 모듈은 `import`와
`export`로 모듈은 정의하고 불러올 수 있습니다.

브라우저에서 모듈을 사용하기 위해서는 다음과 같이 `type="module"`이라고 명시해
주어야 합니다.

```js
// greetings.js
const greetings = "Hi there!";

export default greetings;
```

```html
<body>
  <div>
    <button id="button">클릭</button>
  </div>
  <script type="module">
    import greetings from './greetings.js';

    const button = document.getElementById("button");
    button.addEventListener('click', () => {
      alert(greetings);
    });
  </script>
</body>
```

## 참고

* [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
* [https://hacks.mozilla.org/2015/08/es6-in-depth-modules/](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/)
* [https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
* [https://hacks.mozilla.org/2015/08/es6-in-depth-modules/](https://hacks.mozilla.org/2015/08/es6-in-depth-modules/)
* [https://exploringjs.com/es6/ch_modules.html](https://exploringjs.com/es6/ch_modules.html)
* [https://nodejs.org/api/modules.html](https://nodejs.org/api/modules.html)
* [https://nodejs.org/api/esm.html](https://nodejs.org/api/esm.html)
