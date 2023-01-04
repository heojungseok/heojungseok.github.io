---
layout: single
title: "SpringBoot - Core #02"
categories: springboot_core
Tag: [spring-core]
toc: true
sidebar:
  nav: "docs"
---

# 회원 도메인 개발/ 회원 도메인 실행과 테스트

## 회원 도메인 개발

### 회원 엔티티

**회원 등급**

_enum 설정_

```java
package hello.core.member;

public enum Grade {
// 회원 등급을 나누기 위함
 BASIC,
 VIP
}
```

**회원 엔티티**

```java
package hello.core.member;

public class Member {

//회원에 대한 속성들
 private Long id;
 private String name;
 private Grade grade;

 public Member(Long id, String name, Grade grade) {
 this.id = id;
 this.name = name;
 this.grade = grade;
 }

//데이터를 가져오고 뽑기 위함
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
 public Grade getGrade() {
 return grade;
 }
 public void setGrade(Grade grade) {
 this.grade = grade;
 }
}
```

### 회원 저장소

**회원 저장소 인터페이스**

```java
package hello.core.member;

public interface MemberRepository {

    void save(Member member); //회원을 저장

    Member findBy(Long memberid); //회원의 Id로 회원 찾는 기능
}
```

**메모리 회원 저장소 구현체**<br>
**(원래는 인터페이스와 구현체를 같은 패키지에 두지 않음, 오류처리 또한 생략)**

```java
package hello.core.member;

import java.util.HashMap;
import java.util.Map;

public class MemoryMemberRepository implements MemberRepository{

    private static Map<Long, Member> store = new HashMap<>(); //저장소

    @Override
    public void save(Member member) {
        store.put(member.getId(), member);
        //오류 처리를 생략함
    }

    @Override
    public Member findBy(Long memberid) {
        return store.get(memberid);
    //오류 처리를 생략함
}

}
```

> HashMap 은 동시성 이슈가 발생할 수 있다. 이런 경우 ConcurrentHashMap 을 사용하자.

### 회원 서비스

**회원 서비스 인터페이스**

```java
package hello.core.member;

public interface MemberService {

    void join(Member member); //회원 가입

    Member findMember(Long MemberId); //회원 조회
}
```

**회원 서비스 구현체**<br>
**(구현체가 하나만 있을 땐 인터페이스명+Impl 관례)**

```java
package hello.core.member;

public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository = new MemoryMemberRepository();

    @Override
    public void join(Member member) {
        memberRepository.save(member);

    }

    @Override
    public Member findMember(Long MemberId) {
        return memberRepository.findBy(MemberId);
    }
}
```

<pre>
- 가입을 하고 회원을 찾기 위한 memberRepository(회원 저장소) 인터페이스를 만듦

- 인터페이스만 가지고 있으면 nullpointException 발생, 그러므로 구현 객체를 선택
</pre>

join메소드에서 save를 호출하면 다형성에 의해서 MemoryMemberRepository의 인터페이스가 아닌 구현체 _(***Impl.java)_ 의 save()가 호출 됨

## 회원 도메인 개발

### 회원 도메인 - 회원 가입 main

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;

public class MemberApp {

    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        Member member = new Member(1L, "mamberA", Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("new member = " + member.getName());
        System.out.println("find Member = " + findMember.getName());
    }
}
```
### 회원 도메인 - 회원 가입 테스트

```java
package hello.core.member;

import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {

    MemberService memberService = new MemberServiceImpl();
    @Test
    void join(){
        //given: 주어진 환경
        Member member = new Member(1L, "memberA", Grade.VIP);

        //when: 이럴 때
        memberService.join(member);
        Member findMember = memberService.findMember(1L);
        //then: 이렇게 됨
        Assertions.assertThat(member).isEqualTo(findMember);
    }
}
```
<pre>
회원 도메인 설계의 문제점: 설계상 문제점, 
다른 저장소로 변경할 때 OCP 원칙 준수, DIP 준수

→ 의존관계가 인터페이스 뿐만 아니라 구현까지 
    모두 의존하는 문제점 발생
</pre>

> 포스팅에 참고한 강의 링크
>
> [김영한 - 스프링-핵심-원리-기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)
