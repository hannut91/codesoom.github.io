---
layout  : wiki
title   : 새탭을 여는 E2E테스트를 작성할 때 Tab이 없다고 하는 경우
date    : 2022-10-06 11:08:00 +0900
updated : 2022-10-06 11:08:00 +0900
author  : 김현지
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제

⚠️ 신청하기 버튼을 누르면 네이버 스토어팜으로 이동하는 테스트를 기대했습니다.
그러나 git 강의를 구매하는 페이지가 아닌 다른 url을 입력해도 테스트가 통과 되는 문제가 있었습니다. 
openNewTab은 새로운 탭이 열렸는지에 대한 여부만 확인하는 method였기 때문에 이러한 문제가 발생하는 것이었습니다.

![스크린샷 2022-10-05 오후 6.13.35.png](/resource/wiki/cookbook/new-tab/a.png)

## 문제해결

- 최근 url을 확인하는 method인 seeInCurrentUrl을 사용하였습니다.
- 새로운 탭이 열리고 해당 탭의 url을 확인하는 것이기 때문에 seeInCurrentUrl하기 전에 특정 탭으로 포커스를 이동시켜주는 switchToNextTab을 해주었습니다.

⚠️ 스위치할 탭이 없다는 에러가 발생했습니다.
테스트에서는 탭이 너무 빠르게 열리고 닫히기 때문에 발생한 에러였습니다.

![스크린샷 2022-10-06 오전 11.04.23.png](/resource/wiki/cookbook/new-tab/b.png)

- 몇 초 동안 실행을 중지 시키는 method인 wait에 sec 파라미터 1을 넣어주어서 탭이 열릴때 시간을 1초 동안 중지 시켰습니다.

![스크린샷 2022-10-06 오전 10.11.21.png](/resource/wiki/cookbook/new-tab/c.png)

## 출처

- [WebDriver \| CodeceptJS](https://codecept.io/helpers/WebDriver/#methods)
