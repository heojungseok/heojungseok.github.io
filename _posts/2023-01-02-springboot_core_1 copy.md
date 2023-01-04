---
layout: single
title: "SpringBoot - Core #01"
categories: springboot_core
Tag: [spring-core]
sidebar: 
    nav: "docs"
---
# SpringBoot 환경 설정/ 비즈니스 요구 사항 설계/ 회원 도메인 설계

## 초기 환경 설정

**Project:** Gradle Project

**Spring Boot:** 2.6.7 

**Language:** Java 

**Packaging:** Jar 

**Java:**8 (강의는 11)

***Project Metadata***

- groupId: hello
- artifactId: core

***Dependencies***: 선택 안함

**※ IntelliJ Gradle 대신에 자바 직접 실행**

최근 IntelliJ 버전은 Gradle을 통해서 실행 하는 것이 기본 설정이다. 이렇게 하면 실행속도가 느리다. 다음과 같이 변경하면 자바로 바로 실행해서 실행속도가 더 빠름
`Preferences Build → Execution Deployment → Build Tools → Gradle `

- Build and run using: Gradle IntelliJ IDEA 
- Run tests using: Gradle IntelliJ IDEA

![설정 이미지](/assets/images/2023-01-02-22-14-52.png){: .full}

## 비즈니스 요구사항 설계

1. 회원
   - 회원 가입 및 조회 가능
   - 일반과  VIP 두 가지 등급
   - 회원 데이터는 자체 DB 구축, 외부 시스템과 연동할 수 있음(미확정)
2. 주문과 할인 정책
   -  회원은 상품 주문 가능
   -  회원 등급에 따라 할인 정책 적용 가능
   -  모든 VIP는 1000원 할인해주는 고정 금액 할인 적용(추후 변경 가능)
   -  할인 정책은 변경 가능성 높음.. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

## 회원 도메인 설계

**회원 도메인 협력 관계**

![회원 도메인 협력 관계 이미지](/assets/images/2023-01-02-22-22-48.png){: .full}

회원 저장소: 회원 데이터에 접근하는 계층(인터페이스)

메모리 회원 저장소: 테스트, 로컬 개발용으로만 쓰임(자바 코드로)

**회원 클래스 다이어그램**

![회원 클래스 다이어그램](/assets/images/2023-01-02-22-24-05.png){: .full}

*실제 구현 단계*

    impl: 구현체
    MemberRepository → 회원 저장소 역할, 
    구현 클래스: 밑에 두개 또는 외부 저장소(MemoryMemberRepository, DbMemberRepository, ...)

![회원 객체 다이어그램 이미지](/assets/images/2023-01-02-22-25-58.png){: .full}

실제 서버에 올라오면 객체 간에 메모리 참조가 어떻게 되는지 설명 되는 것

실제 유효한 인스턴스들의 참조

회원 서비스: ***MemberServiceImpl***

 > 포스팅에 참고한 강의 링크 
 >
 >[김영한 - 스프링-핵심-원리-기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)