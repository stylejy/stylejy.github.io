---
layout: post
title: HoC(Higher-Order Component) 간단하게 알아보기
date: 2020-09-24
img: hoc.jpg # Add image post (optional)
tags: [HoC패턴, HOC, 패턴, 설계, Higher Order Component, 컴포넌트, 리액트, react]
author: JooYoung Lee # Add name author (optional)
---

React 를 사용 하여 개발을 진행하다 보면, HOC 라는 말을 듣게 된다. 특히 라이브러리 사용때 많이 듣게 되는데 너무나 당연하게 축약어를 사용하였어서 당황했던 경험이 있었다. Higher-Order Component의 약자인 HOC는 프로젝트가 커질수록 아주 유용하다. 혼자 이것저것 만들다가 꽤 커진 프로젝트까지 만들었던 아직 기본 개념이 부족했던 초기 에는 HOC가 무엇인지는 몰랐지만 이미 HOC를 쓰고 있었다는 사실을 나중에 알게 되었다.

사실 HOC 에 대한 자세한 내용들은 인터넷에 많이 있다. 대표적으로 [React Docs][react-hoc]와, 한국어로 잘 설명된 케이스로는 [VELOPERT.LOG][velopert.log-hoc]가 있다. 설명과 예제가 친절하게 나와 있으니 바로 자세히 알고 싶으신 분은 위의 참고 하시면 좋을 것 같다. 하지만 정말 이게 뭔지 간단하게 알고 싶은 사람들을 위해서(내가 그랬다...) 이 포스트에서 아주아주 간단하게 개념적으로 이게 뭔지만 정리해 보려 한다.

HOC의 핵심은 한단어로 아래와 같다.
> A higher-order component is a function that takes a component and returns a new component.
React Docs 의 설명에서 가져왔다. 

간단히 예를 들자면,
A와 B라는 두개의 컴포넌트가 있다고 가정하자. 그리고 H라는 Higher Order Component 가 있다고 해보자.

이럴 경우, H가 HOC로 불리기 위해서는  A나 B 같은 컴포넌트를 H가 parameter로 쓸 수 있어야 하고,
컴포넌트를 리턴 하는 것이라야 한다.
아래 예를들면,
{% highlight javascript %}
import React from 'react';

const H = (WrappedComponent, ...parameters) => {
  return class extends React.Component {
    /**
    * parameter 들을 활용한 처리 로직들...
    */
    render() {
      return <WrappedComponent {...this.props} />
    }
  }
}
{% endhighlight %}
위의 예제는 아주 간단한 예제로 H(A)혹은, H(B) 이렇게 콜을 한다면, 처리 로직들을 통해 나온 props들과 함께 연산이 된 A혹은 B의 결과 값을 return 하게 되는 구조이다. 
이렇게 쓰는 이유는 A와 B에서 똑같거나 비슷한 방법으로 사용하는 통합된 로직을 H에만 정의해 놓고, A와 B에서 다른 부분만 각각 가지고 있게 함으로써 코드를 깔끔하게 유지할 수 있고, 유지 보수시에 한 코드의 수정으로 H 를 HOC방식으로 사용 하는 모든 컴포넌트에 한번에 적용할 수 있다는 장점이 있다. 그 외에도 다양한 방법으로 응용 및 활용이 가능한 패턴이다. 앞으로도 계속 다양한 사용방식에 대해서 공부하고 사용하면 실제 프로젝트에서도 좋은 코드를 기대할 수 있을 것 같다.

[velopert.log-hoc]: https://velopert.com/3537
[react-hoc]: https://reactjs.org/docs/higher-order-components.html