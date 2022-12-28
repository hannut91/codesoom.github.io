---
layout  : wiki
title   : 스프링 시큐리티에서 @AuthenticationPrincipal를 쓰면 안 되는 이유
date    : 2022-12-28 17:52:00 +0900
updated : 2022-12-28 17:52:00 +0900
author  : 한윤석
tag     : 
toc     : true
public  : true
parent  : scrum
latex   : false
---
* TOC
{:toc}

스프링 시큐리티에는 `@AuthenticationPrincipal`이라는 어노테이션이 있습니다.
이 어노테이션은 `Authentication.getPrincipal()`을 실행하고 결과를 핸들러 메서드 인수로
넣어줍니다.

> Annotation that is used to resolve Authentication.getPrincipal() to a method

`getPrincipal`의 반환 타입이 변경된다면, principal을 사용하는 곳의 타입을 변경해
주어야 합니다. 하지만 `@AuthenticationPrincipal`을 사용하면 컴파일러가 타입에
대해 경고해 주지 않습니다. 실제로 코드가 호출될 때만이 문제가 발생한다는 것을
발견할 수 있습니다. 예를 들어 다음과 같은 예제가 있다고 해보겠습니다.

```kotlin
fun handleProjectCreate(
    authentication: UserAuthentication,
    @RequestBody request: ProjectCreateRequest
): Project {
    val userId: Long = authentication.principal

    return projectCreateService.create(
        userId = userId,
        name = request.name,
        description = request.description,
        domain = request.domain
    )
}
```

`@AuthenticationPrincipal`은 위 설명과 같이 Authentication.getPrincipal()을 한 값을 함수 argument로 추가해 주어서 
`val userId = authentication.principal`를 안 해도 돼서 더 편할 수 있습니다. 

```kotlin
fun handleProjectCreate(
    @AuthenticationPrincipal userId: Long,
    @RequestBody request: ProjectCreateRequest
): Project {
    return projectCreateService.create(
        userId = userId,
        name = request.name,
        description = request.description,
        domain = request.domain
    )
}
```

하지만 만약 `Authentication.getPrincipal`이 다음과 같이 변경하여 반환 타입이 변경되면 어떻게 될까요?

```kotlin
class UserAuthentication(
    private val id: Long
) : AbstractAuthenticationToken(authorities()) {
    // ... 생략

    override fun getPrincipal(): User = user // 반환 타입이 Long에서 User로 바뀜
    
    // ... 생략
}
```

getPrincipal의 응답 타입이 변경되어, 더 이상 getPrincipal의 응답 타입이
Long이 아닙니다. 따라서 컴파일 타임에 타입에 대해서 에러가 발생해야 합니다. 하지만 실제로는
컴파일러는 아무런 경고도 해주지 않습니다. 이 에러는 실제로 API 호출하여 코드가 실행될 때 에러가 발생합니다. 

```bash
java.lang.IllegalArgumentException: null
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77) ~[na:na]
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
# ... 생략
```

만약 그래도 `authentication: UserAuthentication`를 사용하고 `val userId = authentication.principal` 이렇게 사용한다면 userId를 사용하는 곳에서 타입이 맞지 않는다고 컴파일러가 아주 빠르게 피드백을 줍니다

## 참고

* [Annotation Interface AuthenticationPrincipal](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/core/annotation/AuthenticationPrincipal.html)
