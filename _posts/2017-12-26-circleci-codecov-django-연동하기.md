---
layout: post
title: "django 프로젝트에 CircleCI와 CodeCov 연동하기"
author: "amazingguni"
tags: python ci django test
---

우리 프로젝트가 팀이 기대하는 수준의 품질이 보장된다는 것을 확인해주는 안전장치(어떤 책에서는 심장 박동이라고 하더라)가 있다면, 우리는 좀 더 자신감을 가지고 개발을 할 수 있을 겁니다. 우리가 잘못된 길을 갈 때마다 우리에게 그 길이 아닌 다른 길을 가야한다고 알려줄거라는 믿음이 있기 때문이죠. 그래서 Repository를 처음 셋업할 때에 Continuous Integration(줄여서 CI) 체계를 갖추는 것은 그 어떤 일보다 중요합니다.

만약 소스 코드를 github에 올리는 상황이라면, 많은 Continuous Integraion 서비스들이 코드를 공개하는 경우에 한해 무료로 서비스를 제공하고 있습니다. 여기서는 그 중에서도 CircleCI를 사용하도록 하겠습니다.

> 현재 가장 대중적으로 많이 쓰이고 있는 Travis와 거의 같은 기능을 제공하면서도 좀 더 많은 Dashboard와 기능을 제공하기 때문이다. 

이제부터 python django framework를 사용한 프로젝트와 Circle CI/ codecov의 연동을 시작합니다.

- [https://github.com/amazingguni/python-cleancode-tdd](https://github.com/amazingguni/python-cleancode-tdd)
- [클린 코드를 위한 테스트 주도 개발](http://www.yes24.co.kr/24/goods/16886031) 이라는 책을 공부하기 위해 만든 repository 입니다.
- 프로젝트 내에서 테스트 코드를 많이 작성해서 본 포스트에 적절하기에 사용합니다.

### Circle CI 연동

먼저, circleci에 접속해 로그인합니다.

![{{site.url}}/images/2017-12-26/1_circleci.png]({{site.url}}/images/2017-12-26/1_circleci.png)
  
