---
layout: post
title: JS Named Parameter 실패사례
date: 2020-10-25
img: named-param.png # Add image post (optional)
tags: [Named Parameter, 네임드 파라미터, Object Parameter, JS paramter]
author: JooYoung Lee # Add name author (optional)
---

회사에서 리팩토링 파트를 맡게 되면서 JS Named Parameter 스타일을 도입해 보기로 했다. JS 에서는 네이티브로 Named Parameter를 지원 하지는 않지만, [Destructuring Assignment][MDN.Destructuring-assignment]를 이용하여 구현한 방식을 현재 많이 사용되고 있다.

이 스타일을 사용하면 빠른 개발 속도로 여러명이 수정하는 코드에서 파라미터에 이름을 사용하면 실수를 줄일 수 있을 것 같아서 도입했다. 아래와 같이 명시적으로 불러야 하게 때문이다.
{% highlight javascript %}
function({
  myId: 'myid',
  userOption: { ... }
})
{% endhighlight %}

이후 개발 관련 책을 읽다가([자바스크립트 코딩의 기술][book]), 좀더 효율적인 테스트를 해보고자 도입한 Parameter를 활용한 Dependency Injection(DI) 스타일을 적용하면서 문제가 발생했다.

{% highlight javascript %}
import { importedFunction } from 'others';

const main = {
  test1({
    myId, userOption
  }) {
    const res = importedFunction.blog(myId)
  }
}
{% endhighlight %}
위와 같은 코드가 있었다면,

{% highlight javascript %}
import { importedFunction } from 'others';

const main = {
  test1({
    myId, userOption, blog = importedFunction.blog
  }) {
    const res = blog(myId)
  }
}
{% endhighlight %}
importedFunction 같은 코드내 모든 dependency를 없애고 모두 parameter 를 통해연결하여, 외부 라이브러리를 사용하지 않고 DI를 구현하였다.

이렇게 되면서 한가지 문제가 발생하였는데, 유저의 입력이 필요 없어서 parameter가 하나도 없던 function들도 DI를 위해서 paramter로 올라오게 되었다. 물론 유저 입장에서는 default value들이 지정되어 있기 때문에 parameter 는 필요 없었지만, 단한가지 바뀌는 것이있는데 아래와 같다.

{% highlight javascript %}
function() // DI 적용 전
function({}) // DI 적용 후
{% endhighlight %}

만약에, DI 적용 후 {} 없이 function() 를 부르게 된다면 test 에서는 해당 function을 찾을 수 없다는 에러가 나온다.
하지만, chrome devtool 에서는 내부에 default value의 module(위의 예제에서 importedFunction 부분)의 binding 이 제대로 되지 않았는데, 정상적으로 파일에 function에 대한 코드가 정의되어 있음에도 불구하고 'function of undefined' 이런식의 에러가 나타났다. 
실제로 많은 개발자들이 업데이트가 되지 않았거나 실수로 {} 를 없이 사용하면서 문제가 발생하였는데, 초기에는 에러값만 보고 에러를 파악하기도 헤맸던 일이 있었다.

이후 리팩토링을 하면서 기존의 parameter 입력 방식과, named parameter 방식 모두를 지원하게 하여 실수 및 호환성 문제도 모두 없애는 쪽으로 갔지만, 예상지 못했던 사이드 이펙트가 큰 문제를 일으켰던 순간 이었다. 교훈 삼아서 좀더 세심하게 설계하고 적용해야 되겠다는 다짐을 하는 계기였다.

[MDN.Destructuring-assignment]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment
[book]: https://book.naver.com/bookdb/book_detail.nhn?bid=15971893