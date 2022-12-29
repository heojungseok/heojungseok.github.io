---
layout: single
title: "SpringBoot - Basic #10"
categories: springboot
Tag: [springboot-basic]
---
# JPA

## JPA를 사용하는 이유
    - 기존의 반복 코드는 물론이고, 기본적인 SQL도 JPA가 직접 만들어서 실행

    - SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있음

    - 개발 생산성을 크게 높일 수 있음

---

*   _build.gradle_ jpa 관련 라이브러리 추가 및 _application.properties_ jpa 관련 설정 추가
   
```
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
//    implementation 'org.springframework.boot:spring-boot-starter-jdbc' //jpa에가 jdbc를 포함해서 없애도 됨
    
}
```

```
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none #자동으로 테이블 생성 하는 것을 방지

#spring.jpa.hibernate.ddl-auto=create 설정 시 테이블 자동 생성
```

> _**JPA = 객체와 ORM(Object Relational Mapping)**_

* Member class에 @Entity 어노테이션 생성

<pre>
어노테이션을 통해 mapping
@Id, @GeneratedValue(strategy = GenerationType.Identity) 생성
DB 자동으로 id를 생성하는 것 = Identity
</pre>

![설정 이미지](/assets/images/2022-12-29-11-47-56.png)
`변수 name을 테이블의 컬럼으로 쓰고 싶으면 @Column 어노테이션에 (name = "컬럼명") 설정`

* JpaMemberRepository 생성

  *   Jpa는 EntityManager로 동작
    ![JpaMemberRepository 이미지](/assets/images/2022-12-29-12-00-19.png)

  *   JPA를 쓸 때는 항상 Transaction 있어야 함 - 데이터를 저장하고 변경하기 위해서
    MemberService 클래스에 생성
    ![@Transactional 사용](/assets/images/2022-12-29-12-01-55.png)

  *   JpaMemberRepository가 완성 됐으니 당연히 SpringConfig 안에 MemberRepository 설정
   EntityManager가 필요함
    ![springConfig 설정 이미지ㄴ](/assets/images/2022-12-29-12-02-49.png)

  * 테스트 로그 이미지
    ![테스트 로그 이미지](/assets/images/2022-12-29-12-03-19.png)

_Member.java_
```java
package com.hello.hellospring.domain;

import javax.persistence.*;

@Entity //JPA 가 관리하는 Entity
public class Member {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) //primary key 생성
    private Long id; //데이터를 구분 및 저장 하기 위한 시스템상 임의의 값
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

_JpaMemberRepository.java_
```java
package com.hello.hellospring.repository;

import com.hello.hellospring.domain.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em; //jpa를 쓰기위해 EntityManager를 주입 받아야 함

    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        //Entity를 대상으로 쿼리를 던짐
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

_SpringConfig.java_

```java
package com.hello.hellospring;

import com.hello.hellospring.repository.JpaMemberRepository;
import com.hello.hellospring.repository.MemberRepository;
import com.hello.hellospring.service.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;

@Configuration
public class springConfig {

    private EntityManager em;

    public springConfig(EntityManager em) {
        this.em = em;
    }

    @Bean
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository() {
        return new JpaMemberRepository(em);
    }
}
​
```

---
 > 포스팅에 참고한 강의 링크 
 >
 >[김영한 - 스프링-핵심-원리-기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)