---
layout: single
title: "SpringBoot - Intro #11"
categories: springboot
Tag: [spring-intro]
sidebar: 
    nav: "docs"
---
# 스프링 데이터 JPA

## 스프링 데이터 JPA 제공 기능

- 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행
- SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있음
- 개발 생산성을 크게 높일 수 있음


## 스프링 데이터 JPA 사용 이유

1.  편리하게 도와주는 라이브러리
2. 개발 생산성 증가
3. 코드 감소
4. 리포지토리에 구현 클래스 없이 인터페이스만으로도 개발 완료
5. 기본 CRUD 기능 제공

>   개발자는 핵심 비즈니스 로직을 개발하는데 집중할 수 있음
> 
>   이러한 이유로 실무에서 관계형 데이터베이슬르 사용한다면 굉장히 유용

---
* SpringDataJpaMemberRepository 생성
*repository/SpringDataJpaMemberRepository*
<pre>
인터페이스가 인터페이스를 받을 땐 extends 
JpaRepository<t=member, id= pk type>, MemberRepository 다중 상속

스프링 data jpa 가  JpaRepository를 상속 받고 있으면 
구현체가 자동으로 생성되어 spring bean에 등록
</pre>
![인터페이스 설명 이미지](/assets/images/2022-12-30-10-22-33.png){: .full}

* SpringConfig 설정
<pre>
자동으로 만들어진 구현체를 가져다 씀
MemberRepository 객체, 생성자 생성하여 MemberService와 의존관계 셋팅
</pre>
![SpringConfig.java 이미지](/assets/images/2022-12-30-10-24-20.png){: .full}

---
 > 포스팅에 참고한 강의 링크 
 >
 >[김영한 - 스프링-입문-스프링부트](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)