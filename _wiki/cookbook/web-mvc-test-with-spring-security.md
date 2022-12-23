---
layout  : wiki
title   : WebMvcTest에서 SpringSecurity 설정값 사용하기
date    : 2022-12-23 17:30:00 +0900
updated : 2022-12-23 17:30:00 +0900
author  : 이수빈
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제
`@WebMvcTest`에서 Spring Security 설정값이 제대로 등록되지 않아 `permitAll`로 설정한 URI에서도 401 상태코드가 반환되고 있었다.

## 예시
Spring Boot는 3.0.0, Spring Security는 6.0.0 버전이며, 코드는 Kotlin으로 작성됐다.

먼저 SpringSecurity 설정부터 살펴보자.
```kotlin
@EnableWebSecurity
@Configuration
class SecurityConfig {

    @Bean
    fun filterChain(http: HttpSecurity): SecurityFilterChain {
        http.csrf().disable()
            .headers()
            .frameOptions().disable()
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests()
            .requestMatchers("/").permitAll()

        return http.build()
    }
}
```
`.requestMatchers("/").permitAll()`설정이 `/`요청에 대해 모두 허용한다는 뜻이다. 즉, 인증정보가 없더라도 `/`경로는 모두 접근할 수 있다.
Health Check를 위해 해당 경로만 모두 허용 설정을 해놓은 것이다.

다른 설정은 이 글과 상관이 없으므로 따로 설명하지 않겠다.
> 참고로 [Spring Security 설정 방법이 5.7.0-M2 버전을 기점으로 `WebSecurityConfigurerAdapter`가 deprecated 됐다.](https://spring.io/blog/2022/02/21/spring-security-without-the-websecurityconfigureradapter)

`HealthCheckController`는 이렇게 생겼다. `GET /`요청이 오면 `"Hello!"`를 응답하는 간단한 컨트롤러다.
```kotlin
@RestController
class HealthCheckController {
    @GetMapping("/")
    fun hello() = "Hello!"
}
```

그리고 이 `HealthCheckController` 테스트에서 문제가 발생했다.
```kotlin
@WebMvcTest(HealthCheckController::class)
class HealthCheckControllerTest(mockMvc: MockMvc) : DescribeSpec({
    describe("GET /") {
        it("Hello!를 응답한다") {
            mockMvc.get("/")
                .andExpect {
                    status { isOk() }
                    content {
                        string(containsString("Hello!"))
                    }
                }
        }
    }
})
```
`@WebMvcTest`로 컨트롤러만 테스트하는 아주 간단한 테스트 코드다.
하지만 응답이 기대한 `200 Ok`가 아닌 `401 UnAuthorized`이 와서 실패한다.

Spring Security 설정에서 `/` 경로에 대해 모든 접근을 허용했기 때문에 설정한 Bean이 잘 등록됐다면 `200 Ok`를 응답받아야 하는데 말이다.

## 원인
`@WebMvcTest`의 스캐닝 필터에 지정된 유형과 일치하지 않아 `Security Config`클래스가 스캐닝 필터에서 선택되지 않았고, 그래서 설정 정보가 읽히지 않은 것이다.

`@WebMvcTest`애노테이션은 MVC 테스트와 관련된 `@Controller`, `@ControllerAdvice` 같은 구성만 로드한다. 만약 전체 애플리케이션 구성을 로드해서 테이스하고 싶다면 `@WebMvcTest`가 아닌 `@SpringBootTest`와 `@AutoConfigureMockMvc`를 조합해 사용해야 한다.

## 해결 과정
하지만 `@SpringBootTest`는 스프링 전체 구성을 로드하기 때문에 상대적으로 테스트 소요 시간이 길다.

[Spring Boot 6.0.0 공식문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto.testing.slice-tests)를 참고해 해결할 수 있었다. `@WebMvcTest`를 하면서 MVC 관련 구성 외에 다른 구성을 로드하고 싶다면 `@Import`애노테이션을 활용해야 한다.

`@Import`애노테이션에서 설정한 클래스에 등록된 모든 Bean들을 로드해서 테스트가 실행된다. 그래서 `@Configuration`애노테이션을 붙인 설정 클래스는 작게 쪼갤수록 좋다. 테스트 시 쓸모없는 설정까지 로드하지 않을 수 있기 때문이다.

```kotlin
@Import(SecurityConfig::class)
@WebMvcTest(HealthCheckController::class)
class HealthCheckControllerTest(mockMvc: MockMvc) : DescribeSpec({
    describe("GET /") {
        it("Hello!를 응답한다") {
            mockMvc.get("/")
                .andExpect {
                    status { isOk() }
                    content {
                        string(containsString("Hello!"))
                    }
                }
        }
    }
})
```

## 참고
- [Spring Boot 6.0.0 공식문서](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto.testing.slice-tests)
