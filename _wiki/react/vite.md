---
layout  : wiki
title   : vite를 선택한 이유
date    : 2022-10-24 11:23:00 +0900
updated : 2022-10-30 11:23:00 +0900
author  : 김현지, 양민석, 우종혁
tag     : 
toc     : true
public  : true
parent  : react
latex   : false
---

* TOC
{:toc}

관리자 페이지 프로젝트를 시작하면서 초기 세팅을 진행해야 했습니다.
리액트 프로젝트를 생성하기 위한 방법에 대해 찾아보았고
`CRA`, `Webpack`, `Vite`, `Rollup.js`, `Parcel.js` 들을
사용해 프로젝트를 생성할 수 있다는 것을 알았습니다.
그중 왜 저희가 `Vite`를 선택한 이유를 설명하겠습니다.

### CRA를 선택하지 않은 이유
- 사용하지 않는 기능까지 전부 설치되기 때문에 모듈 사이즈가 크다.
- 커스텀 빌드를 하는 것이 어렵다.

### Webpack을 선택하지 않은 이유
- 개발 서버 시작할 때, 모든 모듈을 합치기 때문에 느리다.
- 간단한 작업도 플러그인이 필요하다.
- 번들 사이즈가 너무 크다.

### Rollup.js를 선택하지 않은 이유
- 웹팩에 대한 최소한의 대안이며 소규모 프로젝트에 적합하다.
- 규모가 커질수록 설정이 복잡해진다.

### Parcel을 선택하지 않은 이유
- 소규모 프로젝트에 적합한 번들러이다.

### Vite를 선택한 이유
- [dependencies](#dependencies) 그리고 [source code](#source-code) 두 가지 카테고리로 나누어 개발 서버를 시작함으로써 서버 구동 속도가 빠릅니다.

- vite는 [HMR](#hmr이란)을 지원하기 때문에, 앱 사이즈가 커져도 HMR을 포함한 갱신 시간에는 영향을 끼치지 않습니다.

### Vite란?
vite는 빠르고 간결한 모던 웹 프로젝트 개발 경험에 초점을 맞춰 탄생한
빌드 도구입니다.
vite는 브라우저에서 지원하는 ES Modules(ESM) 및 네이티브 언어로 작성된 자바스크립트 도구 등을 활용해 문제를 해결하고자 합니다.
vite는 기본적으로 최적화된 설정을 제공하지만, Plugin API 또는 JavaScript API를 이용할 수 있습니다.

#### HMR 이란?
HMR(Hot Module Replacement)은 앱을 종료하지 않고 갱신된 파일만을 교체하는 방식입니다.
어떤 모듈이 수정되면 vite는 그저 수정된 모듈과 관련된 부분만을 교체할 뿐이고, 브라우저에서 해당 모듈을 요청하면 교체된 모듈을 전달할 뿐입니다.

#### dependencies
vite의 사전 번들링 기능은 Esbuild를 사용하고 있어 기존의 번들러 대비 10-100배 빠른 번들링 속도를 보입니다.

#### source code
vite는 Native ESM을 이용해서 소스 코드를 제공합니다. 브라우저가 곧 번들러이기 때문에 특정 모듈에 대한 소스 코드를 요청하면 이를 전달합니다. 따라서 조건부 동적 import 이후의 코드는 현재 화면에서 실제로 사용이 되어야만 처리가 됩니다.

## 참고
- [Vite를 사용해야 하는 이유](https://vitejs-kr.github.io/guide/why.html)
- [webpack vs. babel vs. rollup vs. gulp vs. parcel vs. vite](https://ritza.co/articles/gen-articles/webpack-vs-babel-vs-rollup-vs-gulp-vs-parcel-vs-vite/)
- [Parcel vs. WebPack 2021](https://levelup.gitconnected.com/parcel-vs-webpack-2021-64c347bcb31)