---
layout: single
title: "SpringBoot - Intro #8"
categories: springboot
Tag: [spring-intro]
sidebar: 
    nav: "docs"
toc: true
---
# 회원 웹 기능 (조회) / h2 설치 및 연결/ 연결을 위한 설정

### 웹 기능 - 조회

*MemberController*에 리스트를 보여주는 메소드 생성 *@GetMapping("/members")*

메소드 안에 멤버 리스트에 대한 데이터를 불러오고 멤버 리스트를 담는 것을 설정
넘어갈 페이지 설정 `return members/memberList` 후 템플릿에 파일 생성

![웹 기능 - 조회 이미지](/assets/images/2022-12-29-10-17-39.png){: .full}


![view source](/assets/images/2022-12-29-10-23-40.png){: .full}

프로퍼티 방식으로 getId, getName 을 이용하여 가져오는 것

---

### 스프링 DB 접근 기술

<pre>
H2 데이터 베이스 설치
다운받고 h2.bat 파일을 실행 시킴 끄면 서버가 죽음
처음 설정을 위한 폴더(test) 를 사용자 계정(C:\Users\{사용자})에 만들어 줘야함
(보안 때문에 자동 생성이 안됨)
</pre>

[h2database](https://www.h2database.com/html/download-archive.html){: .btn .btn--inverse}에서 1.4.200 버전으로 설치 후 실행 시 폴더를 만들 필요 없음


1. ![입력 예시](/assets/images/2022-12-29-10-36-30.png){: .align-center}
    생성을 한 뒤 위 이미지 처럼 입력하고 연결을 하면 DB가 만들어짐


2.  ![경로 설정](/assets/images/2022-12-29-10-35-40.png){: .align-center}
    ​만들어진 후 파일 충동이 발생할 수 있기에 여러 군데에서 접근을 위해 위 이미지 처럼 설정

3.  ![sql directoty](/assets/images/2022-12-29-10-38-12.png){: .align-center}
    sql 문 관리를 위한 파일 만들어 줌

```sql
create table member
(
 id bigint generated by default as identity, --generated by default as identity: db가 알아서 채워주기 위한 것
 name yarchar(255),
 primary key(id)
);
```

### db와 연결을 위한 설정

1. _build.gradle_ 파일에 libraries 추가
    ```
    dependencies {
        implementation 'org.springframework.boot:spring-boot-starter-jdbc'
        runtimeOnly 'com.h2database:h2'
    }
    ```

2. _application.properties_ 에 명령어 추가
    ```
    spring.datasource.url=jdbc:h2:tcp://localhost/~/test
    spring.datasource.driver-class-name=org.h2.Driver
    ```

---
 > 포스팅에 참고한 강의 링크 
 >
 >[김영한 - 스프링-입문-스프링부트](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)