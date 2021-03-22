---
layout: post
title: java, jpa code improvement
date: 2021-03-22
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Side Project, Spring Boot, JPA]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

개인적으로 기술이나 개념을 배웠으면, 실제로 코딩해보고 실행시켜보고 그 결과를 봐야지 진짜 내 것이 되는 첫 번째 발판이라고 생각한다.  
그런 측면에서 jpa, redis관련해서 side project를 하고 있는 것이 있다. side project를 하면서, 기록하고 싶은 부분이 생겨서 이 포스트를 작성하려고 한다.  
<br>
배경을 먼저 설명하면, users entity와 coupons entity가 있고, username과 password가 주어지면 coupons entity에서 해당하는 coupon 정보를 받아오는 코드가 변화를 기록하려고 한다.  
그러면서 잘못된 코드와 비교적 그나마 발전된 코드를 공유하려고 한다. 
<br>

```java
@PostMapping("/coupons")
@ResponseBody
public String getCoupons(@ModelAttribute Event event){
    String password = event.getPassword();
    String username = event.getUsername();

    ...
    Optional<Users> user = usersRepository.findByUsernameAndPassword(username, password);
    
    if(user.isPresent()) {
        Optional<Coupons> coupon = couponRepository.findById(user.get().getCoupons().getId());
        if(coupon.isPresent()) {
            return "yes";
        }
    }
    return "no";
}
```

users entity에서 username과 password에 대응되는 user id를 가져온다. 그리고 그 user id로 coupons 정보를 가져온다.  
<br>

개인적으로 Optional을 사용하고 싶었다. 그래서 위와 같이 적었지만, 사실 위와 같이 작성하는 것은 아래 코드와 같이 Optional을 쓰지 않는 것과 큰 차이가 없다. 오히려, Optional이라는 비싼 객체를 만듦으로써, 이점 없이 메모리만 더 사용하는 코드이다.  
<br>

```java
Users user = usersRepository.findByUsernameAndPassword(username, password);
if(user != null) {
    Coupons coupon = couponRepository.findById(user.getCoupons().getId());
    if(coupon != null) {
        return "yes";
    }
}

return "no";
```

그래서 나는 Optional에서 제공하는 API를 활용해보려고 한다. 그 중에서 orElseThrow를 활용하면 적절하겠다고 판단했다. 

```java
try {
    ...
    Optional<Users> user = usersRepository.findByUsernameAndPassword(username, password);
    Users _user = user.orElseThrow(() -> new NoSuchElementException("no"));
    Optional<Coupons> coupon = couponRepository.findById(_user.getCoupons().getId());
    Coupons _coupon = coupon.orElseThrow(() -> new NoSuchElementException("no"));
    ...
}catch(NoSuchElementException e){
    return e.getMessage();
}
return "yes";
```
Optional 안에 있는 객체가 null일 때는 무조건 exception을 throw하여서, try문 안에서는 user, coupon 객체가 null이 아니라고 생각하고 코딩을 했고, catch문 안에서는 null이라고 생각하고 코딩했다.  
그렇게 하니, 코딩을 하는 나도 로직을 짤 때 명확해지고, 코드를 리뷰를 할 때에도 더 가독성이 있어보인다.  
<br>
제일 좋은 건, 중첩 if문이 사라져서 편안해졌다.  

다음으로, 지훈님의 [이 블로그](https://liketech.codes/jpa-m+1-problem/)를 참고했다. 이 프로젝트를 진행하던 중에 이 글을 읽고, 많은 것을 배웠다. 그리고, 배운 것들을 실제 나의 프로젝트에 적용시켜보았다.  

-- EventController.java
```java
try {
    ...
    // 개선 전 --- 1번 
    Optional<Coupons> coupon = couponRepository.findById(_user.getCoupons().getId());
    // 개선 후 --- 2번
    Optional<Coupons> coupon = Optional.ofNullable(_user.getCoupons());

    ...
}catch(NoSuchElementException e){
    return e.getMessage();
}
return "yes";
```

개선 전과 개선 후를 보니, jpa의 장점이 무엇인 지 느껴진다.  
첫 번째로는, 훨씬 가독성이 좋아졌다. 1번 함수를 통해 얻어왔던 쿠폰 정보를 훨씬 단순한 2번 함수로도 얻어올 수 있다.
> 동일한 쿼리가 발생한다. 

두 번째로는, 훨씬 객체지향적인 것 같다. 즉, 2번 코드에서는 데이터베이스의 패러다임이 전혀 보이지가 않는다. 그냥 객체에서 정보를 꺼내는 것처럼 코딩을 해도 쿼리가 날라가고 정보를 가져온다.  
> 이전 블로그에서 살펴봤지만, 저 _user가 hibernate가 관리하는 persistence상태여서 getCoupons함수만 호출해도 hibernate가 쿼리를 만들고, 정보를 가져온다.
> 마치 persistence 상태인 example객체의 setName만 호출해도 update 쿼리가 날라가는 것처럼 말이다.  

개선하고 나면, 다음과 같이 쿼리가 날아간다. 
```
Hibernate: 
    select
        users0_.id as id1_1_,
        users0_.password as password2_1_,
        users0_.username as username3_1_ 
    from
        users users0_ 
    where
        users0_.username=? 
        and users0_.password=?

Hibernate: 
    select
        coupons0_.id as id1_0_0_,
        coupons0_.couponnumber as couponnu2_0_0_,
        coupons0_.username as username3_0_0_,
        coupons0_.users_id as users_id4_0_0_ 
    from
        coupons coupons0_ 
    where
        coupons0_.users_id=?
```

hibernate가 날리는 쿼리를 보면 알겠지만, users table로 가서 username과 password에 해당하는 users_id를 가져오고, 또 coupons table로 가서 users_id에 해당하는 coupon 정보를 가져오는 방식이다. 
<br>
여기서 나는 username과 password가 주어지면 해당하는 coupon정보를 one query로 가져오고 싶었다. 
> 데이터 베이스에 user call을 날리는 것은 network를 경유하기 때문에, 주어진 요건을 최대한 one query로 해결하는 것이 좋다고 들었기 때문이다. 

그래서 jpql을 사용했다.  
<br>
-- CouponsRepository.java
```java
@Repository
public interface CouponRepository extends JpaRepository<Coupons, Long> {
    
    ...

    @Query(value = "select c from Coupons c where c.users.id = (select u.id from Users u where u.username = :username and u.password = :password)")
    Coupons findByUsernameAndPassword(String username, String password);

    ...

}
```

-- EventController.java
```java
try {
    ...

    Optional<Coupons> coupon = Optional.ofNullable(couponRepository.findByUsernameAndPassword(username, password));

    ...
}catch(NoSuchElementException e){
    return e.getMessage();
}
return "yes";
```

이렇게 사용하면 one query로 가져올 수 있다.

```
Hibernate: 
    select
        coupons0_.id as id1_0_,
        coupons0_.couponnumber as couponnu2_0_,
        coupons0_.username as username3_0_,
        coupons0_.users_id as users_id4_0_ 
    from
        coupons coupons0_ 
    where
        coupons0_.users_id=(
            select
                users1_.id 
            from
                users users1_ 
            where
                users1_.username=? 
                and users1_.password=?
        )

```
위와 같이 내가 실제로 jpql로 query를 만들어도 되지만, repository의 함수 이름을 잘 정하면 hibernate가 만들어주는 query를 날릴 수도 있다.  
<br>
-- CouponsRepository.java
```java
@Repository
public interface CouponRepository extends JpaRepository<Coupons, Long> {
    ...

    Coupons findByUsersUsernameAndUsersPassword(String username, String password);

    ...
}
```
hibernate는 쿼리를 이렇게 만들어서 날려준다.  
```
Hibernate: 
    select
        coupons0_.id as id1_0_,
        coupons0_.couponnumber as couponnu2_0_,
        coupons0_.username as username3_0_,
        coupons0_.users_id as users_id4_0_ 
    from
        coupons coupons0_ 
    left outer join
        users users1_ 
            on coupons0_.users_id=users1_.id 
    where
        users1_.username=? 
        and users1_.password=?

```

만약에 너무 복잡한 것이 아니라면 이름을 잘 지어서 hibernate가 만들어주는 쿼리를 사용하는 것도 괜찮은 방법같다.

