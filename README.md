# 해당 문서는 현재 작성중입니다. 최근 수정 시간은 2017년 12월 29일 오전 09시 27분 입니다.

# AWS - Lex와 Lambda, DynamoDB를 이용한 챗봇 구현 안내서

> AWS 기반 지식 없이 AWS로 시작하는 챗봇 구현



##### AWS 기반 챗봇을 만드는 여러가지 방법

- `AWS Lex`로 구현하고 AWS에서 제공하는 대응 플랫폼(페이스북 메신저, 트윌리오 등)을 이용하여 배포하기
- `AWS Lex`로 구현하고 `AWS Gateway API`를 이용하여 커스텀 UI(웹, 앱을 직접 제작하기 등)로 배포하기
- `AWS Lambda`를 이용하여 대화 흐름을 Programming 하기
- `AWS Lambda`와 DB (`AWS DynamoDB`, `AWS RDS` 등)을 이용하여 원하는 데이터와 함께 Programming 하기
- 등등 ...

> 이 안내서는 간단하면서도 많은 일을 할 수 있는 **AWS Lex, Lambda, DynamoDB를 이용한 챗봇을 만들고, 페이스북 메신저에 배포하는 과정**에 대해 설명합니다.
>
> Lambda는 Python, Java, Node.js 를 지원하며 이 안내서는 **Python 3.6**을 사용하였습니다.



##### 더 좋은 질의 안내서들

- [Amazon Lex 와 Lambda 를 이용한 CoffeeBot 만들기](https://techpump.s3.amazonaws.com/AI%20Services%20Bootcamp_Serverless_Lex.pdf)
- AWS 공식 문서
  - [Lex](http://docs.aws.amazon.com/lex/latest/dg/what-is.html), [Lambda](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html), [DynamoDB](http://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/Introduction.html), [boto3](https://boto3.readthedocs.io/en/latest/), ...
- 등등 ...



## 목차

1. [Lex만 가지고 챗봇 만들기](#Lex)
2. [Lex에 Lambda를 더해보기](#Lambda)
3. [Lambda에서 DynamoDB 호출하기](#Lex와 Lambda의 연동)
4. [만든 챗봇을 페이스북 메신저에서 사용해보기](#페이스북 메신저에 배포하기)
5. [참고하기](#참고하기)
6. [기여하기](#기여하기)




## Lex만 가지고 챗봇 만들기

### Lex 콘솔에서 챗봇 만들기

대화 의도형 로봇입니다. USER가 원할 의도에 대해 대화흐름을 구성해 놓으면 USER의 의도에 맞춰 구성해 놓은 대화를 진행합니다. 하지만 LEX만으로는 제약이 많이 따릅니다. 예를 들면 순차적인 대화만 구성할 수 있습니다. 보다 똑똑한 LEX를 위해서는 개발자가 LEX와 USER사이에 프로그래밍 요소를 집어넣어야 하는 데 이것이 후술할 Lambda 입니다.

> 콘솔은 ~~

#### 인텐트

#### 슬롯타입

#### 어터런스

#### 슬롯

#### 에러 핸들링

먼저 인식할 때 뭘 원하는지 파악하고 데이터를 출력해줌. speech recognition

어터런스를 좀 넣으면 비슷한 말 다 인식

에러처리도 쉽게 가능

웹에 building System, 페이스북 API, 기타 챗서비스 API에 연동중

### Lex 콘솔에서 챗봇 테스트하기

빌드, publish, 옆에 클릭 ㅇㅋ

## Lex에 Lambda를 더해보기

### Lambda

`AWS Lambda`는 serverless 컴퓨팅을 제공하는 서비스입니다. 별도의 서버 없이 개발자가 정의한 함수만 수행될 수 있게 구성되어 있습니다. 개발자는 서버에 관한 지식 없이 `Python`, `Java`, `Node.js` 중 하나를 선택하여 프로그램을 작성할 수 있으며 [정해진 파라미터](#Event)로 입력을 받고 개발자가 작성한대로 실행됩니다. *본 문서는 Python 3.6을 사용하였습니다.*

Lambda의 특징은 계속 서버가 살아있는 것이 아니라 수행될 때만 서버가 켜졌다가 함수가 종료되면 서버가 종료됩니다. 그러므로 Lambda 함수 내에서 계산된 변수들은 보존되지 않습니다.

### Lambda함수 정의하기

1. `AWS Lambda` 콘솔에 접속합니다.

2. 함수생성 (블루 프린트로 생성하면 lex와 관련된 코드를 하드코딩하지 않아도 됩니다.)

   > 이름 : 사용자가 임의로 지정
   >
   > 런타임 : 작성할 언어 지정
   >
   > 역할 : 정의한 역할을 선택합니다. 해당 역할은 Lambda가 맡을 수 있고 CloudWatch Logs 권한이 있어야 합니다.

   - 역할 정의하기
     1. `AWS IAM` 콘솔에 접속합니다.
     2. `역할`탭 선택
     3. `역할 만들기`
     4. `신뢰할 수 있는 유형 개체` : Lambda 클릭 후 다음
     5. Lambda, DynamoDB, Lex로 검색하여 각각 FullAccess를 하나씩 선택합니다.
     6. 이름과 설명을 설정하고 역할 만들기를 누릅니다.

### Lex에 후크함수를 추가하기

Lex는 유저의 입력 하나하나 마다 Lambda 함수를 호출할 수 있습니다.

1. `AWS Lex` 콘솔에서 `Editor` 탭의 임의의 인텐트를 하나 선택합니다.
2. `Lambda initialization and validation`에서 Initialization and validation code hook 체크
3. 정의한 Lambda 함수를 목록에서 선택합니다.

### Event 데이터

Lambda가 다른 AWS 서비스들과 통신하는 데 있어 정해진 약속입니다. 이 규격을 벗어난 데이터는 에러를 발생시킬 수 있습니다. 그러므로 개발자는 Event 데이터 형식에 맞게 데이터를 읽고, 규격에 맞게 리턴해주어야 합니다.

```python
# lambda_handler : 람다의 메인함수
def lambda_handler(event, context):
  
    event = ?	# : 첫번째 파라미터인 event가 lex가 lambda한테 전달한 Event 데이터입니다.
    			# : Input Event 라고도 합니다.
    
    return ? 	# : lambda가 lex한테 전달할 Event 데이터입니다.
  				# : Response 라고도 합니다.
```

### Input Event, Response의 형식

> Reference : [AWS 공식 문서](http://docs.aws.amazon.com/lex/latest/dg/lambda-input-response-format.html)
>
> 해당 형식은 언제든지 바뀔 수 있습니다. 이는 "messageVersion": "값" 에 좌우합니다.

```python
# Input Event Format

{
  "currentIntent": {
    "name": "intent-name",
    "slots": {
      "slot name": "value",
      "slot name": "value"
    },
    "slotDetails": {
      "slot name": {
        "resolutions" : [
          { "value": "resolved value" },
          { "value": "resolved value" }
        ],
        "originalValue": "original text"
      },
      "slot name": {
        "resolutions" : [
          { "value": "resolved value" },
          { "value": "resolved value" }
        ],
        "originalValue": "original text"
      }
    },
    "confirmationStatus": "None, Confirmed, or Denied (intent confirmation, if configured)"
  },
  "bot": {
    "name": "bot name",
    "alias": "bot alias",
    "version": "bot version"
  },
  "userId": "User ID specified in the POST request to Amazon Lex.",
  "inputTranscript": "Text used to process the request",
  "invocationSource": "FulfillmentCodeHook or DialogCodeHook",
  "outputDialogMode": "Text or Voice, based on ContentType request header in runtime API request",
  "messageVersion": "1.0",
  "sessionAttributes": { 
     "key": "value",
     "key": "value"
  },
  "requestAttributes": { 
     "key": "value",
     "key": "value"
  }
}
```

```python
# Response Format

{
    "sessionAttributes": {
    "key1": "value1",
    "key2": "value2"
    ...
  },
  "dialogAction": {
    "type": "ElicitIntent, ElicitSlot, ConfirmIntent, Delegate, or Close",
    Full structure based on the type field. See below for details.
  }
}
```

dialogAction 에 대한 자세한 구조는 공식 문서를 참고바랍니다. *우측 상단에서 언어를 선택할 수 있습니다.*

### Lex Input Event Echo 해보기

보다 쉬운 이해를 위해 Lex Input Event가 어떻게 전달되는지 눈으로 직접 확인할 수 있는 예제입니다.

```python
def lambda_handler(event, context):
    return {
        'sessionAttributes': '',
        'dialogAction': {
            'type': 'Close',
            'fulfillmentState' : 'Failed',
            'message' : {
                'contentType': 'PlainText',
                'content': str(event),
            }
        }
    }
```

위 코드를 Lambda 함수에 작성하고 Lex 콘솔에서 대화를 해보세요.

이제 Lex에 Lambda를 올릴 수 있게 되었습니다.

## Lambda에서 DynamoDB 호출하기

### DynamoDB 내부 구조

`AWS DynamoDB`는 NoSQL형태의 구조를 가지는 데이터베이스 서비스입니다. 때문에 sql 관련 지식 없이도 쉽게 사용할 수 있다는 장점이 있습니다. 구조는 다음과 같은 JSON 구조를 띕니다.

```json
{
  "Disease": {
    "S": "질병명"
  },
  "Symptoms": {
    "L": [
      {
        "S": "증상1"
      },
      {
        "S" : "증상2"
      }
    ]
  }
}
```

S는 String, L은 List 를 나타내며 `AWS DynamoDB 콘솔`에서 편리한 UI를 제공하므로 자료 입력시 마다 직접 위와 같이 하드 타이핑을 할 필요는 없습니다.

### DynamoDB 호출하기

위 Lex와 Lambda는 호출하고 리턴하는 관계(Caller와 Callee의 관계) 입니다. 그래서 별도의 스킬 없이 파라미터와 리턴 값을 이용하여 정보를 주고받게 됩니다. 그렇기 때문에 그 이외 제 3의 아마존 서비스를 사용하려면 별도의 기능이 필요합니다. 이러한 기능을 제공해주는 라이브러리를 `AWS SDK` 라고 부르며 Python에서는 `boto3`이라고도 부릅니다.

DynamoDB를 호출하기 위해서 먼저 `AWS DynamoDB`에서 테이블을 만들고, 데이터를 넣습니다. 다음 코드는 DB내용 전체를 Lex에서 출력하는 예제입니다.

```python
# import boto3
# Lambda에서는 boto3을 import할 필요가 없습니다. 자동으로 선언되어있습니다.

def call_my_dynamo_records():
        myddb = boto3.client('dynamodb')
        response = myddb.scan(
            TableName='테이블이름' # 생성한 테이블 이름으로 바꿔주어야 합니다.
        )
        # print(response["Items"][0])

        return response

def lambda_handler(event, context):
    db = call_my_dynamo_records()
    return {
        'sessionAttributes': '',
        'dialogAction': {
            'type': 'Close',
            'fulfillmentState' : 'Failed',
            'message' : {
                'contentType': 'PlainText',
                'content': str(db),
            }
        }
    }
```

### 권한 부여하기

위 boto3의 scan함수가 원활하게 작동하기 위해서는 해당 Lambda 함수의 권한에 DynamoDB 읽기 권한이 포함되어있어야 합니다. 위 Lambda를 정의할 때 DynamoDB에 대한 Access 권한을 주지 않았다면 Lambda가 가지는 Role(실행 역할)을 `AWS IAM`콘솔에서 수정합니다.

1. `AWS IAM`콘솔 접속

2. 좌측 `역할`탭 선택 및 우측에서 `해당 역할 선택` 

3. 인라인 정책 추가

4. 서비스 : DynamoDB

5. 작업 : 읽기 체크

   > Scan 함수만 사용한다면 Scan만 체크해도 무방합니다.

6. 권한 추가

## 만든 챗봇을 페이스북 메신저에서 사용해보기

> reference : [AWS 공식 문서](http://docs.aws.amazon.com/ko_kr/lex/latest/dg/fb-bot-association.html), [Facebook 공식 문서](https://developers.facebook.com/docs/messenger-platform/getting-started/app-setup#app_and_page_setup)

1. `페이스북 페이지`를 생성합니다.

2. [페이스북 개발자 페이지](https://developers.facebook.com/apps)에서 `앱 추가`를 합니다.

3. 페이스북에서 2가지 정보를 생성하고 받아야 합니다.

   1.  `대시보드` - `앱 시크릿 코드`
   2. `제품추가` - `Messenger` - `토큰 생성` - `페이지 액세스 토큰` 

4. `AWS Lex`의 `Channels`탭에서 `Facebook`을 클릭하고 작성합니다.

   > KMS Key : aws/lex
   >
   > Alias : Lex를 Publish했을 때 설정한 Alias
   >
   > Verify Token : 임의로 지정 가능하나, 꼭 기억하고 있어야 함.
   >
   > Page Access Token : 페이스북 페이지 액세스 토큰
   >
   > App Secret Key : 페이스북 앱 시크릿 키

5. 다시 `페이스북 개발자 페이지`로 돌아가 `Webhooks`를 설정합니다.

   1. 선택된 이벤트 : messages, messaging_postbacks, messaging_optins 체크
   2. Callback URL 추가
   3. Verify Token (AWS에서 작성한) 추가

6. `페이스북 페이지`로 돌아가서 `Messenger 플랫폼`탭의 `응답 방법`을 `자동`으로 설정합니다.

7. 하단 `앱 역할`을 `primary receiver`로 수정합니다.

8. 자신의 `페이스북 페이지`와 메신저로 대화해봅니다.

## 참고하기

### 세션속성의 자유로운 자료형 사용

세션속성은 key와 value로 매칭된 Map 구조로 이루어져 있고 value는 String이나 Int 같은 단일 자료형입니다. 때문에 tuple, list, dict 등을 사용하는 데 있어 제약을 받게 됩니다. 이때 `ast.literal_eval`을 사용하면 쉽게 다중 자료형을 세션으로 사용할 수 있습니다.

이 함수는 문자열로 된 파이썬 구문을 파이썬 구문으로 치환해주는 역할을 합니다.

아래 예제는 세션에 리스트를 만들어 대화를 한 번 할 때마다 `'item'`을 한 번씩 넣어주는 예제입니다.

```python
import ast

def lambda_handler(event, context):
    
    if 'my_list' in event['sessionAttributes'] :
    	my_list = ast.literal_eval(event['sessionAttributes']['my_list'])
    else :
        my_list = []
        # 이 자료형을 session에 넣을 때는 '[]' 와 같이 들어가야 합니다.
        # 이 문자열 '[]'를 literal_eval 함수로 [] 와 같이 리스트로 만들 수 있습니다.
        
    my_list.append('item')
    event['sessionAttributes']['my_list'] = str(my_list)
    
    return {
        'sessionAttributes': event['sessionAttributes'],
        'dialogAction': {
            'type': 'ElicitSlot',
            'message' : {
                'contentType': 'PlainText',
                'content': str(my_list),
            },
            'intentName' : event['currentIntent']['name'],
            'slots' : event['currentIntent']['slots'],
            'slotToElicit' : 'slot' # Lex에서 정의한 slot이름으로 바꿔주어야 합니다.
        }
    }
```

단, Lex가 읽는 Event 크기는 25K를 초과할 수 없으므로 이에 유의하여 대용량의 자료를 세션에 넣는 것은 피하시기 바랍니다.

### Cloud watch를 이용하여 로그 보기

1. CloudWatch 서비스에 들어갑니다.

2. 왼쪽 탭에 로그를 클릭합니다.

3. 로그 그룹 중 해당되는 그룹을 선택합니다.

4. 가장 위에 있는 (최신) 로그스트림을 확인합니다.

5. 스크롤을 내립니다.

   > 비정상 종료, SDK 외부 서비스 호출, print 함수 출력물 등을 확인할 수 있습니다.
   >
   > 바로 반영이 되지 않는 경우가 있으니 원하는 로그가 나올때까지 스크롤을 올렸다 내렸다 반복하세요.

### 에러에 관한 설명

#### An error has occurred: Invalid Lambda Response: Received error response from Lambda: Unhandled

- Event 데이터 형식에 맞지 않게 Response를 Lex에 return 할 경우
- Lambda 함수가 비정상으로 종료되어 Return 된 경우
  - TypeError, IndexError, etc...

#### An error has occurred: Invalid Lambda Response: Lambda response exceeds maximum size (27509 vs. 25K)

Response 길이가 25K 이상인 경우

### API 제한, 규칙 문서

써야댐

### 모르는 이슈에 대해서는?

예시 : https://stackoverflow.com/questions/45039251/amazon-lex-error-an-error-occurred-badrequestexception-when-calling-the-putin

​	boto3을 사용하고 있는 중 ?가 문제를 일으키는 경우.
​	주) aws console 과 lambda 간에는 이러한 문제가 발생하지 않음.

​		검색하고 문서에 기여해주세요



## 기여하기

> 문서의 질을 향상시키기 위해 기여하기(contribution)을 장려합니다.
>
> 틀린 정보가 있다면 피드백 부탁드립니다.

1. 문서기여 : 해당 프로젝트를 Fork하여 문서를 수정하고 Pull Request를 요청합니다.

   ​	[Github 문서 기여에 대해 더 자세히 알아보기](https://git-scm.com/book/ko/v2/GitHub-GitHub-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-%EA%B8%B0%EC%97%AC%ED%95%98%EA%B8%B0)

2. 응원 : 더 많은 사람이 볼 수 있게 Star를 누릅니다.

[기여자 목록
