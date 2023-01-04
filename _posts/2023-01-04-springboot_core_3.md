---
layout: single
title: "SpringBoot - Core #03"
categories: springboot_core
Tag: [spring-core]
toc: true
sidebar:
  nav: "docs"
---

# 주문과 할인 도메인 설계/ 개발/ 테스트

* ## 주문과 할인 도메인 설계
  1. 회원은 상품 주문 가능
  2. 회원 등급에 따라 할인 정책 적용 가능
  3. 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인 적용(추후 변경 가능)
  4. 할인 정책 변경 가능성 높음. 회사의 기본 할인 정책을 못 정함, 오픈 직전까지 고민 미룸, 최악의 경우 할인 미적용(미확정)


![주문 도메인 협력, 역할, 책임 이미지](/assets/images/2023-01-04-15-07-26.png){: .full}

## 주문 도메인 전체(역할과 구현)

![주문 도메인 전체 이미지](/assets/images/2023-01-04-15-09-00.png){: .full}

**역할과 구현 분리해서** 자유롭게 구현 객체를 조립할 수 있게 설계.<br>(회원 저장소와 할인 정책 유연하게 변경 가능)

### 주문 도메인 클래스 다이어그램

![주문 도메인 클래스 다이어그램 이미지](/assets/images/2023-01-04-15-10-37.png){: .full}

**_주문 서비스 구현체: OrderServiceImpl_**

### 주문 도메인 객체 다이어그램 1

![주문 도메인 객체 다이어그램 1 이미지](/assets/images/2023-01-04-15-12-12.png){: .full}

### 주문 도메인 객체 다이어그램 2

![주문 도메인 객체 다이어그램 2 이미지](/assets/images/2023-01-04-15-12-42.png){: .full}

> 객체 다이어그램: 동적으로 연관 관계가 맺어지는 그림

<pre>
위의 그림처럼 저장소나 할인정책이 바뀌어도 
주문 서비스 구현체는 변경하지 않아도 됨

협력 관계를 그대로 재사용할 수 있게 설계함

→ 역할과 구현을 분리했기 때문에
</pre>

* ## 주문과 할인 도메인 개발

### 할인 정책 인터페이스

_discount package 생성_<br>
_(할인된 것에 관한 것을 리턴해주기 위해)_

```java
package hello.core.discount;

import hello.core.member.Member;

public interface DiscountPolicy {

    /**
     * @return 할인 대상 금액
     */

    int discount(Member member, int price);
}
```

### 정액 할인 정책 구현체

_천원 할인에 대한 곳_<br>
_(enum 타입은 == 쓰는게 맞음)_

```java
package hello.core.discount;

import hello.core.member.Grade;
import hello.core.member.Member;

public class FixDiscountPolicy implements DiscountPolicy {

    private int discountFixAmount = 1000; //1000원 할인

    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP) {//Vip는 1000원 할인
            return discountFixAmount;
        } else {
            return 0;
        }
    }
}
```

_vip를 위한 1000원 할인, 그 외는 할인 없음_

### 주문 엔티티

```java
package hello.core.order;

public class Order {

    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

    public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
        this.memberId = memberId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
        this.discountPrice = discountPrice;
    }
    //할인된 가격 계산 하는 메서드
    public int calculatePrice(){
        return itemPrice - discountPrice;
    }

    public Long getMemberId() {return memberId;}

    public void setMemberId(Long memberId) {this.memberId = memberId;}

    public String getItemName() {return itemName;}

    public void setItemName(String itemName) {this.itemName = itemName;}

    public int getItemPrice() {return itemPrice;}

    public void setItemPrice(int itemPrice) {this.itemPrice = itemPrice;}

    public int getDiscountPrice() {return discountPrice;}

    public void setDiscountPrice(int discountPrice) {this.discountPrice = discountPrice;}

    @Override
    public String toString() {
        return "Order{" +
                "memberId=" + memberId +
                ", itemName='" + itemName + '\'' +
                ", itemPrice=" + itemPrice +
                ", discountPrice=" + discountPrice +
                '}';
    }
}
```

### 주문 서비스 인터페이스

_주문 서비스의 역할 참조_

![주문 서비스 관계도](/assets/images/2023-01-04-15-29-11.png){: .full}

```java
package hello.core.order;

public interface OrderService {
    //주문 생성할 때 회원id, 상품명, 상품 가격을 파라미터로 넘기면 리턴으로 주문 결과 반환
    Order createOrder(Long memberId, String itemName, int itemPrice);

}
```

### 주문 서비스 구현체

_회원을 찾기 위한 MemberRepository, 할인 정책을 위한 DiscountPolicy를 구현체로 생성_

```java
package hello.core.order;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.Member;
import hello.core.member.MemberRepository;
import hello.core.member.MemoryMemberRepository;

public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {

        Member member = memberRepository.findById(memberId);
        int discountPrice = discountPolicy.discount(member, itemPrice); //최종 할인된 가격

        return new Order(memberId, itemName, itemPrice, discountPrice); //최종 생성된 주문
    }
}
```
<pre>
할인에 대한 것은 DiscountPolicy 의 책임

OrderServiceImpl 에서는 할인에 관한 것을 신경쓰지 않아도 됨 
  - 단일 책임의 원칙이 잘 지켜짐

주문 생성 요청이 오면, 회원 정보 조화, 할인 정책 적용 후에 주문 객체 생성해서 반환
</pre>

## 주문과 할인 도메인 실행과 테스트

### 실행

```java
package hello.core;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.order.Order;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class OrderApp {

    public static void main(String[] args) {
        MemberService memberService = new MemberServiceImpl();
        OrderService orderService = new OrderServiceImpl();

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 1000000);

        System.out.println("order = "+ order);
        System.out.println("calculatePrice = "+ order.calculatePrice());
    }
}
```
### 주문과 할인 정책 테스트

```java
package hello.core.order;

import hello.core.member.Grade;
import hello.core.member.Member;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;

public class OrderServiceTest {

    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder() {
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 1000000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
        Assertions.assertThat(order.calculatePrice()).isEqualTo(999000);
    }
}
```

> JUnit 테스트를 사용하는것이 바람직하다

> 포스팅에 참고한 강의 링크
>
> [김영한 - 스프링-핵심-원리-기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8)
