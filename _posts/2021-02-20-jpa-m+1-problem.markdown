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
앞선 내용에서 JPA를 사용하면 `관계형` 데이터베이스를 `객체지향적`으로 사용할 수 있다는 것을 숙지하였습니다.
다소 지루하고 반복되는 내용일 수 있으나 리뷰한다는 차원에서 한번 더 언급하고자 합니다.
<br>
<br>
더 강해져야하기 때문에.. 아주 복잡한 테이블 예제로 이야기를 이어나가고자 합니다. 아래와 같은 복잡한 테이블이 있다고 가정하자. 이때 정보를 찾아가는 측면에서 관계형 데이터베이스와 객체 차이를 보입니다.
<br>
![다이어그램](/./assets/img/2021-02-20-N+1-diagram.png){: width="30%" height="30%"}
![테이](/./assets/img/2021-02-20-N+1-table.png){: width="10%" height="10%"}
<br>
관계형 데이터베이스에서 **나**가 속한 팀을 조회하라.
   {% highlight sql %}
   SELECT * FROM MEMBER LEFT JOIN TEAM ON MEMBER.team_id = TEAM.team_id
   {% endhighlight %}
   
객체에서 **나**가 속한 팀을 조회하라.
   {% highlight java %}
   Team team = new Team(1,"드림팀");
   Member member = new Member(2,"나");
   member.setTeam(team);
   
   member.getTeam();
   {% endhighlight %}
   
위의 두 예제에서 처럼 **관계형 데이터베이스**에서 아이디를 기준으로 조회를 하고 있고
**객체**에서는 참조를 따라가 member.getTeam()을 하는 방식으로 가져오고 있습니다. 
만약 member.getTeamId()로 아이디를 꺼내서 이를 통해 객체를 조회하는 방식은 매우 불편할 것입니다.
JPA는 이러한 차이를 알아서 보완해서 관계형 데이터베이스를 쓰기는 쓰지만 JAVA 코드에서는 객체지향적으로 
사용할 수 있도록 처리해주는 역할을 해줍니다. 앞선 포스팅에서 예제가 있지만 다시 보자면 단지
이렇게 객체를 정의해주면 관계형 데이터베이스에 있는 데이터를 객체지향적으로 다룰 수 있게 되는 것입니다.

{% highlight java %}

@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    @Column(name = "name")
    private String username;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name="team_id")
    private Team team;
}

{% endhighlight %}

{% highlight java %}

@Entity
public class Team {
    @Id
    @GeneratedValue
    @Column(name="team_id")
    private Long id;
    @Column
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}

{% endhighlight %}


이렇게만 선언해주면 **member.getTeam()** 으로 멤버로부터 팀을 조회할 수 있게 되는것이다.

내부적으로 쿼리를 생성하고 정보를 mapping해주는 과정을 거쳤기 때문인데, 이렇게 JPA를 사용하면 편리한 만큼 신경써줘야할 부분도 있다.
오늘 이야기하고자 하는 N+1 문제가 그 중에 하나이다.
