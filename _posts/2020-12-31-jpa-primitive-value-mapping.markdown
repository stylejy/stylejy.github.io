---
layout: post
title: jpa value mapping - 1 
date: 2020-12-31
img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, Spring Boot, JPA, MariaDB]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

jpa-entity-mapping post에 이어지는 내용이다.
객체에서 entity type은 database에 table 개념으로 mapping되는 개념이다.
객체에서 value type은 database에 column 개념으로 mapping되는 개념이다. 

### value type mapping ###
@Column annotation을 붙이면 한 table의 한 column으로 mapping된다.

-Account.java
{% highlight java %}
@Entity
public class Account {
  ...

  @Column
  private Long id;

  @Column
  private String username;

  ...
}
{% endhighlight %}

하지만, 사실 @Entity annotation을 domain class에 붙이면 class의 모든 member variables에는 default로 @Column annotation이 붙어진다.
즉, 모든 member variables는 database의 column으로 mapping된다. 

{% highlight java %}
@Entity
public class Account {
  ...

  private Long id;

  private String username;

  ...
}
{% endhighlight %}

이렇게 써도 똑같은 의미이다. 자신의 코드 스타일에 따른 선택 사항이다.


domain class에 database column으로 mapping되는 것을 원하지 않고 비지니스 로직만을 위한 변수가 필요한 경우가 있다.
@Transient annotation을 사용하면 된다. 

{% highlight java %}
@Entity
public class Account {
  ...

  @Transient
  private String noColumn;

  ...
}
{% endhighlight %}

Transient라는 것은 jpa가 관리하는 객체 상태와 관련된 개념인데, 이건 추후에 다뤄보자.


알다시피 database에 여러가지 제약 조건들이 있다. 예를 들면, not null이나 primary key 같은.
당연히, jpa에서 그 제약 조건들도 걸 수 있다. 

{% highlight java %}
public class Account {
  ...

  /**
  * @Id를 사용하면 primary key 제약 조건을 걸 수 있다. 
  */
  @Id
  @GeneratedValue
  private Long id;
  
  /**
  * @Column property로 nullable 속성을 줄 수 있다. 
  */
  @Column(nullable = false)
  private String username;
  
  ...
}
{% endhighlight %}

이런 것들은 빙산의 일각이다. 이외에도 다른 많은 제약 조건들을 지원하는 기능들이 있다.
같이 찾아서 사용해보자.