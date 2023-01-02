---
layout: single
title: "SpringBoot - Intro #12"
categories: springboot
Tag: [spring-intro]
sidebar: 
    nav: "docs"
---
# AOP, AOP 적용

## AOP

**AOP가 필요한 상황**

 - 모든 메소드의 호출 시간을 측정

 - 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern)

 - 서비스의 시간을 측정하고 싶을 때

**AOP 적용**

 - AOP: Aspect Oriented Programming 관점지향

 - 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 분리

![스프링 컨테이너 이미지](/assets/images/2022-12-30-10-58-20.png){: .full}

* **TimeTraceAop.java 생성**
![TimeTraceAop.java 이미지](/assets/images/2022-12-30-10-59-27.png){: .full}

<pre>
aop 패키지 생성 후 TimeTraceAop 클래스 생성, @Aspect 어노테이션
메소드를 생성 후 SpringConfig 에서 @Bean 으로 등록하거나
TimeTraceAop 클래스에 @Component 해도 됨
</pre>
---

**AOP 를 만듦으로써** 

 - 핵심 관심 사항(회원가입, 회원조회 등)과 공통 관심 사항(시간 측정) 분리

 - 공통 관심 사항을 별도의 로직으로 만듦

 - 핵심 관심 사항을 깔끔하게 유지할 수 있음

 - 변경이 필요하면 별도의 로직만 변경

 - 원하는 적용 대상 선택 가능 @Around("execution(* <span style="color:green">*com.hello.hellospring*</span>...*(..))") 초록 부분 수정하여 가능

*@Around("execution") : 어디에 AOP를 사용할 것인지 설정하는 어노테이션 문법이 따로 존재 보통 패키지 레벨로 함*

---

**의존관계 및 전체 흐름도**

<p align="center" width="100%">
    <img alt=" AOP 적용 전 의존관계 이미지" width="45%" src="/assets/images/2022-12-30-11-08-08.png">
    <img alt=" AOP 적용 후 의존관계 이미지" width="53%" src="/assets/images/2022-12-30-11-09-33.png">
</p>

<p align="center" width="100%">
    <img alt=" AOP 적용 전 전체 이미지" width="45%" src="/assets/images/2022-12-30-11-08-08.png">
    <img alt=" AOP 적용 후 전체 이미지" width="53%" src="/assets/images/2022-12-30-11-21-01.png">
</p>
---

*TimeTraceAop.java*

```java
package com.hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimeTraceAop {
    @Around("execution(* com.hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {

        long start = System.currentTimeMillis();
        System.out.println("START :" + joinPoint.toLongString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString() + " " + timeMs + "ms");
        }
    }
}
```

---
 > 포스팅에 참고한 강의 링크 
 >
 >[김영한 - 스프링-입문-스프링부트](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)