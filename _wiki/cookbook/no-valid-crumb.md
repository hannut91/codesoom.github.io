---
layout  : wiki
title   : 서버에서 Error 403 No valid crumb was included in the request 응답이 올 때
date    : 2022-10-14 12:17:00 +0900
updated : 2022-10-14 12:17:00 +0900
author  : 안지환
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제

`로그인.http` 해서 로그인 테스트를 하려고 했습니다. 로그인 테스트를 해서 JWT토큰이 발급이 되어야 한다고 생각했었는데 다음과 같은 문제가 발생했습니다.

```bash
**HTTP/1.1 403 Forbidden
X-Content-Type-Options: nosniff
Cache-Control: must-revalidate,no-cache,no-store
Content-Type: text/html;charset=iso-8859-1
Content-Length: 554
Server: Jetty(9.4.46.v20220331)
<head>
    <meta http-equiv="Content-Type" content="text/html;charset=ISO-8859-1"/>
    <title>Error 403 No valid crumb was included in the request</title>
</head>
<body><h2>HTTP ERROR 403 No valid crumb was included in the request</h2>
<table>**
```

## 원인

이 문제의 원인은 Jenkins의 문제였습니다. 이전에 Jenkins를 테스트 해볼려고 설치 했던 것이 남아 있어서 jenkins 서버가 계속해서 실행 된 상태였습니다. 처음에는 이 문제가 Jenkins가 아닌 다른 곳에서 발생하는 문제인거 같아서 다른곳에서 분석 했지만 Jenkins의 설정 문제였습니다.

## 답변

명령어 `brew services stop jenkins` 서버를 종료 하면 위에 에러가 해결이 됩니다. 

하지만 Jenkins를 사용하는 경우에는 설정을 해야 합니다.

### 참고사항

- ['403: 유효한 부스러기 없음' Jenkins GitHub 웹훅 오류 수정](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Fix-No-Valid-Crumb-Error-Jenkins-GitHub-WebHook-Included)
- [[jenkins 403, no valid crumb] 에러 리포트](https://blog.mglee.dev/blog/jenkins-403-no-valid-crumb-%EC%97%90%EB%9F%AC-%EB%A6%AC%ED%8F%AC%ED%8A%B8)
