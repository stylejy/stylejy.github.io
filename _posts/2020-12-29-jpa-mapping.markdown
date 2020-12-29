---
layout: post
title: jpa mapping 
date: 2020-12-29
img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, Spring Boot, JPA, MariaDB]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

jpa는 객체에 대한 정보를 database에 mapping할 수 있는 여러가지 기술들을 제공하고 있다.

### entity type mapping ### 
entity 객체는 database에서 table에 mapping시킬 수 있다. 
@Entity annotation을 사용하면 hibernate에게 이 객체를 프로젝트에 연결된 database에 table로 mapping정보를 준다. 

-Account.java
{%highlight java %}
@Entity
public class Account {
  ...
}
{% endhighlight %}

Entity 객체 Account를 만들고
Runner를 다음과 같이 만든다. 

-JpaRunner.java
{%highlight java %}
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {
  @PersistenceContext
  EntityManager entityManager;

  @Override
  ublic void run(ApplicationArguments args) throws Exception {
    Account account = new Account();
    ...
    entityManager.persist(account);
  }
}
{% endhighlight %}

이 Runner함수를 실행시키고 연결된 database에서 확인하면
account table에 생성된 것을 확인할 수 있다. 

참고로, 여기서 EntityManager는 jpa에서 매우 핵심적인 기능을 하는 클래스이다.
마치 spring framework에서 application context와 같다고 생각하면 좋다.


