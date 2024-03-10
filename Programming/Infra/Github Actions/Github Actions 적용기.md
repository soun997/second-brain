## 서론

Jenkins에서 Github Actions로의 마이그레이션 과정 중 했던 삽질들을 기록하고자 한다.


## Github Actions 적용하기

![[Pasted image 20240310053621.png]]

아무것도 설정하지 않은 초기 상태에서 repo의 `Actions` 탭에 들어가면 다음과 같은 화면을 볼 수 있다.

처음에는 아무것도 모르고 `Publish Java Package with Gradle`을 눌렀었는데, **`Java with Gradle`을 선택해주어야 한다!**
- 둘의 차이는 차차 알아가는 중이다^^

![[Pasted image 20240310053822.png]]

그럼 이렇게 IDE가 뜨는데, 저기에 적혀진 코드가 Github Actions가 돌아갈 때 수행할 Workflow이다.
Jenkins의 파이프라인이라고 보면 될 것 같다.

Spring의 `application.yml` 파일과 비슷하게 `.yaml` 파일을 작성해주면 된다.
- 들여쓰기로 계층 구조를 표현한다.

대략적으로 살펴보겠다.
- `on`: 해당 워크플로가 어떤 상황에서 트리거 되는지 설정할 수 있다.
	- `push`, `pull_request` 등을 설정해줄 수 있다.
	- `branches`를 통해 대상 브랜치들도 설정해줄 수 있다.
- `jobs`: 어떤 작업들을 실행할 지 적는다. (ex. Build, Deploy)
- `steps`: 해당 작업에서 실행할 것들을 나열한다. (ex. Build 작업이라면 gradlew 권한 세팅, gradlew build 등...)
	- `name`: `steps`에서 진행할 작업 한 개의 이름을 정의한다.
	- `run`: 해당 `name`에서 실행할 스크립트를 작성한다.

초반에는 `build`와 `dependency-submission` 두 가지의 작업이 정의되어 있다.

이미 템플릿이 만들어져 있으니 일단 commit! 차차 수정하면서 구조를 파악해보고자 한다.

> 사용하면서 알게 되었는데, Github Actions는 해당 브랜치의 gradle.xml 파일에 맞게 워크플로를 실행하는 것 같다. main과 develop의 gradle.xml 파일의 내용이 다르다면, 둘은 다른 워크플로를 실행한다.

### 첫 시도는 실패!

![[Pasted image 20240310061303.png]]

`gradlew` 실행파일에 권한이 없기 때문에 나는 오류였다.

`gradle.xml` 파일에서 `gradlew` 실행 전에 해당 단계를 추가해주면 된다.
```
- name: Grant execute permission for gradlew  
  run: chmod +x gradlew
```

### 두번째 시도도 실패!

![[Pasted image 20240310061440.png]]

테스트에 실패했다.
프로젝트에 세팅되어 있던 환경변수를 넣어주지 않아 우르르 실패한 것이다!

### 환경변수 세팅하기

Github repo의 `settings` 탭에 들어가서 환경변수를 추가해 줄 수 있다.
자세한 방법은 아래에 기술되어 있다.
ref) https://docs.github.com/ko/actions/security-guides/using-secrets-in-github-actions

이제 `gradle.xml` 파일에 환경변수를 주입하면 된다.
```
- name: Build with Gradle 
  run: ./gradlew build 
  env: YOUR_VARIABLE: ${{ secrets.YOUR_VARIABLE }}$
```

![[Pasted image 20240310063901.png]]

CI까지는 성공!


## EC2로 자동배포

작성한 스크립트들은 Github에서 제공하는 VM에서 실행이 된다.
나의 EC2로 배포하기 위해서는 VM에서 SSH 연결을 통해 EC2로 접속해야 한다.
- 배포를 위해 Code Deploy 대신 직접 Docker Hub에 업로드하는 방식을 사용하기로 했다. (익숙하고 더 잘 사용할 수 있는 것을 골랐다!)

흐름은 대충 이러하다.
1. `gradlew build` 후 build된 결과물을 Docker Image로 빌드한다.
2. Docker Hub에 Image를 push 한다.
3. SSH 연결을 통해 내 EC2에 접속한다.
4. 기존에 실행되고 있는 컨테이너가 있다면 내린다. (Docker Down)
5. Docker Compose로 Image를 실행한다. (Docker Up)
6. 리소스를 정리한다.
7. 서버 헬스체크!

### SSH 연결을 어떻게 하지?

참고를 위해 검색해봤던 대부분의 블로그들은 라이브러리를 사용하여 손쉽게 SSH 연결을 했다.
- 라이브러리: https://github.com/appleboy/ssh-action

그러나 나는 Third-Party 라이브러리를 사용하는데에 있어 주의가 필요하다고 생각한다.
- 만약 라이브러리가 없어진다면?, 라이브러리에 의존하는 코드는 지양할 필요가 있다.
- 특히, SSH 연결은 bash 스크립트로도 충분히 가능할 정도이기 때문에 더더욱 사용할 필요성을 느끼지 못했다.

또한, `appleboy/ssh-action` 같은 경우에는 `scripts`에 실행할 커맨드들을 모두 작성해야 한다는 점이 불편하게 느껴졌다.
+) Docker를 사용하기 때문에 bash 스크립트보다 느릴 수밖에 없다!
ref) https://blog.benoitblanchon.fr/github-action-run-ssh-commands/

어쨌던, 워크플로에서 직접 `ssh` 명령어를 사용하기로 결정했다.

위의 블로그에서 너무 설명을 잘 해주어서 이를 차용했다.

```yaml
- name: Configure SSH  
  run: |  
    echo "Configure SSH"  
    mkdir -p ~/.ssh/  
    echo "$SSH_PRIVATE_KEY" > ~/.ssh/veganlife-dev.key
    chmod 600 ~/.ssh/veganlife-dev.key  
    cat >>~/.ssh/config <<END  
    Host veganlife-dev  
      HostName $SSH_HOST  
      User $SSH_USERNAME  
      IdentityFile ~/.ssh/veganlife-dev.key  
      StrictHostKeyChecking no  
    END  
  env:  
    SSH_HOST: ${{ secrets.SSH_HOST }}  
    SSH_USERNAME: ${{ secrets.SSH_USERNAME }}  
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

SSH 클라이언트(VM)가 특정 호스트에 대해 사용할 연결 설정을 정의한 것이다. 
하나하나 알아보자!

```yaml 
echo "$SSH_PRIVATE_KEY" > ~/.ssh/veganlife-dev.key
```

1. `.ssh` 폴더 아래에 EC2의 private key의 내용을 `.key` 파일로 생성한다.
2. 해당 파일의 권한을 600으로 바꾸어 줘야 로그인이 가능하다! (보안을 위해)

```yaml
cat >>~/.ssh/config <<END  
Host veganlife-dev  
	HostName $SSH_HOST  
    User $SSH_USERNAME  
    IdentityFile ~/.ssh/veganlife-dev.key  
    StrictHostKeyChecking no  
END  
```

1. `~/.ssh/config` 파일에 아래의 텍스트를 추가한다. `END`로 끝나는 줄까지 입력한 텍스트는 `config` 파일에 기록된다.
2. 호스트 설정 정보(호스트명, 사용자 이름, 개인키 파일 경로)를 입력한다. `END`로 입력의 끝임을 명시한다.

이제 `ssh veganlife-dev`와 같이 사용하여 간편하게 EC2 호스트에 연결할 수 있다.

### Docker Compose

이제 끝이 보인다...!
SSH에 접속하여 명령어를 실행하고 싶다면
```bash
ssh [호스트설정이름] 'docker compose -f veganlife-dev/docker-compose.yml down --rmi all'
```
와 같이 사용하면 된다.

아래처럼 Docker Down & Up 명령을 실행하도록 했다.
```yaml
- name: Docker Compose Down  
  # home 기준 실행파일의 상대경로를 적어주어야 한다.  
  run: |  
    echo "Docker Compose Down"  
    ssh veganlife-dev 'docker compose -f veganlife-dev/docker-compose.yml down --rmi all'  
  
- name: Docker Compose Up  
  run: |  
    echo "Docker Compose Up"  
    ssh veganlife-dev 'docker compose -f veganlife-dev/docker-compose.yml up -d'
```


## 배포 성공!

![[Pasted image 20240310194338.png]]

![[Pasted image 20240310194309.png]]

마이그레이션이 끝났다...
이제 Jenkins에게 작별인사할 시간이다.


## 추가적으로 할 것들

1. Build 성공, 실패 시 Discord에 알람이 가도록 설정
2. Deploy 성공, 실패 시 Discord에 알람이 가도록 설정
