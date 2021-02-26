---
layout: post
title: hibernate
date: 2021-02-26
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA, Spring-boot, Hibernate]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

이전 글에 대해서는 jpa에 대해 알아보았다.  
오늘은 jpa를 구현하는 framework 중 hibernate에 대해 알아보자.  

보통은 database에 transaction을 처리하기 위해 하는 일련의 과정들을 jpa가 한다라고 표현하는 것 같다.  
하지만, 적어도 이 글에서는 jpa와 hibernate를 엄격히 분리해서 이야기해보려고 한다. 
> jpa는 orm 기술에 대한 api 표준 명세서 역할이고 
> hibernate는 그런 명세서를 따르는 실제로 일처리를 하는 역할이다. 
> 이렇게 분리해둔 이유가 궁금하신 분들에게 [JPA](https://liketech.codes/jpa/) 블로그가 참고가 되었으면 좋겠다. 


# EntityManager

EntityManager는 Entity들을 관리하는 jpa의 핵심 interface이다.
우선 실제로 어떻게 사용되는 지를 보자.  
쉽게 확인하기 위해 나는 spring boot를 활용할 것이다. spring boot에서 EntityManager를 다음과 같이 사용해 database에 저장할 수 있다.

-- Coupons.java
{% highlight java %}
@Entity
public class Coupons {

    ... 

    @Id
    @GeneratedValue
    private Long Id;

    private String couponnumber;
    private String username;

    ...
}
{% endhighlight %}
위와 같이 Entity class를 정의하였을 때, 

-- JpaRunner.java
{% highlight java %}
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @Autowired
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Coupons coupons = new Coupons();
        coupons.setCouponnumber("couponnumber");
        coupons.setUsername("seokkyu");
        entityManager.persist(coupons);
    }
}
{% endhighlight %}

이렇게 Autowired로 entityManager를 주입받아서 persist 함수를 사용하면 database에 저장할 수 있다.  
이런 식으로 Entity class 정보를 database에 저장하고 받아오는 과정을 entityManager를 통해서 할 수 있다.


# Session

우리는 jpa가 어떤 것인지 알고 있다. 단순한 명세서 뿐이라는 것을..  
그리고 다음 코드를 보면 EntityManager가 interface라는 것도 알 수 있다.  

-- EntityManager.class
{% highlight java %}
public interface EntityManager {
    ...
    void persist(Object var1);
    ...
}
{% endhighlight %}

나는 그렇다면 내가 실제로 주입받은 entityManager는 무엇이고, 그 기능이 구현된 것은 어디에 있을까 라는 궁금증이 생겼다. 
이는 이 entityManager를 구현한 클래스들을 따라가보니 알게되었다. 
Session interface가 EntityManager interface를 extends했고, SessionImplementor interface가 Session interface를 extends했고, 이 interface를 SessionImple class가 구현했다. 그리고 이 과정은 hibernate에서 이루어졌다. 

-- SessionImpl.class
{% highlight java %}
public class SessionImpl extends AbstractSessionImpl implements EventSource, SessionImplementor, HibernateEntityManagerImplementor {
    ...
    public void persist(Object object) throws HibernateException {
        this.checkOpen();
        this.firePersist(new PersistEvent((String)null, object, this));
    }
    ...
}
{% endhighlight %}

디버깅으로 확인한 결과 Autowired로 주입받은 entityManager의 persist함수를 호출하면 SessionImpl의 persist함수가 호출되는 것을 확인했다. 이로써, 우리는 hibernate가 jpa를 구현하고 있다라는 사실과 hibernate도 jpa를 직접 구현하고 있는 것이 아니라 여러 겹의 interface를 통해 구현하고 있다는 것을 알 수 있었다.  
