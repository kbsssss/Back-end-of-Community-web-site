<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151309696-7ab7e274-101f-4467-ba73-74c65cc872bf.png">
</p>

# 📖 [스프링부트] Github Action과 Beanstalk으로 CI/CD 하기 A부터 Z까지 (2)

* Github Action과 Beanstalk으로 CI/CD 진행
* 
* 

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *


    
### 1.스프링부트에서 CI/CD는 Github Action과 Beanstalk으로 진행하도록 한다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151473981-f0504fde-808a-4c1f-9cea-a887f2bb9ace.png">
</p>

기존에는 CI/CD 툴로 Travic Ci, Jenkins, AWS Code Deploy를 많이 사용했었다. 또는,      
Travis CI + AWS Code Deploy     
Travis CI + AWS Beanstalk   
와 같은 환경조합으로 CI/CD 파이프라인을 구축했다.

하지만, 대세가 Travis CI에서 Github Action으로 넘어가고있으며, Auto Scaling과 Load Balancer, 클라우드 워치 등을 한번에
관리할 수 있는 Beanstalk 또한 많이 사용되고 있기에 우리는 Github Action + Beanstalk으로 CI/CD 파이프라인을 구축해보도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151483421-031667e6-0d78-4a8e-94a6-12b933cbff47.png">
</p>

Github Action과 Beanstalk를 사용했을 경우의 기본 구조는 이러하다.

#### 🪁 References
* 참조링크 : [기존 CI/CD 툴 (1)](https://artist-developer.tistory.com/24)
* 참조링크 : [기존 CI/CD 툴 (2)](https://jojoldu.tistory.com/543)

<br>



### 2.Github Action으로 빌드하기 위한, deploy.yml 작성

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151495796-3c4e532a-bbb3-433e-939d-6aa87ccfddee.png">
</p>

Github repository의 action에서 템플릿을 만들어서 해당 코드들을 적용해도 되지만, 우리는 프로젝트내에서 deploy.yml을 작성하여 
진행해 보도록 하겠다.

위의 그림처럼 루트 디렉토리에서 .github/workflows/deploy.yml을 만들어 준다. 저처럼 .github 폴더에 아이콘이 뜨길
원한다면, [extra-icons 플러그인](https://plugins.jetbrains.com/plugin/11058-extra-icons)을 적용해주면 된다.
이 외에도 인텔리제이 아이콘 플러그인 검색해서 원하는것을 설치하고 적용해 주면 된다. 

> 필자는 CI/CD 관련 코드들을 프로젝트 내에서 작성하여 한번에 관리하는것을 선호한다. Github에서 작성하여 적용하는
> 방법도 있으니, 궁금하신분들은 찾아보셔서 한번쯤 공부해보는것도 좋다.

```yml
name: dev-spring-springboot

on:
  push:
    branches:
      - master # (1).브랜치 이름
  workflow_dispatch: (2).수동 실행

jobs:
  build:
    runs-on: ubuntu-latest # (3).OS환경

    steps:
      - name: Checkout
        uses: actions/checkout@v2 # (4).코드 check out

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1.4.3
        with:
          java-version: 1.8 # (5).자바 설치

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
        shell: bash # (6).권한 부여

      - name: Build with Gradle
        run: ./gradlew clean build
        shell: bash # (7).build 시작

      - name: Get current time
        uses: 1466587594/get-current-time@v2
        id: current-time
        with:
          format: YYYY-MM-DDTHH-mm-ss # (1)
          utcOffset: "+09:00"작 # (8).build 시점의 시간확보

      - name: Show Current Time
        run: echo "CurrentTime=${{steps.current-time.outputs.formattedTime}}" # (2)
        shell: bash # (9).확보한 시간 보여주기
```

Github Action으로 빌드를 하기 위해서는 위의 코드들을 deploy.yml에 작성해주면 된다. 그럼 각각의 문단의 코드들이
어떠한 의미가 담긴것인지 보도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151495800-119b0984-2334-454d-8c84-0996c28d9495.png">
</p>

name은 위의 사진처럼 프로젝트의 해당 코드를 push하면 해당하는 깃헙의 repository에서 메뉴 목록에 Actions를 클릭하게 되면 나오는
화면의 글자를 의미한다.

조금 더 쉽게 말하자면, Commit했을때 설명 + deploy.yml의 name + 내가 여태 Push한 workflow갯수 로 표시되는거다.
필자는 commit할때 github action build라고 commit message를 적었기에 이와같이 나온거다.

> 즉, 이 name은 Repo Action 탭에 나타나는 이름으로 실제 workflow 진행창에는 나오지 않는다. 그렇기에
> build나 배포에 영향을 주는것은 아니고 통상 필자처럼 프로젝트명으로 입력하거나 원하는 nmae을 입력하면 된다.  

이제 나머지 #(1) 부터 #(9) 까지의 개념들에 대해 설명하도록 하겠다.

##### (1).브랜치 이름
```yml
on:
  push:
    branches:
      - master # (1).브랜치 이름
```

이 'master'가 들어가는 자리가 branch의 이름인데, 해당하는 branch 이름으로 push가 진행된다면
Github Action이 시작된다는 의미이다. 즉, Github Action 트리거 브랜치인것이다. master로 적어놓았지만,
다른 브랜치에서 Github Action을 실행시키고 싶다면 해당하는 branch 명을 바꾸어도 된다.

> 당연히, 지정한 브랜치외에 다른 명을 갖은 브랜치에서 push를 하게되고, 레포지토리에 pull request되어
> merge가 진행이 되었다고 해도, Github Action은 실행되지 않는다. 추가로 push외에 pull request나
> 다른 이벤트에 대해서도 트리거를 작동시키게 할 수 있다.      
> [push외에 다른 이벤트에 대한 Github Action 작동](https://hwasurr.io/git-github/github-actions/)
> [pull request에 대한 트리거 설정 코드](https://stalker5217.netlify.app/devops/github-action-aws-ci-cd-1/)

##### (2).수동 실행
```yml
on:
  workflow_dispatch: # (2).수동 실행
```

위의 push이벤트외에도 깃헙액션을 수동으로 실행하는것도 가능하게 하는 옵션이다.


##### (3).OS 환경
```yml
jobs:
  build:
    runs-on: ubuntu-latest # (3).OS환경
```
a

##### (4).코드 check out
```yml
jobs:
  build:
    steps:
      - name: Checkout
        uses: actions/checkout@v2 # (4).코드 check out
```
a

##### (5).자바 설치
```yml
jobs:
  build:
    steps:
      - name: Set up JDK 1.8
        uses: actions/setup-java@v1.4.3
        with:
          java-version: 1.8 # (5).자바 설치
```
a

##### (6).권한 부여
```yml
jobs:
  build:
    steps:
      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
        shell: bash # (6).권한 부여
```
a

##### (7).build 시작
```yml
jobs:
  build:
    steps:
      - name: Build with Gradle
        run: ./gradlew clean build
        shell: bash # (7).build 시작    
```
a

##### (8).build 시점의 시간 확보
```yml
jobs:
  build:
    steps:
      - name: Get current time
        uses: 1466587594/get-current-time@v2
        id: current-time
        with:
          format: YYYY-MM-DDTHH-mm-ss # (1)
          utcOffset: "+09:00"작 # (8).build 시점의 시간확보
```
a

##### (9).확보한 시간 보여주기
```yml
jobs:
  build:
    steps:
      - name: Show Current Time
        run: echo "CurrentTime=${{steps.current-time.outputs.formattedTime}}" # (2)
        shell: bash # (9).확보한 시간 보여주기
```
a

#### 🪁 References
* 참조링크 : [](https://jojoldu.tistory.com/543)
* 참조링크 : []()

<br>



### 🚀 추가로

깃헙액션의 코어 개념에 대해 간단하게 정리하도록 하겠다.    

**Workflow** : 자동화된 전체 프로세스로. 하나 이상의 Job으로 구성되고, Event에 의해 예약되거나 트리거될 수 있는 자동화된 절차를 말한다. Workflow 파일은 YAML으로 작성되고, Github Repository의 .github/workflows 폴더 아래에 저장된다. Github에게 YAML 파일로 정의한 자동화 동작을 전달하면, Github Actions는 해당 파일을 기반으로 그대로 실행시킨다.    

**Event** : Workflow를 트리거(실행)하는 특정 활동이나 규칙. 예를 들어, 누군가가 커밋을 리포지토리에 푸시하거나 풀 요청이 생성 될 때 GitHub에서 활동이 시작될 수 있다.   

**Job** : Job은 여러 Step으로 구성되고, 단일 가상 환경에서 실행된다. 다른 Job에 의존 관계를 가질 수도 있고, 독립적으로 병렬로 실행될 수도 있다.   

**Step** : Job 안에서 순차적으로 실행되는 프로세스 단위. step에서 명령을 내리거나, action을 실행할 수 있다.   

**Action** : job을 구성하기 위한 step들의 조합으로 구성된 독립적인 명령. workflow의 가장 작은 빌드 단위이다. workflow에서 action을 사용하기 위해서는 action이 step을 포함해야 한다.   

**Runner** : Gitbub Action Runner 어플리케이션이 설치된 머신으로, Workflow가 실행될 인스턴스이다.      

#### 🪁 References
* 참조링크 : [깃헙액션의 코어개념](https://velog.io/@ggong/Github-Action%EC%97%90-%EB%8C%80%ED%95%9C-%EC%86%8C%EA%B0%9C%EC%99%80-%EC%82%AC%EC%9A%A9%EB%B2%95)

<br>

태그 : #Github Action, #Benastalk, #깃헙액션 코어개념, #extra-icons 플러그인, #deploy.yml, 





1.빈스톡 connect fail해서 한 경우도 인스턴스가 바뀌지않음
2.이상하게 그 디그레이드 되서 기다리게 된거 있잖아 배포는 됬는데, 그거는 EC2가 바뀌었는데 ? 직접해봄
3.혹시 여태 안됬던게, rest가 아닌 다른 정적 머스테치 연동은 안되니까, 받을게 없어서 에러가 난건가 ? 그래서 rest를 하니 된거
    아.. 우선은 rest가 없어서 그랬던건지 아니면 그냥 mustache가 /를 못받아서 그랬던건지 알아야 한다.(이거 이유도 알자 404 등등)
4.그리고 실제 port나(어플리케이션에서 server.prot하는거나, 아니면 빈스톡구성에서 PORT 5000하는거나) 이거도 영향을
    주는지 해보고, 보안그룹에서 80포트를 없애도 되는지 확인해보자.

6.거기다가, 그냥 아무것도 커밋없이도 푸쉬되는지도 보자.    - 안됨
7.그 모냐 왜 t2.micro에서 잘되다가 잘 안되다가 하는지 알기 그리고 용량증가하면 더 잘되는거같다.
8.아 빌드하는거에 이미 테스트를 진행하고 빌드하는구나, 깃헙액션 보니 그렇다. + 
9.혹시나 /에 대한 맵핑이 되어야 (여태, /맵핑을 restcontroller만 함 되는건지 일반 컨트롤러로 /맵핑해봤다. 결과는 ? 와
    RestController가 있어서 된게 아니라, /맵핑자체를 통신을 못받아서 안된거였네. 404에러가 계속 뜨니 /에 실제, /ho는
    레스트 해놓고 /와 /hohoho를 일반 컨트롤러 했는데 그 계속 오류 됬던거로 뜸
10.보면, 정상적으로 배포완료되도 degraded됬다고 나와 1 out of 2 instances에서 버전이 맞지 않다고 근데, 이건
    조졸두 블로그에도 똑같이 나오는데 이거 해결하고 가자.