---
layout: post
title: Jest spyOn으로 unit test function mocking 하기
date: 2020-08-28
img: spyon-jest.jpg # Add image post (optional)
tags: [Blog, Development, Jest, spyOn]
author: JooYoung Lee # Add name author (optional)
---

최근 개발 능력을 향상시키고자 읽었던 '[소프트웨어 장인][book]'이라는 책에서 나왔던 TDD(Test Driven Development)를 나의 업무에 적용해 보고자 했다.

좀더 세심한 test case 를 계획하다 보니 그동안은 보이지 않았던 해결해야 할 문제들이 떠올랐다.
테스트 코드에서 호출하는 function를 대체해야 올바른 테스트 결과를 얻을 수 있는 문제였는데, 어떤 방식을 써야 할지 마땅히 떠오르지 않았다. Test 에 익숙하지 않았던 탓에, 처음에 떠 올렸던 아이디어는 임시로 원본 코드에 테스트용 function 을 추가 하여 대체하고 테스트만 하고 다시 원래 대로 돌려 놓는 아이디어였다.

근데 이건 좋은 Practice 는 아닌 것 같다. 테스트를 하기 위해서 원본 소스를 건드린다는 것 자체가 좋은 방법은 아닌 것 같았고, 실수로 인해서 test 는 통과 하였지만 실제 환경에서는 동작하지 않는 코드가 되는 문제가 발생할 수도 있었기 때문에 다른 대안을 찾기 시작했다.
다행히 테스트 툴로 사용하던 Jest 에서 spyOn과 mockImplementation의 조합을 해당 문제를 해결 할 수 있었다.

`spyOn` 함수는 테스트 코드에서 어떤 function 이 실제로 불리는지는 알 수 있게 해주는 Api 이다. 함수가 포함되어 있는 object를 spyOn의 첫번재 parameter로 보내고, methodName을 string으로 두번째 parameter 로 보내면 된다.

{% highlight javascript %}
const video = {
  play() {
    return true;
  },
};

module.exports = video;
{% endhighlight %}

{% highlight javascript %}
const video = require('./video');

test('plays video', () => {
  const spy = jest.spyOn(video, 'play');
  const isPlaying = video.play();

  expect(spy).toHaveBeenCalled();
  expect(isPlaying).toBe(true);

  spy.mockRestore();
});
{% endhighlight %}

위의 예제는 jest docs 페이지에서 가지고 온 것인데, video object 에서 정의된 play function 을 spying 하기 위해 `jest.spyOn(video, 'play')`를 사용한 것을 볼 수 있다.

spyOn 만으로는 내부에 있는 function call 을 override 하지 않는다. 그래서 `mockImplementation` 을 사용해야 하는데, spyOn에 연결하여 chain 방식으로 쓸 수 있다.

{% highlight javascript %}
const video = require('./video');

test('plays video', () => {
  const spy = jest.spyOn(video, 'play').mockImplementation(() => false);
  const isPlaying = video.play();

  expect(spy).toHaveBeenCalled();
  expect(isPlaying).toBe(true);

  spy.mockRestore();
});
{% endhighlight %}

위의 예제는 spyOn에 mockImplementation을 chaining 한 것을 볼 수 있는데, 이렇게 해 놓으면 const spy 밑에서 불리는 play 함수는 mockImplementation 에서 정의 한 function 이 불리게 된다. 그렇게 되면 밑에 코드에서 `expect(spy).toHaveBeenCalled()` 는 video.play() 가 불려기 때문에 test 통과를 하겠지만, `expect(isPlaying).toBe(true)` 는 mockImplementation 에서 play 가 false 를 반환하게 설정을 해 놓았기 때문에 실패하는 케이스가 된다. (예제를 위한 코드일뿐 당연히 테스트를 통과 하게 만들어 놔야 한다.........)

[Jest Docs][jest-docs]

[book]: https://book.naver.com/bookdb/book_detail.nhn?bid=9585753
[jest-docs]: https://jestjs.io/docs/en/jest-object
