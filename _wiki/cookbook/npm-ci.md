---
layout  : wiki
title   : 협업 시 pacakge-lock.json이 계속 변경될 때
date    : 2022-10-23 12:17:00 +0900
updated : 2022-10-23 12:17:00 +0900
author  : 우종혁
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제

협업을 진행할 프로젝트를 git을 통해 clone 받은 후 `npm install` 을 하면,
`package-lock.json`이 변경됩니다.

## 원인
`npm install` 커맨드는 `package.json` 내의 dependencies와 devDependencies를 기준으로
패키지 파일을 설치하게 되어있는데 `pacakge.json`은 버전 정보를 저장할 때,
version ragne로 명시되어 있습니다.
- `"react": "^17.0.1"`

위와 같이 범위로 버전을 표기하다 보니, 협업하는 과정에서 `npm install`을 할 때마다, 버전 정보가 변경되어
`pacakge.lock.json` 파일 또한 변경되는 것이었습니다.

## 해결 방법
`npm install` 대신 [`npm ci`](#npm-ci란)를 사용하여 문제를 해결했습니다.
하지만 실수로 `npm install` 커맨드를 입력할 수 있기 때문에 방법을 찾기 시작했고 제가 선택한 방법은 `preinstall`이었습니다.
`preinstall`은 말 그대로 `install`을 하기 전, 실행하는 것을 의미합니다.
`preinstall`을 적용하는 방법은 다음과 같습니다.
```json
{
  "scripts": {
    "preinstall": "node tools/preinstall-script.js"
  }
}
```
pacakge.json에 있는 `scripts`에 `preinstall`을 추가한 뒤,
루트 폴더 아래에 tools라는 폴더를 생성 후 `preinstall-script.js`를 생성합니다.
`preinstall-script.js`의 코드는 다음과 같습니다.
```js
if (process.env.npm_command === 'install') {
  console.error('npm ci를 사용하세요.')
  process.exit(1);
}
```
위의 코드를 살펴보면 커맨드에 `install`을 입력하면 에러 로그를 출력하고 설치를 중단합니다.
에러 로그에는 `npm ci`를 사용하라는 로그가 출력되어 실수로 `npm install`을 하는 일이 없어져
`package.lock.json`을 변경하지 않고 사용할 수 있게 됩니다.

## npm ci란?

`npm ci`는 `npm install`의 경우 비슷하지만 다음과 같은 차이점이 있습니다.

- 프로젝트에 기존 `package-lock.json` 또는 `npm-shrinkwrap.json`이 있어야 합니다.
- `package-lock.json`의 의존성 `package.json`의 의존성과 일치하지 않으면 
`npm ci`는 `package-lock.json`을 업데이트하지 않고 오류와 함께 종료됩니다.
- `npm ci`는 한 번에 전체 프로젝트만 설치할 수 있습니다. 이 명령으로는 개별 의존성을 추가할 수 없습니다.
- 만약 `node_modules`가 이미 있는 경우 `npm ci`가 설치를 시작하기 전에 자동으로 제거됩니다.
- 쓰기 권한이 없습니다. 오직 `package-lock.json`을 읽고 의존성 목록을 설치하게 됩니다.

## 참고

- [npm-ci](https://docs.npmjs.com/cli/v7/commands/npm-ci)
- [https://github.com/yarnpkg/yarn/issues/4895](https://github.com/yarnpkg/yarn/issues/4895)