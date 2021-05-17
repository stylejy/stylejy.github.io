---
layout: post
title: macOS버전에 따라 동작 가능한 iOS 시뮬레이터 버전이 달라진다?
date: 2021-05-17
# img: jpa-mapping.jpg # Add image post (optional)
tags: [macOS, iOS, simulator, xcode, 시뮬레이터, 안나옴]
author: JooYoung Lee # Add name author (required-fixed)
author-pic: jy.jpg # Add author-pic (required-fixed)
about-author: Love... # Add about-author (about 과 같음) (optional-fixed)
email: stylejy@gmail.com # email (optiona-fixed)
---

웹뷰 개발을 하다보면 오래된 시뮬레이터 버전에서 테스트를 해야 하는 상황이 생기게 됩니다. 그런데 xcode 에서 해당 버전이 안보여서 정말 난감한 경험을 했습니다.

해당 버전을 살려보려고 xcode도 여러번 지웠다 깔아보고, xcode의 버전도 낮춰 시도 해보고, 시뮬레이터도 여러번 다시 지웠다가 깔았는데도 시뮬레이터 생성시 해당 버전이 나타나지 않았습니다. 분명히 Preferences-Components 탭에 해당 런타임을 다운 받았는지 확인을 했고, xcode도 깨끗하게 지우기 위해 여러 방법들을 사용했는데도 끝내 돌아오지 않았습니다.

꽤 많은 삽질 끝에 이유를 알았습니다.
> macOS 버전에 따라서 구동시킬 수 있는 simulator version이 달라진다.
네 맞습니다. 구 버전의 macOS 에서 잘 테스트하던 시뮬레이터가 xcode나 시뮬레이터의 변경 없이 macOS 버전만 올려도 구동이 불가할 수도 있습니다.

최신 버전의 xcode(글을 작성하는 시점에서 12.5버전)에서는 구동 불가능한 버전의 시뮬레이터 런타임(iOS 11.4 미만의 버전)은 Preferences-Components 탭에서도 없어지는 것을 확인 했습니다. 하지만 그 이전 버전의 xcode 구버전에서는 Components 탭에서 나타나기는 하나 구동할 수는 없습니다.

>근데? 이게 다른 이유일 수도 있지 않나? 정말 xcode나 simulator 설정이 꼬인 것일 수도 있잖아?
그럴 수도 있죠, 이 때 사용할 수 있는 명령어가 있습니다.

> xcrun simctl list runtimes

위 명령어를 입력하면 아래와 같은 화면을 보실 수 있습니다.
![Screenshot](/./assets/img/2021-05-17-list-runtimes.png){: width="100%" height="100%"}

저 스크린샷은 찍은 기준으로 저의 macOS 버전은 11.3.1 Big Sur 입니다. 보시다시피 11.01, 11.1, 11.2 는 macOS 10.15.99 버전부터 지원하지 않는다고 알려줍니다. 이것을 통해서 무엇이 문제인지 확인할 수 있습니다.

macOS와 구동 가능한 iOS 시뮬레이터 버전에는 상관 관계가 있으니 오래된 시뮬레이터 버전의 사용이 필요한 환경이라면 macOS 업그레이드를 하기 전에 신중하게 검토를 해보셔야 할 것 같습니다.