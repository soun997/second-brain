
## 개요

자동으로 서버 프로젝트를 배포하기 위해 Jenkins Pipeline을 구축하고자 한다.


## 목적

#### PR 시, 서버 프로젝트를 빌드 및 Dev 서버에 배포
- PR이 올라갔을 때, 자동으로 테스트와 빌드를 수행하여 오류를 방지한다.
- Webhook을 통해 빌드 실행 결과를 Discord에 전송한다.

## Pipeline 스크립트

```groovy

```


## Ref

[Github PR을 올리면 자동으로 테스트가? 심지어 멀티 프로젝트에도 가능하다!](https://meetup.nhncloud.com/posts/212)

[Vue + Spring boot + Junit + Sonarqube 젠킨스 파이프라인 script 문법 예제 코드](https://pjs21s.github.io/jenkins-pipeline-script-grammar/)