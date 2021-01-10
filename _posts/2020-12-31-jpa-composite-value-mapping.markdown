---
layout: post
title: jpa value mapping - 2 
date: 2020-12-31
img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, Spring Boot, JPA, MariaDB]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

### value type mapping ###

jpa value mapping -1 post에 이어지는 내용이다.
jpa value mapping에서는 value가 primitive 타입인 경우만 다뤘었다.
그렇다면 객체 타입도 column으로 mapping이 가능할까 하는 궁금증이 생긴다. 
결론부터 말하자면 가능하다. 참 잘 되어있다.  

이전부터 계속 사용해왔던 account 객체 예제를 보자.

{% highlight java %}
public class Account {
  ...

  private String street;

  private String city;

  private String state;
 
  private String zipCode;

  ...
}
{% endhighlight %}

이 account entity를 보면 column으로 street, city, state, zipCode가 매핑될 것이다.
하지만, 우리는 저 정보들이 주소와 관련된 정보들이고 이를 한 class(이하 Address class) 로 묶어서 관리하고 싶은 욕구가 생긴다.
또한, 주소 관련 비지니스 로직은 Address class에 있는 것이 훨씬 설득력있는 코드가 될 것 같다.

그래서 우리는 다음과 같이 코딩하고 싶어진다.  

{% highlight java %}
- Address.java
public class Address {
  ...

  private String street;

  private String city;

  private String state;
 
  private String zipCode;

  ...
}

- Account.java
public class Account{
  ...

  private Address address;

  ...
}
{% endhighlight %}

그렇다면 우리는 Account table에 어떻게 Address 정보들을 column으로 mapping할 수 있을까

Address Class에 @Embeddable annotation을 붙여주고
Account Class에는 Address 멤버 변수에 @Embedded annotation을 붙여주면 된다.
아주 이름을 잘 지어서 annotation만 봐도 어떤 느낌인 지 알 수 있다. 

@Column annotation처럼 @Embedded annotation은 붙이지 않아도 default로 붙는 것 같다.
하지만 Address Class의 @Embeddable annotation을 붙여주지 않는다면 hibernate는 Address 타입의 변수를 인식하지 못한다.

최종적으로 다음과 같다.
{% highlight java %}
- Address.java
@Embeddable // is required
public class Address {
  ...

  private String street;

  private String city;

  private String state;
 
  private String zipCode;

  ...
}

- Account.java
public class Account{
  ...

  @Embedded // is default
  private Address address;

  ...
}
{% endhighlight %}


이제껏 포스트를 하다보니 hibernate라는 단어를 많이 쓰는 데, 돌아보니 나도 정확한 정의, 의미를 잘 모르고 있다는 것을 알았다. 
다음 편에는 hibernate의 정의(?)랑 jpa와의 관계와 jpa, hibernate를 쓰면서 주의해야 할 점을 다뤄보려고 한다. 
