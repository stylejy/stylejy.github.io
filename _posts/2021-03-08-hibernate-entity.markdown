---
layout: post
title: entity-state
date: 2021-03-08
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA, Spring-boot, Hibernate]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

이전 글에 대해서는 hibernate에 대해 알아보았다.  
특히, jpa의 EntityManager를 hibernate의 구현 방식을 통해 hibernate가 jpa를 어떤 방식으로 구현하는 지 짐작할 수 있었다.  
이번 블로그에서는 hibernate가 entity 객체들을 어떻게 관리하는 지를 알아보자. 
> Entity란 database의 table과 비슷하다고 생각할 수 있다.   
> database의 table의 패러다임을 객체 패러다임으로 옮겨온 개념이다. 

### entity state

entity에는 4가지 상태가 있다.  
> Transient, Persistence, Detached, Removed  

간단히 설명하면 아래와 같다.  
1. Transient: hibernate가 모르는 상태  
2. Persistence: hibernate가 관리 중인 상태  
3. Detached: hibernate가 더 이상 관리하지 않는 상태  
4. Removed: hibernate가 관리하지만 삭제하기로 한 상태  

예시와 같이 보자. 다음과 같은 Entity 클래스가 있다고 생각해보자.  

-- Example.java
{% highlight java %}
@Entity
public class Example {

    ...

    @Id
    @GeneratedValue
    private Long id;


    private String name;
    private String nick;
    
    ...
}
{% endhighlight %}
> 아무 생각없이 @GeneratedValue를 아무 조건없이 String 타입의 변수에 적용했더니 "Unknown integral data type for ids : java.lang.String" 에러가 발생했다.  
> @GeneratedValue에 조건으로 generator를 준다면, String이어도 value를 자동 생성할 수 있다.  
> [참고링크](https://stackoverflow.com/questions/40177865/hibernate-unknown-integral-data-type-for-ids) 

-- ApplicationRunner.java
{% highlight java %}
    ...
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        // Transient
        Example example = new Example();
        example.setName("seokkyu");
        example.setNick("screw");
    }

    ...
{% endhighlight %}

위와 같이 그냥 example객체를 만들고 아무것도 안해준다면, 당연히 아무 일도 일어나지 않는다. 이를 hibernate가 관리하지 않는 상태인 Transient라고 한다.  
<br>

> jpa value primitive type mapping에서 Entity 클래스 멤버변수에 @Transient를 사용하면 hibernate는 이 변수를 인지하지 않고 table을 생성한다고 했었었다. 비슷한 개념으로 이해하면 좋을 것 같다.  
> [참고링크](https://liketech.codes/jpa-primitive-value-mapping/)

<br>
이제 나머지 상태를 확인해보자.  
우선 나는 jpa의 EntityManager를 이용하지 않고, Spring이 주입해주는 EntityManager에서 hibernate의 구현체인 Session을 꺼내서 예제를 작성할 것이다.


다음과 같이 작성할 수 있다. 

{% highlight java %}

    ...
    //어떻게 @PersistContext만 붙여도 
    //우리가 factory객체에서 꺼내지 않고도 주입받을 수 있는 걸까.
    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        // Transient
        Example example = new Example();
        example.setName("seokkyu");
        example.setNick("screw");


        Session session = entityManager.unwrap(Session.class);

        // Persistence
        session.save(example);
    }
    ...

{% endhighlight %}

위의 방식으로 session 객체를 꺼낼 수 있다. 
> 바로 이전 [블로그](https://liketech.codes/jpa-primitive-value-mapping/) 에서 코드를 까며 확인했던 것을 (hibernate가 어떻게 jpa를 구현했는지) 다시 한 번 확인할 수 있는 순간이다. 

<br>
session.save() 함수를 이용해서 example객체를 hibernate가 인지하고 관리할 수 있는 상태로 만들 수 있다. 이를 Persistence상태라고 한다.
그렇다면 hibernate가 관리를 한다는 의미는 뭘까?

다음의 코드를 보자.

{% highlight java %}
    ...
    
    Example example = new Example(); // Transient
    example.setName("seokkyu");
    example.setNick("screw");


    Session session = entityManager.unwrap(Session.class);

    
    session.save(example); // Persistence
    example.setName("screw_new");

    ...
{% endhighlight %}

session에 save를 한 후에 example에 이름을 바꾸었다.  
다음의 쿼리 기록을 보면 우리는 session.update()를 하지 않았는 데도 update query가 날아간 것을 볼 수 있다. 
<br>

```
Hibernate: 
    select
        nextval(hibernate_sequence)

Hibernate: 
    insert 
    into
        example
        (name, nick, id) 
    values
        (?, ?, ?)

Hibernate: 
    update
        example 
    set
        name=?,
        nick=? 
    where
        id=?

```
<br>

다음의 데이터베이스 테이블 상태를 봐도 이름이 screw_new로 되어있는 것을 확인할 수 있다. <br>
![table](/./assets/img/hibernate-entity-persistence.JPG){: width="30%" height="30%"}

그렇다면 hibernate가 관리하지 않는 상태라는 Detached 상태일 때는 어떻게 될까?

{% highlight java %}
    ...
    
    session.save(example); // Persistence
    session.flush();

    session.evict(example); // Detached
    example.setName("screw2");
    ...
{% endhighlight %}

> Q. 갑자기 flush()는 왜 한 걸까?  

```
Hibernate: 
    select
        nextval(hibernate_sequence)

Hibernate: 
    insert 
    into
        example
        (name, nick, id) 
    values
        (?, ?, ?)

```
![table](/./assets/img/hibernate-entity-detached.JPG){: width="30%" height="30%"}

hibernate가 update를 치지 않는다는 것을 알 수 있다. 이름도 처음 저장할 때 seokkyu로 되어있다.  

>참고로, 다시 persistence 상태로 만들고 database에 새로운 값으로 업데이트 치고 싶다면, session.update() 함수를 사용하면 된다. 

<br>
Removed 상태도 써야 하는데, 나는 개인적으로 한 블로그의 글이 길어지는 것을 선호하지 않기 때문에, 이 내용은 다음 블로그에서 이어서 쓰려고 한다.  
그리고 추가로, flush() 함수는 무엇이고, 왜 flush() 함수를 호출했는 지도 같이 쓸 것이다. 

