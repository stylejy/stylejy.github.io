---
layout: post
title: Daily design pattern : 전략 패턴 (strategy pattern)
date: 2021-02-21
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, JAVA, JPA, n+1]
author: Jihoon Na # Add name author (required-fixed)
author-pic: jh.jpg # Add author-pic (required-fixed)
about-author: 고수가 되고싶ㄷ.. # Add about-author (about 과 같음) (optional-fixed)
email: naji0630@gmail.com # email (optiona-fixed)
---

# Introduction
안녕하세요. 오늘은 전략패턴에 대해서 알아보도록 하겠습니다. 전략 패턴의 정의는 다음과 같습니다.
> 전략패턴에서는 알고리즘군을 정의하고 각각을 캡슐화하여 교환해서 사용할 수 있도록 만든다. 전략 패턴을 사용
> 하면 알고리즘을 사용하는 클라이언트와는 독립적으로 알고리즘을 변경할 수 있다. ref) head first design patterns

크흠... 무슨 이야기인지..? 사실 잘 감이 오지 않습니다. 

본격적인 이야기로 들어가보겠습니다.

# God of 객체지향

오늘은 심시티 컨셉의 프로그램을 개발하는 객체지향의 신이 되어보겠습니다. god of 객체지향 건셉으로 가보겠습니다.

<br>
주의!! 아래의 내용은 다소 성차별적인 내용이 적혀있을 수 있지만 단순히 예제를 위한 예제입니다. 남자도 화장하고 여자도 수염을 깎을 수 있습니다.
<br>

오늘은 인간이라는 것을 만들어봐야겠군.. 

먼저 재사용성을 위해 인간이라는 클래스를 만들고 그 클래스를 상속받아서 구체적인 인간을 창조해 내야겠어..

우선 Person 객체를 만들어서 상위 클래스로 두어야겠군..

{% highlight java %}
class Person {
    public void breath(){
        System.out.println("입과 코로 후하");
    }

    public void eat(){
        System.out.println("우걱우걱");
    }

    public void walk(){
        System.out.println("뚜벅뚜벅");
    }
}
{% endhighlight %}

좋아..

그러면 여자와 남자를 만들때 이 상위 클래스를 만들어 상속하고 각자 차이가 있는 부분을 별도로 구현해주어야겠군...

{% highlight java %}
class Woman extends Person{
}

class Man extends Person{
}

{% endhighlight %}

좋군 좋아 그럼 이제 인간세상을 구현해볼까~?

{% highlight java %}
public class main {
    public static void main(String[] args) {
        Person man = new Man();
        Person woman = new Woman();
        System.out.println("남자가 밥을 먹는다.");
        man.eat();
        System.out.println("남자가 걷는다.");
        man.walk();
        System.out.println("여자가 밥을먹는다.");
        woman.eat();
        System.out.println("여자가 걷는다.");
        woman.walk();
    }
}

{% endhighlight %}
좋군 좋아.. 엇 근데 아기를 추가 하고 싶은데 .. 어쩌지..?
일단 자바의 오버라이드를 쓰자..!
{% highlight java %}
public class Baby extends Person{
    @Override
    public void walk(){
        System.out.printf("아장아장");
    }
}
{% endhighlight %}

흠.. 그런데 아기는 우걱우걱 밥을 먹지 않고 쩝쩝 귀엽게 먹잖아?
그리고.. 아장아장 기어다니는 경우가 또 필요하잖아..? 
프로그램 실행 도중 술을 많이 먹은 만취한 사람들의 걷는 방법을 동적으로 뚜벅뚜벅에서 아장아장으로 바꿔야하는 경우가 있군!
하.. 생각보다 많은 경우에 걷는 방법이 다르기때문에 하나하나 오버라이드 해주는 것도 비효율적인 것 같고.. 어찌해야할꼬..?

이쯤되어 다시 보는 전략패턴의 정의!

> 전략패턴에서는 알고리즘군을 정의하고 각각을 캡슐화하여 교환해서 사용할 수 있도록 만든다. 전략 패턴을 사용
> 하면 알고리즘을 사용하는 클라이언트와는 독립적으로 알고리즘을 변경할 수 있다. ref) head first design patterns

알고리즘 군을 정의하고... 각각 캡슐화하여 교환해서 사용할 수 있도록 한다!?

그러니까.. 알고리즘 군을 정의하라는 것은 뭐 먹는 동작.. 걷는 동작 이러한 알고리즘 군을 정의하라는 건가!?
그럼 이런 것들을 캡슐화해서 교환해서 사용할 수 있도록 하란 그런말이군! 그래 좋아 가보자! let's 고!

<br>
추상 클래스로 접근해보겠어!

> 추상클래스란 인터페이스와 조금은 유사한 개념으로 인터페이스가 모든 메서드가 추상 메서드인 반면 추상클래스는 한개 이상의 추상 메서드를 갖는 것을 말한다. 즉 추상 메서드와
> 일반 메서드 모두를 가지는 클래스가 추상클래스 인 것이다.

내가 대체 왜 추상 클래스를 선택해야한다고 생각한거지??? 그것은 같은 것은 같이 묶고 변화하는 것은 변화하는 대로 그룹별로 정의하기 위해서!
아래의 예시를 보자 숨쉬는 방식은 모든 인간이 동일하기 때문에 같은 것은 같도록, 걷거나 먹는 방식은 사람별로 다르기 때문에 군별로 교체해서 사용할 수 있도록 하기 위함이다!

{% highlight java %}
public abstract class Person {

    EatBehavior eatBehavior;
    WalkBehavior walkBehavior;
    boolean isDrunken;

    public void setEatBehavior(EatBehavior eatBehavior) {
        this.eatBehavior = eatBehavior;
    }

    public void setWalkBehavior(WalkBehavior walkBehavior) {
        this.walkBehavior = walkBehavior;
    }

    public void breath(){
        System.out.printf("입과 코로 후하");
    }
    
    public void performEat(){
        eatBehavior.eat();
    }
    
    public void performWalk(){
        walkBehavior.walk();
    }
}
{% endhighlight %}

이처럼 변하지 않는 부분은 변하지 않도록 고정하고 변할 수 있고 다양해질 수 있는 부분은 그룹별로 묶어서 추상 메서드로 빼낸 것이다.
그럼 구체적으로 저 인터페이스를 규정해보겠습니다.

{% highlight java %}
public interface EatBehavior {
    public void eat();
}
{% endhighlight %}

{% highlight java %}
public interface WalkBehavior {
    public void walk();
}
{% endhighlight %}

그리고 실제 구현체들을 만들어보겠습니다.
{% highlight java %}
public class WalkWithStand implements WalkBehavior{
    @Override
    public void walk() {
        System.out.printf("뚜벅뚜벅");
    }
}
{% endhighlight %}

{% highlight java %}
public class WalkWithNoStand implements WalkBehavior{
    @Override
    public void walk() {
         System.out.printf("아장아장");
    }
}
{% endhighlight %}

{% highlight java %}
public class EatHard implements EatBehavior{
    @Override
    public void eat() {
        System.out.printf("우걱우걱");
    }
}
{% endhighlight %}

{% highlight java %}
public class EatHard implements EatBehavior{
    @Override
    public void eat() {
         System.out.printf("우걱우걱");
    }
}
{% endhighlight %}

그리고 이제는 이 **정의된 알고리즘 군을 갭슐화 한 것**을 실제 구현체에 넣어 사용해보도록 하겠습니다.

{% highlight java %}
class Man extends Person{
    public Man(){
         eatBehavior = new EatHard();
        walkBehavior = new WalkWithStand();
    }
}
{% endhighlight %}


{% highlight java %}
class Woman extends Person{
    public Woman(){
        eatBehavior = new EatHard();
        walkBehavior = new WalkWithStand();
    }
}
{% endhighlight %}

{% highlight java %}
public class Baby extends Person{
    public Baby(){
        eatBehavior = new EatCute();
        walkBehavior = new WalkWithNoStand();
    }
}
{% endhighlight %}

이제 이걸 마음껏 이용을 해볼까요?

{% highlight java %}
public class main {
    public static void main(String[] args) {
        Person man = new Man();
        Person woman = new Woman();
        Person baby = new Baby();
        System.out.println("남자가 밥을 먹는다.");
        man.performEat();
        System.out.println("남자가 걷는다.");
        man.performWalk();

        System.out.println("여자가 밥을먹는다.");
        woman.performEat();
        System.out.println("여자가 걷는다.");
        woman.performWalk();

        System.out.printf("아기가 밥을 먹는다.");
        baby.performEat();
        System.out.println("아기가 걷는다");
        baby.performWalk();

        //오랜만에 친구들을 만난다.
        man.isDrunken = true;

        if(man.isDrunken){
            man.setWalkBehavior(new WalkWithNoStand());
        }

        man.performWalk();
    }
}

{% endhighlight %}

이처럼 변화가 있는 부분에 대해서 그룹화해서 캡슐화하여 교환하여 사용할 수 있는 것을 보았습니다.
또한 동적으로 프로그램 수행 도중에 변경을 할 수 있도록 보았습니다. 위의 예제처럼 먹는 행동, 걷는 행동을
원하는 구현체로 조합해서 사용하는 것을 **구성(composition)** 이라고 하고 상속보다는 구성을 사용하는 것이 더 
유연한 시스템을 설계할 수 있도록 합니다.

>오늘의 내용 요약 : 전략 패턴은 알고리즘을 그룹화하여 교환하여 사용할 수 있도록 하는 패턴이다.
> 구체적으로 말하면 변경될 수 있는 부분들은 변경될 수 있는 부분끼리 그룹화하여 교환하여 사용할 수 있도록 하는 것을 의미한다.
> 그렇게 하기위해 단순히 상속을 이용하는 것이 아니라 구성을 활용하면 더 유연한 구조를 가져갈 수 있다.
