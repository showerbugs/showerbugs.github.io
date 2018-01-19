---
layout: post
title: "Apex, AWS로 Facebook 날씨 봇 개발 (1)"
author: "qodot"
tags: python apex aws lambda apigateway facebookbot
categories: python aws
---

서버리스를 향한 관심이 점점 높아지고 있는 가운데, 나도 이걸로 무언가 만들어보고 싶다는 생각을 종종 하곤 했다. 그러다가 출근 준비할 때마다 하는 날씨 검색을 페이스북 봇을 이용해서 자동화할 수 있겠다는 생각, 그 작업을 서버리스 기술을 이용해서 할 수 있겠다는 생각이 들었다.

이 아티클에서는 페이스북 봇을 생성하고 웹훅과 람다를 연결하는 작업을 진행한다.

# 요구사항

1. 페이스북 봇을 통해 원하는 시간과 방송사를 입력 받는다. (ex: ytn 7시)
2. 지정된 시간이 되면, 지정한 방송사의 가장 최근 날씨 영상 뉴스(TV 방영분) URL이 페이스북 봇으로 푸시된다.

# 기술 선택

1. 파이썬을 사용한다.
2. 페이스북 봇의 webhook(사용자 입력)은 AWS API Gateway -> AWS Lambda로 처리한다.
3. 유저가 입력한 상태는 AWS DynamoDB에 저장한다.
4. 푸시를 위한 시간 이벤트는 AWS CloudWatch 이벤트를 이용한다.
5. CloudWatch 이벤트가 발생하면 실행될 코드는 Lambda를 이용한다.
6. 모든 Lambda는 [Apex](http://apex.run/)로 관리한다.

# 페이스북 봇 생성

우선 [페이스북 개발자 페이지](https://developers.facebook.com/)에서 애플리케이션을 생성해야 한다.

![fb-new-app](https://wiki.qodot.me/py/apex-aws-facebookbot/fb-new-app.png)

메신저 플랫폼을 선택하고,

![fb-new-messanger](https://wiki.qodot.me/py/apex-aws-facebookbot/fb-new-messanger.png)

페이스북 페이지를 새로 생성해서 엑세스 토큰을 발급 받는다.

![fb-new-page-token](https://wiki.qodot.me/py/apex-aws-facebookbot/fb-new-page-token.png)

# 페이스북 웹훅 등록

엑세스 토큰을 받았으면 웹훅을 내 람다와 연결해보자.

## Apex로 Lambda 생성

[Apex](http://apex.run/)는 람다 함수의 설정, 실행, 배포 등의 관리를 편하게 할 수 있도록 도와주는 프레임워크다. Node.js 진영의 스타 프로그래머였던(지금은 Go로 옮긴) TJ Holowaychuk이 만든 도구이기도 하다.

### 설치

다음 명령어로 설치할 수 있다.

```bash
curl https://raw.githubusercontent.com/apex/apex/master/install.sh | sh
```

### 프로젝트 생성

프로젝트를 생성하려면 AWS 인증 정보가 있어야 한다. [AWS CLI 구성](http://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-chap-getting-started.html) 페이지를 참고해서 AWS 인증 정보를 생성한다.

Apex가 `~/.aws/credentials`, `~/.aws/config`의 AWS 인증 정보를 읽어오는 방법은 여러가지가 있는데, 적용 순서는 다음과 같다.

1. 명령어의 `--profile` 옵션
2. `project.json` 파일의 `profile` 값
3. 환경 변수의 `AWS_PROFILE` 값
4. `~/.aws` 설정 파일의 `default` 섹션

원하는 방법으로 AWS 인증 프로필을 선택할 수 있게 설정한 뒤, 다음 명령어를 입력해서 프로젝트를 생성한다.

```bash
apex init
```

프로젝트의 이름과 설명을 넣고 나면 apex가 자동으로,

- AWS IAM을 생성하고, 그 IAM에 로그를 위한 Policy를 추가한다.
- 현재 디렉토리 안에 `project.json` 파일과 `functions` 디렉토리를 생성한다.
- `functions` 디렉토리 안에는 `hello`라는 람다 함수의 템플릿이 생성되어 있다.

### 함수 생성

이미 `hello` 함수가 생성되어 있으므로 이 함수를 바로 배포하고 실행해보는 것이 가능하다.

```bash
apex deploy hello
apex invoke hello
{"hello":"world"}
```

> `invoke`는 실제 람다 함수를 실행시키는 명령어기 때문에 반드시 먼저 배포를 해야 동작한다.

AWS Lambda의 웹 콘솔에서 `<프로젝트이름_함수이름>`으로 람다 함수가 생성된 것을 확인할 수 있다.

#### Webhook 함수 생성

이제 실제로 웹훅에서 사용할 함수를 생성해보자.

대충 `fb_webhook`쯤의 이름으로 디렉토리를 만들고(디렉토리 이름이 곧 함수 이름이 된다는 것에 유의하자), 그 안에 `main.py`와 `function.json` 파일을 생성해준다.

`function.json`을 다음과 같이 수정해서 람다의 런타임이 파이썬 3.6이라는 것을 명시해준다.

```json
{
    "runtime": "python3.6"
}
```

`main.py`는 다음과 같이 수정해서 진입점을 구성한다.

```py
def handle(event, context):
    pass
```

> 진입점은 `function.json`의 `handler` 설정 값으로 바꿀 수 있다. 파이썬의 기본 진입점은 `main.handle`이다. (`main` 모듈의 `handle` 함수)

일단 여기까지만 작성하고 `deploy`로 배포해두자.

## API Gateway로 웹훅 URL 인증

페이스북 메신저 플랫폼에 웹훅 서버를 등록하는 방법은 다음과 같다.

![fb-webhook](https://wiki.qodot.me/py/apex-aws-facebookbot/fb-webhook.png)

즉, 여기에 입력한  URL에서 입력한 토큰을 처리해주면 인증이 되는 구조다. (해당 URL의 `GET` 메서드로 인증을 받고 나면 `POST` 메서드로 메세지가 들어오게 된다) 그리고 우리가 만든 웹훅용 람다 함수가 인터넷에서 호출이 가능하려면 API Gateway를 람다의 트리거로 등록해야 한다.

### API Gateway 설정

AWS API Gateway 콘솔로 들어가서 'API 작성' 버튼을 누른다. '새 API'를 선택하고 API 이름을 입력한다.

![new-api-gateway](https://wiki.qodot.me/py/apex-aws-facebookbot/new-api-gateway.png)

리소스 -> 작업 -> 메서드 생성을 선택하고 `GET` 메서드를 생성한다. '통합 유형'에서 'Lambda 함수'를 선택하고 'Lambda 리전'을 아까 배포한 람다 함수가 있는 지역으로 선택한다. 람다 함수의 이름까지 입력하고 저장을 누른다.

![new-method](https://wiki.qodot.me/py/apex-aws-facebookbot/new-method.png)

생성된 `GET` 메서드를 클릭하면 아래와 같은 그림을 볼 수 있을텐데,

![api-gateway-flow](https://wiki.qodot.me/py/apex-aws-facebookbot/api-gateway-flow.png)

통합 요청 -> 본문 매핑 템플릿을 선택하면 '요청 본문 패스쓰루'를 설정할 수 있는데, 여기서 '정의된 템플릿이 없는 경우'를 선택하고 아래 매핑 템플릿 추가를 눌러 `application/json`을 입력해준다.

### URL 인증 람다 개발

이제 페이스북의 웹훅 URL 인증을 처리하기 위해서 람다를 개발해보자. 위에서 개발헤 놓은 `handle` 함수 안에 다음 코드를 작성한다.

```python
def handle(event, context):
    http_method = event['context']['http-method']
    if http_method == 'GET':
        return _get(event, context)

def _get(event, context):
    query_params = event['params']['querystring']
    received_token = query_params['hub.verify_token']
    if os.environ.get('FB_VERIFY_TOKEN') == received_token:
        challenge = int(query_params['hub.challenge'])
        return challenge
```

`GET` 메소드의 요청에 대해서 토큰을 검사하고 `hub.challenge` 값을 응답으로 보내고 있다. 이 때, 환경변수 `FB_VERIFY_TOKEN`를 사용하고 있는 것을 볼 수 있다. Apex에서 환경변수와 함께 람다를 배포하는 방법은 다음 두 가지가 있다.

1. `project.json`의 `environment`에 등록한다.
2. `--env-file` 옵션으로 배포시에만 JSON 파일을 환경변수로 등록해준다.

민감하지 않은 정보라면 `project.json` 파일에 직접 등록해서 VCS에서 관리되도록 하는것이 좋겠지만, 민감한 정보라면 따로 JSON 파일을 관리하는 것이 좋을 것이다.

원하는 방식으로 환경변수를 설정해준 후 `apex deploy` 명령으로 배포를 진행한다. 배포가 완료되면 최초 페이스북 웹훅 인증 화면으로 돌아가서 '콜백 URL'에 API Gateway의 URL, '확인 토큰'에 `FB_VERIFY_TOKEN`과 같은 값을 넣어주고 인증을 완료한다.
  
# 메세지 받기 & 보내기

이제 페이스북 봇을 만들기 위한 기반 작업은 끝났다. 이제 실제로 사용자가 입력한 메세지를 처리하는 로직을 개발해보자.

## API Gateway에 POST 메서드 추가

위에서 `GET` 메서드를 추가했던것과 동일한 방법으로 `POST` 메서드를 추가하고 , 역시 아까와 같은 람다 함수에 트리거를 넣어준다.

## Lambda에 코드 추가

다음과 같은 코드를 추가해서 `POST` 메서드를 처리할 수 있게 한다. 일단 받은 메세지를 그대로 돌려주는 에코 서버를 만들어보자.

```python
def handle(event, context):
    http_method = event['context']['http-method']
    if http_method == 'GET':
        return _get(event, context)
    elif http_method == 'POST':
        return _post(event, context)
        
def _post(event, context):
    entry = event['body-json']['entry'][0]
    messaging = entry['messaging'][0]

    user_id = messaging['sender']['id']
    msg = messaging['message']['text']
    
    _push(user_id, msg)

def _push(user_id, msg):
    fb_access_token = os.environ.get('fb_access_token')
    url = f'https://graph.facebook.com/v2.11/me/messages?access_token={fb_access_token}'
    data = {
        'recipient': {
            'id': user_id,
        },
        'message': {
            'text': msg,
        },
    }
    data = json.dumps(data).encode()
    headers = {
        'content-type': 'application/json'
    }
    req = request(url, data=data, headers=headers)
    urlopen(req).read()
```

들어온 메세지의 구조는 직접 `event`를 찍어보면서 확인할 수 있다. 사용자에게 메세지를 전달하는 방법은 [페이스북의 푸시 API](https://developers.facebook.com/docs/messenger-platform/reference/send-api?locale=ko_KR) 문서를 참조하면 자세한 내용을 알 수 있다.

# 다음 포스트

이번 포스트에서는 AWS의 API Gateway와 Lambda를 이용해 페이스북 메신저 봇의 기본적인 형태를 잡아 보았다. 다음 포스트에서는 유저가 메신저를 통해 입력한 값을 DynamoDB에 저장하고 CloudWatch 이벤트를 등록하는 작업을 진행해본다.
