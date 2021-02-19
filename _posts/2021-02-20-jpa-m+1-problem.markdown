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
![테이블](/./assets/img/2021-02-20-N+1-table.png){: width="20%" height="20%"}
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
오늘 이야기하고자 하는 N+1 문제가 그 중에 하나입니다.

위에 작성한 코드를 기준으로 모든 멤버를 조회하는 테스트 코드를 작성해보도록 하겠습니다.

{% highlight java %}
@SpringBootTest
@Transactional
public class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;
    @Test
    public void mPlusOneProblemTest(){
        List<Member> memberList = memberRepository.findAll();
    }
}
{% endhighlight %}

이를 통해 모든 멤버를 조회하면서 실제 사용하는 쿼리 기록을 확인해보면 다음과 같습니다.

```
Hibernate:
    select
    member0_.member_id as member_i1_0_,
    member0_.team_id as team_id3_0_,
    member0_.name as name2_0_
    from
    member member0_
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
```
>db에 있는 멤버 수 만큼 조회를 한다고 ?!

그렇습니다.. 앞서 말씀드린 것 처럼 관계형 데이터베이스에서 조회하고자 하는 데이터들의 관계를 파악하여
객체들에 채워 넣어주기 위해 이처럼 멤버를 조회하고 그 멤버가 속해있는 팀에 대한 정보를 채워주기 위해 다음과 같은 일이 벌어지는 것입니다.
<br>
<br>

사실 많은 상황에서 우리는 모든 데이터의 팀 정보가 필요하지 않을 수 있습니다. 따라서 읽어온 모든 멤버의 팀 정보를 바로 읽어올 것이 아니라
천천히 필요할 때 읽어와도 된다는 생각이 들 수 있습니다. JPA에서는 이러한 것을 지원하기 위해 지연로딩을 제공하고 있습니다.
지연로딩을 하는 방법은 간단합니다. 앞서 정의한 엔티티 클래스에서 지연로딩 옵션 `FetchType.LAZY`만 추가해주면 됩니다.

{% highlight java %}
@Entity
public class Member {
@Id @GeneratedValue
@Column(name = "member_id")
private Long id;

    @Column(name = "name")
    private String username;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="team_id")
    private Team team;

    public Member(String name){
        this.username = name;
    }
}
{% endhighlight %}

이렇게 작성하니 아래와 같이 마법처럼 멤버에 대한 내용만 한번 조회합니다.
```
Hibernate:
    select
    member0_.member_id as member_i1_0_,
    member0_.team_id as team_id3_0_,
    member0_.name as name2_0_
    from
    member member0_
```
그럼 멤버 중 몇 명의 팀을 조회해보도록 할까요?

{% highlight java %}
@SpringBootTest
@Transactional
public class MemberRepositoryTest {

    @Autowired MemberRepository memberRepository;
    @Test
    public void mPlusOneProblemTest(){
        List<Member> memberList = memberRepository.findAll();
        String team1 = memberList.get(0).getTeam().getName();
        String team2 = memberList.get(2).getTeam().getName();

    }
}
{% endhighlight %}
```
Hibernate:
    select
    member0_.member_id as member_i1_0_,
    member0_.team_id as team_id3_0_,
    member0_.name as name2_0_
    from
    member member0_
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
Hibernate:
    select
    team0_.team_id as team_id1_1_0_,
    team0_.name as name2_1_0_
    from
    team team0_
    where
    team0_.team_id=?
```

보시는 것 처럼 내가 더 자세히 들여다보고자 하는 타이밍에 jpa에서 해당 멤버의 팀 정보를 읽어오고 있습니다.
이처럼 간단하게 지연 로딩을 통해서 n+1문제를 해결할 수 있습니다. 하지만 정말로 모든 멤버들의 팀을 알고 싶은 경우라면 어떻게하죠?

>이럴때는 fetch join을 통해서 해결할 수 있습니다.

fetch join은 JpaRepository를 상속받아서 만든 repository 인터페이스에 다음과 같이 선언해서 사용할 수 있습니다.
{% highlight java %}
public interface MemberRepository extends JpaRepository<Member,Long> {
    @Query("select p from Member p left join fetch p.team")
    List<Member> findAllWithFetchJoin();
}
{% endhighlight %}

이렇게 선언했을때 쿼리 로그를 보도록하겠습니다.

```
Hibernate: 
    select
        member0_.member_id as member_i1_0_0_,
        team1_.team_id as team_id1_1_1_,
        member0_.team_id as team_id3_0_0_,
        member0_.name as name2_0_0_,
        team1_.name as name2_1_1_ 
    from
        member member0_ 
    left outer join
        team team1_ 
            on member0_.team_id=team1_.team_id
```

이처럼 한번에 조인을 걸어 데이터를 받아와 객체에 넣어주는 것을 확인할 수 있습니다.

>오늘의 내용 요약 : JPA를 사용하면 정말 편리하지만 세심하게 들여다봐야하는 n+1 문제 같은 것들이 있습니다. 이는 Lazy loading이나 fetch join을 통해 해결할 수 있습니다.

수고하셨습니다!