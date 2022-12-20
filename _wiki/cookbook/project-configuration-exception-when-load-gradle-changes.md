---
layout  : wiki
title   : Gradle 변경사항 로드 시 발생하는 ProjectConfigurationException
date    : 2022-12-20 11:33:00 +0900
updated : 2022-12-20 11:33:00 +0900
author  : 한지영
tag     : 
toc     : true
public  : true
parent  : cookbook
latex   : false
---
* TOC
{:toc}

## 문제

Java 17버전으로 스프링 프로젝트를 새로 생성한 후 Gradle 로딩을 하던 중, 아래와 같은 오류가 발생하며 로딩에 실패했습니다.

```
A problem occurred configuring root project 'api'.
> Could not resolve all files for configuration ':classpath'.
   > Could not resolve org.springframework.boot:spring-boot-gradle-plugin:3.0.0.
     Required by:
         project : > org.springframework.boot:org.springframework.boot.gradle.plugin:3.0.0
      > No matching variant of org.springframework.boot:spring-boot-gradle-plugin:3.0.0 was found. The consumer was configured to find a runtime of a library compatible with Java 15, packaged as a jar, and its dependencies declared externally, as well as attribute 'org.gradle.plugin.api-version' with value '7.5.1' but:
          - Variant 'apiElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.0 declares a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares an API of a component compatible with Java 17 and the consumer needed a runtime of a component compatible with Java 15
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')
          - Variant 'javadocElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.0 declares a runtime of a component, and its dependencies declared externally:
              - Incompatible because this component declares documentation and the consumer needed a library
              - Other compatible attributes:
                  - Doesn't say anything about its target Java version (required compatibility with Java 15)
                  - Doesn't say anything about its elements (required them packaged as a jar)
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')
          - Variant 'mavenOptionalApiElements' capability org.springframework.boot:spring-boot-gradle-plugin-maven-optional:3.0.0 declares a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares an API of a component compatible with Java 17 and the consumer needed a runtime of a component compatible with Java 15
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')
          - Variant 'mavenOptionalRuntimeElements' capability org.springframework.boot:spring-boot-gradle-plugin-maven-optional:3.0.0 declares a runtime of a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares a component compatible with Java 17 and the consumer needed a component compatible with Java 15
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')
          - Variant 'runtimeElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.0 declares a runtime of a library, packaged as a jar, and its dependencies declared externally:
              - Incompatible because this component declares a component compatible with Java 17 and the consumer needed a component compatible with Java 15
              - Other compatible attribute:
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')
          - Variant 'sourcesElements' capability org.springframework.boot:spring-boot-gradle-plugin:3.0.0 declares a runtime of a component, and its dependencies declared externally:
              - Incompatible because this component declares documentation and the consumer needed a library
              - Other compatible attributes:
                  - Doesn't say anything about its target Java version (required compatibility with Java 15)
                  - Doesn't say anything about its elements (required them packaged as a jar)
                  - Doesn't say anything about org.gradle.plugin.api-version (required '7.5.1')

* Try:
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Exception is:
org.gradle.api.ProjectConfigurationException: A problem occurred configuring root project 'api'.
	...
```

## 원인
에러문을 살펴보면, '현재 Java 17과 호환되는 구성 요소를 선언했는데, 소비자는 Java 15와 호환되는 구성 요소를 필요로 하고 있다'라고 말하고 있습니다.
즉 실제 구성요소를 사용하는 소비자인 Gradle JVM이 Java 15와 호환되도록 설정되어 있기 때문에 위와 같은 에러가 발생한 것입니다.

## 해결 방법
Intellij 설정 > Build, Execution, Deployment > Build Tools > Gradle > Gradle JVM에서 자바 버전을 17로 바꿔주면 Gradle Load가 성공적으로 수행됩니다.

## 참고
- [Incompatible because this component declares a component compatible with Java 11 and the consumer needed a component compatible with Java 10](https://stackoverflow.com/questions/72117858/incompatible-because-this-component-declares-a-component-compatible-with-java-11)
