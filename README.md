# AWS - Lex와 Lambda, DynamoDB를 이용한 챗봇 구현 안내서

> AWS 기반 지식 없이 AWS로 시작하는 챗봇 구현



##### AWS 기반 챗봇을 만드는 여러가지 방법

- `AWS Lex`로 구현하고 AWS에서 제공하는 대응 플랫폼(페이스북 메신저, 트윌리오 등)을 이용하여 배포하기
- `AWS Lex`로 구현하고 `AWS Gateway API`를 이용하여 커스텀 UI(웹, 앱을 직접 제작하기 등)로 배포하기
- `AWS Lambda`를 이용하여 대화 흐름을 Programming 하기
- `AWS Lambda`와 DB (`AWS DynamoDB`, `AWS RDS` 등)을 이용하여 원하는 데이터와 함께 Programming 하기
- 등등 ...

> 이 안내서는 간단하면서도 많은 일을 할 수 있는 **AWS Lex, Lambda, DynamoDB를 이용하고 페이스북 메신저에 배포하는 과정**에 대해 설명합니다.
>
> Lambda는 Python, Java, Node.js 를 지원하며 이 안내서는 **Python 3.6**을 사용하였습니다.



##### 더 좋은 퀄리티를 가진 안내서들

- AWS의 커피 봇
- AWS 공식 문서
  - Lex, Lambda, DynamoDB, SDK, ...
- 등등 ...



## 목차

1. [Lex만 가지고 챗봇 만들기](#Lex)
2. [Lex에 Lambda를 더해보기](#Lambda)
3. [Lambda에서 DynamoDB 호출하기](#Lex와 Lambda의 연동)
4. [만든 챗봇을 페이스북 메신저에 배포하기](#페이스북 메신저에 배포하기)
5. [참고하기](#참고하기)
6. [기여하기](#기여하기)




## Lex만 가지고 챗봇 만들기

먼저 인식할 때 뭘 원하는지 파악하고 데이터를 출력해줌. speech recognition

어터런스를 좀 넣으면 비슷한 말 다 인식

에러처리도 쉽게 가능

웹에 building System, 페이스북 API, 기타 챗서비스 API에 연동중

## Lex에 Lambda를 더해보기

Lambda는 serverless, 함수 죽엇다 살앗다 함. python java node.js 함. python3.6 으로 설명함

메인함수는 머다

event, response event구성은 머다

렉스와 람다 echo를 해보고 이해해보자

권한주기

## Lambda에서 DynamoDB 호출하기

lex와 lambda는 호출하고 리턴하는 관계

새로운 AWS 기능을 사용하기 위해서는 다른 방법이 필요하다. 이게 바로 aws sdk

python : boto3

java : 

node.js : 

권한주기

## 만든 챗봇을 페이스북 메신저에 배포하기

따악.

## 참고하기

### 세션사용 팁

​	ast.literal_eval

### Cloud watch를 이용하여 로그 보기

1. CloudWatch 서비스에 들어갑니다.

2. 왼쪽 탭에 로그를 클릭합니다.

3. 로그 그룹 중 해당되는 그룹을 선택합니다.

4. 가장 위에 있는 (최신) 로그스트림을 확인합니다.

5. 스크롤을 내립니다.

   > 비정상 종료, SDK 외부 서비스 호출, print 함수 출력물 등을 확인할 수 있습니다.
   >
   > 바로 반영이 되지 않는 경우가 있으니 원하는 로그가 나올때까지 스크롤을 올렸다 내렸다 반복하세요.

### IAM USER 사용하기

### 에러 (형식에 맞지안흔거, 비정상종료, 리턴 잘못)

​	An error has occurred: Invalid Lambda Response: Received error response from Lambda: Unhandled

​	An error has occurred: Invalid Lambda Response: Lambda response exceeds maximum size (27509 vs. 25K)

### API 제한, 규칙 문서

### 모르는 이슈에 대해서는?

​	https://stackoverflow.com/questions/45039251/amazon-lex-error-an-error-occurred-badrequestexception-when-calling-the-putin

​	boto3을 사용하고 있는 중 ?가 문제를 일으키는 경우.
​	주) aws console 과 lambda 간에는 이러한 문제가 발생하지 않음.

​		검색하고 문서에 기여해주세요



## 기여하기

> 문서의 질을 향상시키기 위해 기여하기(contribution)을 장려합니다.

1. 문서기여 : 해당 프로젝트를 Fork하여 문서를 수정하고 Pull Request를 요청합니다.

   ​	[Github 문서 기여에 대해 더 자세히 알아보기](https://git-scm.com/book/ko/v2/GitHub-GitHub-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EC%97%90-%EA%B8%B0%EC%97%AC%ED%95%98%EA%B8%B0)

2. 응원 : 더 많은 사람이 볼 수 있게 Star를 누릅니다.

[기여자 목록
