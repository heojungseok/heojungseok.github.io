---
layout: single
title: "SpringBoot - Basic #9"
categories: springboot
Tag: [springboot-basic]
---
# DB 연결/ 스프링 통합 테스트/ 스프링 JdbcTemplate

**DB 연결 이전 포스팅** <br>
[springboot_basic_#8](./2022-12-29-springboot_basic_8.md)

## 스프링 통합 테스트
스프링 컨테이너와 DB까지 연결한 통합 테스트, 웹을 켤 필요 없이 확인할 수 있음
_test/service/MemberServiceIntTest.class_ 생성

_@SpringBootTest, @Transactional_ 어노테이션 생성(클래스)
테스트 코드는 제일 편한 방법을 이용하면 됨(기존 코드는 Injection하면 좋음)

![Test 클래스 이미지](/assets/images/2022-12-29-10-50-02.png)
<pre>
@SpringBootTest: 스프링 컨테이너와 함께 실행
​@Transactional로 인해서 @BeforeEach, @AfterEach 어노테이션 필요없음
→ 테스트는 반복할 수 있어야 함,
  데이터베이스는 기본적으로 Transactional 개념이 있음,
  commit 하기 전에 반영이 되지 않음

@Transactional는 테스트 시작 전에 트랜잭션을 실행하고 
테스트가 끝나면 롤백을 해주는 기능이 있음

→ <U>DB에 실제 데이터가 반영이 되지 않기에 반복적으로 테스트가 가능하게 됨</U>
</pre>

​

> 스프링 없이 하는 테스트(단위 테스트 = 순수하게 자바 코드로만 하는 테스트)
> 
> 진짜 좋은 테스트는 단위 테스트를 잘 만든 테스트 

​---

​통합 테스트 코드
```java
package com.hello.hellospring.service;

import com.hello.hellospring.domain.Member;
import com.hello.hellospring.repository.MemberRepository;
import com.hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertThrows;

@SpringBootTest
@Transactional
class MemberServiceIntgTest {

    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    void join() {

        //given
        Member member = new Member();
        member.setName("spring");
        //when
        Long saveId = memberService.join(member);

        //then
        Member findmember = memberService.findOne(saveId).get();
        assertThat(member.getName()).isEqualTo(findmember.getName());
    }

    @Test
    public void 중복_회원_예외(){
        //given
        Member member1 = new Member();
        member1.setName("spring");

        Member member2 = new Member();
        member2.setName("spring");
        //when
        memberService.join(member1);
        //
        /*
         * assertThrows(예외가 발생해야 하는 클래스, () -> 검증할 로직) 람다형식
         * 오른쪽 로직을 실행이 될 때, 왼쪽 예외가 발생해야 하는 문법
         * */
        IllegalStateException e = assertThrows(IllegalStateException.class, () -> memberService.join(member2));
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 아이디 입니다");
    /*    try {
            memberService.join(member1);
            fail();
        }catch (IllegalStateException e){
            assertThat(e.getMessage()).isEqualTo("이미 존재하는 아이디 입니다");
        }*/
        //then
    }
}
```
<pre>
테스트 돌릴 때 유저네임과 패스원드 관련한 에러가 나타나는데 
이때, _application.properties_ 에 username과 관련된 코드 작성하면 해결
</pre>
![에러메세지](/assets/images/2022-12-29-11-00-06.png)
![에러 해결 방법](/assets/images/2022-12-29-11-00-27.png)
`비밀번호는 설정을 안해놨기에 비밀번호 관련된 코드는 추가X`


## JdbcTemplate 설정

repository 디렉토리에 클래스 생성
경로: _com.hello.hellospring/repository/JdbcTemplateMemberRepository_

![참고 이미지1](/assets/images/2022-12-29-11-06-41.png)
jdbcTemplate 쿼리를 날리고 결과를 로우 맵퍼를 통해 맵핑을 하고 리스트를 옵셔널로 바꿔서 반환

<pre>
JdbcTemplateMemberReposritory가 완성이 되면 
SpringConfig에 @Bean으로 설정된 MemberRepository 안에 
JdbcTemplateMemberReposritory(dataSource) 생성
</pre>
![참고 이미지2](/assets/images/2022-12-29-11-08-20.png)


_JdbcTemplateMemberRepository_
```java
package com.hello.hellospring.repository;

import com.hello.hellospring.domain.Member;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;

import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;

public class JdbcTemplateMemberRepository implements MemberRepository {

    private final JdbcTemplate jdbcTemplate;

    @Autowired //생성자가 딱 하나면 AutoWired 생략 가능
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    /**
     * 회원 저장
     * */
    @Override
    public Member save(Member member) {
        // SimpleJdbcInsert 가 테이블명과 GeneratedKey가 있으면 알아서 insert
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        // excuteAndReturnKey 에서 파라미터 받고 셋팅해줌
        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        member.setId(key.longValue());

        return member;
    }
    /**
     * 조회를 위한 쿼리
     * */
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }

    @Override
    public Optional<Member> findByName(String name) {

        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member ", memberRowMapper());
    }
    /**
     *조회를 위한 로우넘버, 객체 생성에 관한 콜백
     * */
    private RowMapper<Member> memberRowMapper(){
        return (rs, rowNum) -> {

            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}
```
`RowMapper 람다 바꾸기 전 코드`
```java
 private RowMapper<Member> memberRowMapper(){
        return new RowMapper<Member>() {
            
            @Override
            public Member mapRow(ResultSet rs, int rowNum) throws SQLException {

                Member member = new Member();
                member.setId(rs.getLong("id"));
                member.setName(rs.getString("name"));
                return member;
            }
        };
    }
```