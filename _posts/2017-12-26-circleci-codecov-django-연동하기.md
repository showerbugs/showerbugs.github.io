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

- [https://github.com/hjwp/book-example](https://github.com/hjwp/book-example)
- [클린 코드를 위한 테스트 주도 개발](http://www.yes24.co.kr/24/goods/16886031)에서 사용되는 예제입니다.
- 프로젝트 내에서 테스트 코드를 많이 작성해서 본 포스트에 적절하기에 사용합니다.

## 테스트 Project Fork

아래 프로젝트를 Fork합니다. Circle CI 연동부터는 해당 Repository가 있어야 진행이 가능합니다.

[https://github.com/hjwp/book-example](https://github.com/hjwp/book-example)

![1_fork]({{site.url}}/images/2017-12-26/1_fork.png)

Fork한 프로젝트를 `YOUR_ACCOUNT`를 자신의 계정으로 바꾸어 Clone합니다. 

```sh
$ git clone git@github.com:YOUR_ACCOUNT/book-example.git
```

## Circle CI 연동

다음으로 circleci에 접속해 로그인합니다.

![{{site.url}}/images/2017-12-26/1_circleci.png]({{site.url}}/images/2017-12-26/1_circleci.png)

> GitHub으로 로그인하면 repository 연동이 간편하니 참고 바랍니다.

화면 좌측에서 `PROJECTS` 버튼을 클릭하고 `Add Project` 버튼을 클릭합니다.

![{{site.url}}/images/2017-12-26/2_circleci.png]({{site.url}}/images/2017-12-26/2_circleci.png)


목록에서 연동할 프로젝트를 찾아 `Setup project`를 클릭합니다.

> 연동이 바로 이루어지지는 않기 때문에, 프로젝트가 보이지 않을 경우 로그아웃 이후 재 로그인을 하거나 5분 정도 기다린 이후에 다시 시도합니다. 

![{{site.url}}/images/2017-12-26/3_circleci.png]({{site.url}}/images/2017-12-26/3_circleci.png)

최근에 Circle CI 2.0이 릴리즈 되었으니 그것을 사용하도록 하겠습니다.

> 큰 특징으로는 workflow별로 분리해서 script를 입력할 수 있어서, 병렬 혹은 순차적으로 진행되는 step들을 가시적으로 볼 수 있습니다. 이전 버전과의 또 다른 차이는 .circleci라는 디렉터리에 빌드 스크립트를 넣어야 한다는 것입니다.

![{{site.url}}/images/2017-12-26/4_circleci.png]({{site.url}}/images/2017-12-26/4_circleci.png)

Repository를 분석해서 Language는 자동으로 파악해 Python이 선택되었네요.

## circleci 스크립트 작성

이제 주어진 안내에 따라서 빌드 스크립트를 작성합니다.

clone한 프로젝트로 들어가서 `.circleci` 디렉토리를 만들고 `config.yml` 파일을 만듭니다.

```sh
$ cd book-example
$ mkdir .circleci
$ vim .circleci/config.yml
```

CircleCI에서 제공하는 Sample `config.yml` 파일을 그대로 사용하지 않고 아래 내용을 붙여 넣습니다

- functional test에서 selenium을 사용하고 있는데 이를 제외하기 위함입니다. selenium 테스트는 해당 포스트에서 다루지 않기 때문에 제외합니다.
- `python manage.py test accounts lists`

```yml
# Python CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-python/ for more details
#
version: 2
jobs:
  build:
    docker:
      # specify the version you desire here
      # use `-browsers` prefix for selenium tests, e.g. `3.6.1-browsers`
      - image: circleci/python:3.6.1
      
      # Specify service dependencies here if necessary
      # CircleCI maintains a library of pre-built images
      # documented at https://circleci.com/docs/2.0/circleci-images/
      # - image: circleci/postgres:9.4

    working_directory: ~/repo

    steps:
      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "requirements.txt" }}
          # fallback to using the latest cache if no exact match is found
          - v1-dependencies-

      - run:
          name: install dependencies
          command: |
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt

      - save_cache:
          paths:
            - ./venv
          key: v1-dependencies-{{ checksum "requirements.txt" }}
        
      # run tests!
      - run:
          name: run tests
          command: |
            . venv/bin/activate
            python manage.py test accounts lists

      - store_artifacts:
          path: test-reports
          destination: test-reports
```

저장하고 `.circleci` 를 추가하고 push합니다.

```sh
$ git add .circleci
$ git commit -m "Add circle ci script"
$ git push origin master
```

그리고 Circle CI에서 `Start Building` 버튼을 클릭합니다.

![{{site.url}}/images/2017-12-26/5_circleci.png]({{site.url}}/images/2017-12-26/5_circleci.png)

빌드 작업이 Queue에 들어가서 Circle CI가 `book-example` 의 테스트를 구동시킨 것을 확인할 수 있습니다.

![{{site.url}}/images/2017-12-26/6_circleci.png]({{site.url}}/images/2017-12-26/6_circleci.png)

`README.md`에 멋진 Badge를 추가하고 Circle CI 연동을 마무리 하도록 하겠습니다

우측 상단의 설정 버튼을 클릭합니다.

![7_circleci.png]({{site.url}}/images/2017-12-26/7_circleci.png)

`Status Badges` 메뉴를 클릭하고 `Embedded Code`를 복사합니다

![8_circleci.png]({{site.url}}/images/2017-12-26/8_circleci.png)

복사한 `Embedded Code`를 `README.md`의 적절한 위치에 붙여 넣습니다.

> 여기서는 최 상단에 붙여 넣었습니다.

![9_circleci.png]({{site.url}}/images/2017-12-26/9_circleci.png)

Repository에 들어올 때 보이는 대문에 멋진 CircleCI Badge가 추가된 것을 볼 수 있습니다.

![10_circleci.png]({{site.url}}/images/2017-12-26/10_circleci.png)

이제 Circle CI가 book-example repository에 변동이 생길 때마다 테스트 성공 여부를 알려줘서 프로젝트의 심장 박동 측정기의 역할을 해줄 것입니다 ^^

## Codecov 연동하기

Test Coverage는 개발 도중에 참고하면 상당히 개발에 도움이 되는 지표입니다. 테스트 코드가 제품 코드를 얼마나 커버하고 있고 어떤 부분이 취약한지를 빠르게 알 수 있기 때문입니다.

> 그 수치 자체를 목표로 한다면 개발자들이 테스트 코드 작성을 수치를 높이기 위한 수명업무처럼 생각하게 되어 테스트 코드가 의미가 없어질 수 있습니다. 또한 무의미한 테스트 케이스가 늘어나게 되어 테스트 구동시간이 길어져 테스트 구동 자체가 큰 버든이 될 수 있습니다. 따라서 수치를 높이기 위한 활동은 매우 신중하게 시작해야 합니다.

Python에는 `coverage`라는 패키지를 사용하면 매우 간편하게 Coverage를 측정할 수 있습니다. 또한, CodeCov와 같은 서비스를 활용하면 Coverage 상세 정보를 web에서 정리된 형태로 볼 수 있어 프로젝트의 현황을 파악하는데 도움을 받을 수 있습니다.

![1_codecov]({{site.url}}/images/2017-12-26/1_codecov.png)

여기서는 위에서 Circle CI를 연동한 프로젝트를 대상으로 Coverage를 측정하고 CodeCov로 결과를 보내보도록 하겠습니다.

먼저 CodeCov에 접속하고 로그인합니다. 위의 CircleCI와 마찬가지로 GitHub 계정으로 로그인하는 것을 권장합니다.

![2_codecov]({{site.url}}/images/2017-12-26/2_codecov.png)

fork한 자신의 계정을 클릭합니다.

![3_codecov]({{site.url}}/images/2017-12-26/3_codecov.png)

`Add new repository` 버튼을 클릭하고, 출력된 리스트에서 book-example을 찾아 클릭합니다.

![4_codecov]({{site.url}}/images/2017-12-26/4_codecov.png)

이제 Codecov 연동이 완료되었습니다. 

> 아래에 Token이 출력되는데 CircleCI나 TravisCI를 사용하는 경우에는 Token 없이도 Report 전송이 가능합니다.

![5_codecov]({{site.url}}/images/2017-12-26/5_codecov.png)

## CodeCov에 Coverage Report 전송

먼저 coverage를 측정하기 위해 config.yml을 수정하도록 하겠습니다.

- 아래 소스를 복사해서 `.circleci/config.yml`에 붙여넣으시면 됩니다

변경 사항은 아래와 같습니다.

- `pip install coverage codecov` 추가
- `python manage.py test accounts lists`를 `coverage run ./manage.py test accounts lists`로 변경
- coverage 이후 `codecov` 추가

```yml
# Python CircleCI 2.0 configuration file
#
# Check https://circleci.com/docs/2.0/language-python/ for more details
#
version: 2
jobs:
  build:
    docker:
      # specify the version you desire here
      # use `-browsers` prefix for selenium tests, e.g. `3.6.1-browsers`
      - image: circleci/python:3.6.1
      
      # Specify service dependencies here if necessary
      # CircleCI maintains a library of pre-built images
      # documented at https://circleci.com/docs/2.0/circleci-images/
      # - image: circleci/postgres:9.4

    working_directory: ~/repo

    steps:
      - checkout

      # Download and cache dependencies
      - restore_cache:
          keys:
          - v1-dependencies-{{ checksum "requirements.txt" }}
          # fallback to using the latest cache if no exact match is found
          - v1-dependencies-

      - run:
          name: install dependencies
          command: |
            python3 -m venv venv
            . venv/bin/activate
            pip install -r requirements.txt
            pip install coverage codecov

      - save_cache:
          paths:
            - ./venv
          key: v1-dependencies-{{ checksum "requirements.txt" }}
        
      # run tests!
      - run:
          name: run tests
          command: |
            . venv/bin/activate
            coverage run ./manage.py test accounts lists
            codecov

      - store_artifacts:
          path: test-reports
          destination: test-reports
```

`config.yml`을 수정한 이후 push하면 CircleCI에서 Coverage를 분석한 이후에 Codecov로 정보를 전송하는 것을 볼 수 있습니다.

![6_codecov]({{site.url}}/images/2017-12-26/6_codecov.png)

리포트는 아래처럼 출력됩니다.

> [https://codecov.io/gh/amazingguni/book-example](https://codecov.io/gh/amazingguni/book-example)

![7_codecov]({{site.url}}/images/2017-12-26/7_codecov.png)

CodeCov도 Badge를 지원하는데 이를 추가하고 포스트를 마무리 하도록 하겠습니다.

CodeCov Report에서 우측 상단의 `Settings`를 클릭합니다.

![8_codecov]({{site.url}}/images/2017-12-26/8_codecov.png)

메뉴에서 Badge를 누르고 Markdown을 복사합니다

![9_codecov]({{site.url}}/images/2017-12-26/9_codecov.png)

Circle CI와 동일하게 README.md에 Markdown을 추가해 Badge를 추가합니다.

![10_codecov]({{site.url}}/images/2017-12-26/10_codecov.png)

![11_codecov]({{site.url}}/images/2017-12-26/11_codecov.png)

CircleCI와 Codecov 연동이 완료되었습니다. 이제부터는 매 Commit마다 이 두 도구가 코드 품질을 관찰하고 알려줄 것입니다. 

또한 Pull Request에 이 도구들이 comment를 달아주어 리뷰에 도움을 받을 수 있습니다.

![12_codecov]({{site.url}}/images/2017-12-26/12_codecov.png)

![13_codecov]({{site.url}}/images/2017-12-26/13_codecov.png)


## Trouble Shooting

Organization의 Repository가 보이지 않을 경우 GitHub 연동 시에 권한을 설정하지 않았을 수 있습니다. 

Github 로그인 이후 `Settings`에 들어갑니다.

![{{site.url}}/images/2017-12-26/1_trouble.png]({{site.url}}/images/2017-12-26/1_trouble.png)

`Applications > Authorized OAuth Apps > CircleCI` 를 클릭합니다

![{{site.url}}/images/2017-12-26/2_trouble.png]({{site.url}}/images/2017-12-26/2_trouble.png)

`Organization access`에서 CircleCI와 연동하려는 Repository의 Organization이 체크되어 있는지 확인하고 그렇지 않을 경우 권한을 부여합니다.

![{{site.url}}/images/2017-12-26/3_trouble.png]({{site.url}}/images/2017-12-26/3_trouble.png)


