---
layout: post
title: JPA n+1 problem
date: 2021-02-21
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA, JPA]
author: Jihoon Na # Add name author (required-fixed)
author-pic: jh.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: naji0630@gmail.com # email (optiona-fixed)
---

# Introduction
이전 석규님의 JPA 설명을 통해 JPA가 무엇이고 어떤 역할을 해주는 지에 대해서 보았다. JPA에 대한 자세한 설명은 이전의 포스팅 참조 부탁드립니다. [JPA란?](https://liketech.codes/jpa-primitive-value-mapping/)
오늘은 JPA에서 발생할 수 있는 문제인 N+1 문제에 대해서 간략히 알아보려고 합니다. JPA를 통해서 많은 코드 작성을 하지 않아도 RDBS를 객체 지향적으로
사용할 수 있도록 도와주지만, 장점이 있는 만큼 세심하게 사용하지 않으면 비효율적인 동작을 할 수 있습니다. 그 대표적인 예가 N+1문제 입니다.

### 다시 또 다시... ###
앞선 내용에서 JPA를 사용하면 "관계형" 데이터베이스를 "객체지향적"으로 사용할 수 있다는 것을 숙지하였다.
다소 지루하고 반복되는 내용일 수 있으나 리뷰한다는 차원에서 한번 더 언급하고자 한다.
<br>
<br>
더 강해져야하기 때문에.. 아주 복잡한 테이블 예제로 이야기를 이어나가고자 한다. 아래와 같은 복잡한 테이블이 있다고 가정하자. 이때 정보를 찾아가는 측면에서
![다이어그램](./assets/img/2021-02-20-N+1-diagram.png)
![테이](./assets/img/2021-02-20-N+1-diagram.png)

관계형 데이터베이스와 객체 차이를 보인다.

관계형 데이터베이스에서 "나"가 속한 팀을 조회하라.
   {% highlight sql %}
   SELECT * FROM MEMBER LEFT JOIN TEAM ON MEMBER.team_id = TEAM.team_id
   {% endhighlight %}
   
객체에서 "나"가 속한 팀을 조회하라.
   {% highlight java %}
   Team team = new Team(1,"드림팀");
   Member member = new Member(2,"나");
   member.setTeam(team);
   
   member.getTeam();
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