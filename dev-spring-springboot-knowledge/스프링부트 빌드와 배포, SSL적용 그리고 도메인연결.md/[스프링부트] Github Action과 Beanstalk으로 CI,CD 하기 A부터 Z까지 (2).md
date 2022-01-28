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

https://gist.github.com/sooolog/adfa10c8a2bf29e82e4a8bb1522cd2f6

<script src="https://gist.github.com/sooolog/adfa10c8a2bf29e82e4a8bb1522cd2f6.js"></script>

https://gist.github.com/sooolog/adfa10c8a2bf29e82e4a8bb1522cd2f6.js

<p align="center">
<img src="">
</p>

ㅁ

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로



태그 : #





1.빈스톡 connect fail해서 한 경우도 인스턴스가 바뀌지않음
2.이상하게 그 디그레이드 되서 기다리게 된거 있잖아 배포는 됬는데, 그거는 EC2가 바뀌었는데 ? 직접해봄
3.혹시 여태 안됬던게, rest가 아닌 다른 정적 머스테치 연동은 안되니까, 받을게 없어서 에러가 난건가 ? 그래서 rest를 하니 된거
    아.. 우선은 rest가 없어서 그랬던건지 아니면 그냥 mustache가 /를 못받아서 그랬던건지 알아야 한다.(이거 이유도 알자 404 등등)
4.그리고 실제 port나(어플리케이션에서 server.prot하는거나, 아니면 빈스톡구성에서 PORT 5000하는거나) 이거도 영향을
    주는지 해보고, 보안그룹에서 80포트를 없애도 되는지 확인해보자.
5.그리고 실제 다른 브랜치에서 push해서 pull request해서 merge되는 경우는 깃헙액션이 반응하는지 보자. 안될거같은데
6.거기다가, 그냥 아무것도 커밋없이도 푸쉬되는지도 보자.    - 안됨
7.그 모냐 왜 t2.micro에서 잘되다가 잘 안되다가 하는지 알기 그리고 용량증가하면 더 잘되는거같다.
8.아 빌드하는거에 이미 테스트를 진행하고 빌드하는구나, 깃헙액션 보니 그렇다. + 
9.혹시나 /에 대한 맵핑이 되어야 (여태, /맵핑을 restcontroller만 함 되는건지 일반 컨트롤러로 /맵핑해봤다. 결과는 ? 와
    RestController가 있어서 된게 아니라, /맵핑자체를 통신을 못받아서 안된거였네. 404에러가 계속 뜨니 /에