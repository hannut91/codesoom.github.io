---
layout  : wiki
title   : Git은 폴더명 대소문자를 변경한 걸 못 알아챈다
date    : 2022-11-12 12:54:00 +0900
updated : 2022-11-12 12:54:00 +0900
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

폴더명을 대문자에서 소문자로 변경 후 merge 했으나, 다른 팀원들이 변경된 사항을 pull 받아도 여전히 폴더명이 대문자인 상태였습니다. 그로 인해 import 해 오는 폴더 경로가 틀린 것으로 되어 빌드가 실패되는 현상이 발생했습니다. 

## 원인

git은 대소문자를 구분하지 않는 것으로 기본 설정이 되어있습니다. git은 운영체제의 파일시스템에 의존합니다. 윈도우나 맥의 파일 시스템이 파일이나 폴더의 대소문자가 달라도 같은 것으로 인식하기 때문에 Git 또한 대소문자를 구분하지 않습니다.

그래서 기본 설정이 대소문자를 구분하지 않는 상태로 파일, 폴더의 대소문자를 변경하는 경우 로컬에서는 파일, 폴더명이 변경되지만 스테이지 상에는 변경이 이뤄지지 않게 됩니다.


## 해결 방법

#### 1. 대소문자를 구분하도록 git 설정을 변경해줍니다.

```
git config core.ignorecase false
or
git config --global core.ignorecase false
```
그 후 캐쉬도 삭제해줍니다.
```
git rm -r --cached .
git add .
git commit -m "캐쉬를 삭제하라"
```

#### 2. git 명령을 사용해서 변경합니다.

```
$ git mv Test.ts test.ts
```
이렇게 하면 삭제하고 다시 올린 것처럼 변경 이력이 추적됩니다.

#### 3. 직접 파일을 삭제했다가 다시 push합니다.

직접 파일을 로컬에서 삭제한 걸 커밋 후, 다시 생성하여 커밋 후 푸시 하면 위 git mv 명령어를 사용한 것과 똑같습니다. 저희 팀은 폴더명을 수정한 걸 커밋하고, 다시 원래 목표의 이름대로 변경한 걸 푸시 해서 해결했습니다.

## 참고

- [https://github.com/git/git/commit/baa37bff9a845471754d3f47957d58a6ccc30058](https://github.com/git/git/commit/baa37bff9a845471754d3f47957d58a6ccc30058)
- [git-config](https://git-scm.com/docs/git-config/2.14.6#Documentation/git-config.txt-coreignoreCase)
- [git-mv](https://git-scm.com/docs/git-mv)
