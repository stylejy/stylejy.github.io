---
layout: post
title: Daily design pattern - 관찰자 패턴 - observer pattern
date: 2021-02-23
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA, Strategy pattern]
author: Jihoon Na # Add name author (required-fixed)
author-pic: jh.jpg # Add author-pic (required-fixed)
about-author: 고수가 되고싶ㄷ.. # Add about-author (about 과 같음) (optional-fixed)
email: naji0630@gmail.com # email (optiona-fixed)
---

# Introduction
안녕하세요. 오늘은 관찰자 패턴에 대해서 알아보도록 하겠습니다. 관찰자 패턴의 정의는 다음과 같습니다.
> 관찰자 패턴에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다 의존성을 정의합니다. 
> ref) head first design patterns

오늘도 역시 바로 본론으로 들어가 보겠습니다. 오늘의 예제는 배달 앱 회사의 가게 팀에 입사한 신입사원의 예제입니다.

# Contents

배달 어플리케이션에서 가게 정보라는 것은 가장 중심이 되는 정보입니다. 그만큼 여러팀에서 그 정보를 받아가 쓸 것입니다.

신입사원 이친절 씨가 관리하는 상점 객체는 다음과 같습니다.
{% highlight java %}
public class Shop {
    private Long id;
    private String name;

    public Shop(Long id){
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
{% endhighlight %}

<center>똑똑.</center>

{% highlight github %}
 이친절 사원(가게팀) : 안녕하세요. 가게 팀의 이친절입니다.
 김경력 대리(선물하기팀) : 안녕하세요. 김경력입니다.
 이친절 사원(가게팀) : 네. 혹시 무슨일로 오셨죠?
 김경력 대리(선물하기팀) : 아 가게 이름이 변경 되어도 저희 페이지에 갱신이 되지 않는 이슈가 있었습니다.
 이친절 사원(가게팀) : 아..! 선물하기 팀 쪽에서는 가게 이름이 변경된 걸 알 수가 없군요!
 김경력 대리(선물하기팀) : 네.. 그렇죠.. 그렇다고 1분에 한번씩 변경되었는지 조회해 볼 수도 없는 노릇이고..
 이친절 사원(가게팀) : 그럼요 그럼요!!! 맞죠..!! 당연히 그쪽에서는 그 타이밍을 알 수가 없죠!
 (마음속에서 이친절 사원의 친절 에너지가 넘처오른다.)
 이친절 사원(가게팀) : 당연히 저희 정보니까 저희가 변동 사항이 생길때마다 선물하기 팀쪽 정보를 직접 변경해드리겠습니다!
 김경력 대리(선물하기팀) : 그래요..? 뭐 그러면 저희야 좋죠. 선물하기 관련 가게정보 변경 api는 giftInfoChange()입니다.
 이친절 사원(가게팀) : 네! 알겠습니다! 그렇게 진행하겠습니다.
 김경력 대리(선물하기팀) : 젊은 친구가 아주 적극적이고 친절하군요. 그럼 그렇게 알고 저희는 돌아가겠습니다.
{% endhighlight %}

이친절 사원은 전달받은 api를 기반으로 열심히 코드를 작성해 나갑니다.

이친절 사원은 상점 이름이 변경될 때 전달받은 api를 호출하도록 코드를 변경하였습니다.

{% highlight java %}
public class Shop {
    private Long id;
    private String name;

    public Shop(Long id){
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
        giftInfoChange(name);
    }
}

{% endhighlight %}
{% highlight github %}

이친절 사원(가게팀) : 말씀해주신 api 연동하였습니다. 이제 선물하기 페이지에도 가게 이름 변동사항이 반영될겁니다!
김경력 대리(선물하기팀) : 오~ 고마워요~
{% endhighlight %}

<center>똑똑.</center>

{% highlight github %}
이친절 사원(가게팀) : 안녕하세요. 가게 팀의 이친절입니다.
박예민 대리(쿠폰팀) : 네.. 저희 페이지에 왜 가게 이름 변경사항이 반영되지 않는거죠?
이친절 사원(가게팀) : 아.. 그게 이미 파악하고 있는 이슈입니다. 관련 api를 주시면 저희가 추가하겠습니다.
박예민 대리(쿠폰팀) : 네. api는 setCouponShop()이에요. 근데, 인자로 이전 이름도 넣어주셔야 해요.
이친절 사원(가게팀) : 네! 알겠습니다!
{% endhighlight %}

<center>쿵.</center> 


{% highlight java %}
public class Shop {
    private Long id;
    private String name;

    public Shop(Long id){
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        setCouponShop(this.name, name);
        this.name = name;
        giftInfoChange(name);
    }
}

{% endhighlight %}

이친절 사원은 뭔가 자신의 코드 안에 다른 모듈의 api가 잔뜩들어가는 것을 보며 불안한 마음을 느끼기 시작합니다.

<center>똑똑.</center>
{% highlight github %}
이친절 사원(가게팀) : 안녕하세요. 가게 팀의 이친절입니다.
박친절 대리(예약팀) : 네! 안녕하세요! 저희 페이지에.. 
이친절 사원(가게팀) : 아.. 혹시 변경된 가게 이름이 뜨지 않나요? api를 알려주시면 저희가 호출해드리겠습니다!
박친절 대리(예약팀) : 아 정말 감사합니다! 근데 저희는 예약이라.. 시간에 대한 정보도 같이 넣어주셔야해요.
이친절 사원(가게팀) : 시간이요..? 네...네! 알겠습니다!
{% endhighlight %}

{% highlight java %}
public class Shop {
    private Long id;
    private String name;

    public Shop(Long id){
        this.id = id;
    }

    public Long getId() {
        return id;
    }

    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        setCouponShop(this.name, name);
        this.name = name;
        giftInfoChange(name);
        changeReservationInfo(new LocalDateTime());
    }
}

{% endhighlight %}

이친절 사원은 이렇게 다른 모듈의 코드를 호출해주는 것이 맞는지.. 그리고 그것에 맞는 인자들을 각각 자세히 알고 넣어주는게 맞는지에 대한 고민에 빠지기 시작합니다.
<br>
<center>며칠 후..</center>

<br>
<br>
<center>똑똑.</center>
{% highlight github %}
이친절 사원(가게팀) : 안녕하세요. 가게 팀의 이친절입니다.
최운영 대리(운영팀) : 안녕하세요. 지금 운영 환경에서 가게 이름이 변경되질 않고 있어요..
이친절 사원(가게팀) : 네? 무슨일이죠?? 무슨 에러가 뜨나요? 
최운영 대리(운영팀) : changeReservationInfo 인자가 잘못되었다고 뜨고있어요.. 
이친절 사원(가게팀) : 아.. 예약팀에 알아보겠습니다! (전화를 건다) 
박친절 사원(예약팀) : 안녕하세요. 무슨일이시죠? 
이친절 사원(가게팀) : 지금 예약 쪽 api에서 에러가 나서 가게 이름 변경이 되질않고 있어요.. 
박친절 사원(예약팀) : 어..! 죄송합니다.. 저희가 api가 변경되었는데, 전달 드린다고 한게.. 잊고있었네요. 
이친절 사원(가게팀) : 아...
{% endhighlight %}

이친절 사원은 다시는 친절하지 않기로 맹세하고 디자인 패턴 책을 꺼내들었습니다.

> 느슨한 결합 : 객체의 느슨한 결합이란 상호작용을 하는 두 객체가 서로에 대해 잘 알지 못하는 것을 의미합니다.

위의 예제에서는 가게팀에서 다른 모듈의 객체에 대해 **너무 자세히 알고 동작**을 하도록 구현하다보니 변경에 매우 취약한 모습을 보였습니다.

이를 해결하기 위해 옵져버 패턴을 이용하여 느슨한 결합(Loose coupling)으로 구현해보겠습니다.

먼저 가게 이름 변경 이벤트를 관찰할 객체에서 구현할 인터페이스를 만듭니다.

{% highlight java %}
public interface Observer {
    void update(String name);
}
{% endhighlight %}

이제 가게 모듈에서 각 모듈의 api를 호출해주는 것이 아니라 가게 이름 변경 이벤트를 받아볼 모듈에서
Observer 인터페이스를 구현하도록 변경하였습니다.

{% highlight github %}
이친절 사원(가게팀) : 안녕하세요.이친절입니다. 이제 저희 정보를 받으시려면 Observer 인터페이스를 구현하고 등록해주셔야 합니다.
{% endhighlight %}

이처럼 변화가 있는 부분에 대해서 그룹화해서 캡슐화하여 교환하여 사용할 수 있는 것을 보았습니다.
이를 통해 유연성있는 구조가 되었고 동적으로 프로그램 수행 도중에 변경을 할 수 있도록 구조가 바뀐 것을 보았습니다. 위의 예제처럼 먹는 행동, 걷는 행동을
원하는 구현체로 조합해서 사용하는 것을 **구성(composition)** 이라고 하고 상속보다는 구성을 사용하는 것이 더 
유연한 시스템을 설계할 수 있도록 한다는 것을 보았습니다.

>오늘의 내용 요약 : 전략 패턴은 알고리즘을 그룹화하여 교환하여 사용할 수 있도록 하는 패턴이다.
> 구체적으로 말하면 변경될 수 있는 부분들은 변경될 수 있는 부분끼리 그룹화하여 교환하여 사용할 수 있도록 하는 것을 의미한다.
> 그렇게 하기위해 단순히 상속을 이용하는 것이 아니라 구성을 활용하면 더 유연한 구조를 가져갈 수 있다.
