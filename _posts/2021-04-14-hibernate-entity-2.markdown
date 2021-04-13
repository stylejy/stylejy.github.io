---
layout: post
title: entity-state-2
date: 2021-04-14
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

[이전 블로그](https://liketech.codes/hibernate-entity/)에 이어서 우리는 두 가지의 이야기를 해볼 것이다.

첫 번째는, entity 상태 중에 removed에 대한 이야기이고,  
두 번째는, jpa가 제공하는 기능에 대한 이야기를 해보자. 

첫 번째로 removed entity 상태에 대해 이야기해보자.  

removed 상태의 entity란, 데이테베이스에서 삭제될 데이터라는 의미를 가진다. session.remove() 함수를 통해 데이터 베이스에서 데이터를 삭제할 수 있고, 당연히 enteiryManager에도 없다. 

```java
Session session = entityManager.unwrap(Session.class);
        
// Persistence
session.save(example);
Example example1 = session.get(Example.class, example.getId());

System.out.println("Before session remove function");
System.out.println("session contains entity is " + session.contains(example1));

// Removed
session.remove(example1);

System.out.println("After session remove function");
System.out.println("session contains entity is " + session.contains(example1));

```
다음은 hibernate가 날린 쿼리이다. 
```
Hibernate: 
    select
        nextval(hibernate_sequence)

Before session remove function
session contains entity is true

After session remove function
session contains entity is false

Hibernate: 
    insert 
    into
        example
        (name, nick, id) 
    values
        (?, ?, ?)

Hibernate: 
    delete 
    from
        example 
    where
        id=?        
```

delete query가 날라가고 session에 들어있지 않다는 것을 확인했다. hibernate는 더 이상 removed 상태의 entity를 관리하지 않는다.

<br>
두 번째로, hibernate가 제공해주는 기능은 어떤 것들이 있는 지 이야기해보자. 대표적인 기능 dirty-check, write-behind 두 가지를 이야기하려고 한다. 
dirty-check기능은 hibernate가 관리하는 entity라면 변경사항을 관찰함을 의미한다. 다음의 예시 코드를 보자! 

``` java

    Example example = new Example(); // Transient
    example.setName("seokkyu");
    example.setNick("screw");

    Session session = entityManager.unwrap(Session.class);
    
    session.save(example); // Persistence
    example.setName("screw_new");

```

현재 example entity의 상태는 persistence이다. 이 때, 우리는 session.update()함수를 사용해 update 쿼리를 날리지 않았음에도 update쿼리가 발생함을 알 수 있다. 

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

이렇듯, hibernate가 entity의 변경 사항을 인지하고, 그것을 DB에 반영해주는 기능이다.  
> 개인적으로 JPA가 가독성이 좋고, 확장성이 좋다라고 이야기 듣는 이유 중 하나라고 생각한다. 

다음은 write-behind 기능이다. 간단하게 말하자면, 쿼리를 최대한 필요할 때에 필요한 쿼리를 DB에 반영하는 기능이다.  
> 트랜잭션 내에서 트랜잭션 커밋 타이밍에 쿼리를 DB에 날린다. 

다음의 코드를 보자!
``` java
    Example example = new Example(); // Transient
    example.setName("seokkyu");
    example.setNick("screw");

    Session session = entityManager.unwrap(Session.class);

    System.out.println("======================");
    session.save(example); // Persistence
    System.out.println("======================");
```

이렇게 코드를 작성했다고 해보자.  
다음 hibernate가 쿼리를 날리는 시점을 보면 알겠지만 저 "================" 출력문 사이가 아니라 트랜잭션이 끝나는 시점에 날리는 것을 확인할 수 있다.  

```
======================
Hibernate: 
    select
        nextval(hibernate_sequence)
======================
Hibernate: 
    insert 
    into
        example
        (name, nick, id) 
    values
        (?, ?, ?)
```

사실 hibernate가 많은 기능을 해주지만, 두 가지만 당장 이 블로그에서 소개하는 것은 이 두 가지가 같이 사용되어 시너지 효과를 발생시킬 수 있는 경우를 보이기 위해서이다. 
따라오는 코드를 보자!

``` java

    Example example = new Example(); 
    example.setName("seokkyu");
    example.setNick("screw"); // nick = "screw"

    Session session = entityManager.unwrap(Session.class);
    
    session.save(example); 

    example.setNick("screw_New"); // nick = "screw_New"
    example.setNick("screw"); // nick = "screw"

```
이렇게 nick을 "screw_New"로 바꾸고, 또 처음 값과 똑같은 "screw"로 set을 하게 되었을 때, hibernate가 위의 2가지 기능을 제공하지 않았다면 update 쿼리가 2번이 생겨야 할 것이다. 하지만, 변경 사항을 감지하는 dirty-checking을 하고 쿼리를 최대한 늦게 DB에 날리기 때문에, 트랜잭션이 끝난 시점에서 변동사항이 없다고 보고 update 쿼리를 하나도 날리지 않는다.  

다음 hibernate가 날린 쿼리를 보면 insert 쿼리만 확인할 수 있다.  
> 단건 쿼리만 봤을 때는 쿼리를 직접 작성한 경우가 더 좋은 성능을 낼 수 있긴 하지만,  
> 이런 기능들 때문에 애플리케이션 전반에 두고 봤을 때보면 jpa도 좋은 성능을 낼 수 있다.   

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

이렇게 jpa에 관한 블로그들을 작성하면서 많은 것을 배웠고, 느낀바가 꽤 있었다.  
jpa가 러닝커브가 높지만 그 배우는 과정도 재밌고, 게다가 잘 사용하면 좋은 성능, 좋은 가독성, 좋은 확장성 등등 많은 이점을 가져갈 수 있다고 생각한다.  
> 우리 모두 jpa 잘 배워서 잘 써먹자!  
> jpa의 세계에 온 것을 환영한다!  
