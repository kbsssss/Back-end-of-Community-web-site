<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152286573-635c26c0-33a9-4413-861a-cbda2822d98a.png">
</p>

# 📖 [스프링부트] Github Action과 Beanstalk으로 CI/CD 하기 A부터 Z까 (4) [완]

* deploy.yml 작성 
* 00-makeFiles.config, Procfile 작성
* nginx.conf 작성

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *

<br>



### 1.build할 때 사용했던 deploy.yml을 빈스톡까지 배포하게 설정해 보자.

```yml
      - name: Generate deployment package 
        run: |
          mkdir -p deploy
          cp build/libs/*.jar deploy/application.jar
          cp Procfile deploy/Procfile
          cp -r .ebextensions deploy/.ebextensions
          cp -r .platform deploy/.platform
          cd deploy && zip -r deploy.zip .  # (1)

      - name: Beanstalk Deploy
        uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: dev-real-test
          environment_name: Devrealtest-env
          version_label: github-action-${{steps.current-time.outputs.formattedTime}}
          region: ap-northeast-2
          deployment_package: deploy/deploy.zip  # (2)
```

빈스톡으로 배포할 수 있는 방법은 많지만, 우리는 Github Action으로 빌드하고 배포까지 간편하게 하나의 스크립트에서
진행하기 위해서, 기존에 작성해 두었던 deploy.yml 뒤에 추가로 작성해준 코드이다. 이해를 돕기위해 deploy.yml의 전체코드를
첨부하도록 하겠다.

<br>

```yml
name: dev-spring-springboot

on:
  push:
    branches:
      - master
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Set up JDK 1.8
        uses: actions/setup-java@v1.4.3
        with:
          java-version: 1.8

      - name: Grant execute permission for gradlew
        run: chmod +x ./gradlew
        shell: bash

      - name: Build with Gradle
        run: ./gradlew clean build
        shell: bash

      - name: Get current time
        uses: 1466587594/get-current-time@v2
        id: current-time
        with:
          format: YYYY-MM-DDTHH-mm-ss
          utcOffset: "+09:00"

      - name: Show Current Time
        run: echo "CurrentTime=${{steps.current-time.outputs.formattedTime}}"
        shell: bash

      - name: Generate deployment package
        run: |
          mkdir -p deploy
          cp build/libs/*.jar deploy/application.jar
          cp Procfile deploy/Procfile
          cp -r .ebextensions deploy/.ebextensions
          cp -r .platform deploy/.platform
          cd deploy && zip -r deploy.zip .

      - name: Beanstalk Deploy
        uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: dev-real-test
          environment_name: Devrealtest-env
          version_label: github-action-${{steps.current-time.outputs.formattedTime}}
          region: ap-northeast-2
          deployment_package: deploy/deploy.zip
```

그렇다면, 이제 이번에 새로배울 코드들의 각 줄의 의미에 대해 알아 보도록 하겠다.

<br>

```yml
      - name: Generate deployment package 
        run: |
          mkdir -p deploy
          cp build/libs/*.jar deploy/application.jar
          cp Procfile deploy/Procfile
          cp -r .ebextensions deploy/.ebextensions
          cp -r .platform deploy/.platform
          cd deploy && zip -r deploy.zip .  # (1)
```

우선, # (1)에 해당하는 코드들을 보겠다.     
이전에 우리가 Build를 진행하여 만들어진 Jar파일을 Beanstalk으로 배포하기 위해서는 zip파로 만들어주어야 한다.      
(1). mkdir -p deploy는 deploy 디렉토리를 만들어준다.    

<br>

> mkdir뒤에 나오는 -p는 부모디렉토리까지 만들어주는 옵션이다. 즉, mkdir -p test/deploy라고 한다면
> 만약 test 디렉토리가 없다면, test 디렉토리와 deploy디렉토리를 한번에 생성시킨다는 의미이다. 그렇기에 위의
> 코드에서 mkdir deploy만 적어주어도 된다. 추가로, 그냥 mkdir test/deploy를 하면 오류가 난다고 한다.               
> [mkdir options](https://phoenixnap.com/kb/create-directory-linux-mkdir-command)

<br>

(2). cp build/libs/*.jar deploy/application.jar는 내가 build해준 jar파일을 (1)에서 만들어준 디렉토리로 application.jar이라는 파일명으로 바꿔서 복사해주는것이다. 
이렇게 해주는 이유는 빌드하고 난뒤의 jar파일명은 버전과 타임스탬프로 파일명이 변경되기 때문에, 이를 application.jar로 통일 시키고 뒤에 나올 00-makeFiles.config내에서
application.jar로 지정한 jar파일명으로 어플리케이션을 실행시키기 위함이다.

<br>

> 뒤에 00-makeFiles.config 파일에 대해서도 보겠지만, application.jar파일 대신에 application123.jar을 하고
> 위의 cp build/libs/*.jar deploy/application123.jar을 해도 정상적으로 작동한다. 그러나, cp build/libs/*.jar deploy/application123.jar으로
> 바꿔주고 00-makeFile.config는 그대로 application.jar이라 적으면 정상적으로 작동하지않는다.(=어플리케이션이 실행되지않아 배포가 되지 않는다.)

<br>

(3). cp Procfile deploy/Procfile도 디렉토리 deploy안에 복사해서 넣어주는것이다.    
(4). 나머지 cp -r .ebextensions~ 와 cp -r .platform~은 각각의 하위 디렉토리와 파일들 모두를 deploy/.ebextensions 그리고
deploy/.platform에 그대로 복사하라는 의미이다.

<br>

> cp -r은 cp의 옵션으로 해당하는 디렉토리의 하위디렉토리와 모든파일들까지 한번에 복사해서 옮겨놓는다는 의미이다.       
> [cp -r에 관하여](https://jframework.tistory.com/6)      

<br>

(5). 마지막으로 cd deploy && zip -r deploy.zip은 deploy디렉토리로 들어가서 해당하는 모든 디렉토리와 파일들을 압축하라는 의미이다.

<br>

> zip -r은 해당 위치에 있는 파일들은 물론이고 디렉터리까지 압축하라는 의미이다.    
> [zip -r에 관하여](https://www.lesstif.com/lpt/linux-zip-unzip-80248839.html)   

<br>

정리하자면, Procfile, 00-makeFiles.config, nginx.conf를 jar파일과 함께 압축해주어서 하나의 zip파일안에
담고자 하는것이다. 이렇게 하나의 zip파일에 jar파일과 다 함께 압축하는 이유는 우리는 스프링부트의 embedded tomcat 을 사용하면서 
.jar로 배포하는 경우에는 jar에 .ebextenions 폴더를 포함시킬 수 없기 때문에 따로 배포전에 jar 파일과 함께 .ebextenions 폴더를 
하나의 zip 파일로 묶어서 배포해야 제대로 설정파일이 적용되기 때문이다.

<br>

> [jar파일과 .ebextensions를 하나의 zip파일안에 두는 이유](https://techblog.woowahan.com/2539/)

<br>

> 추가로, deploy.yml은 github action runner에서만 사용되기에 같이 압축파일에 들어가지 않는것이다.

<br>

```yml
      - name: Beanstalk Deploy
        uses: einaregilsson/beanstalk-deploy@v20
        with:
          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          application_name: dev-real-test
          environment_name: Devrealtest-env
          version_label: github-action-${{steps.current-time.outputs.formattedTime}}
          region: ap-northeast-2
          deployment_package: deploy/deploy.zip # (2)
```

그 다음은, 압축한 파일을 빈스톡으로 배포하는것이다.     
여기서 자세히 보면, einaregilsson내에 있는 beanstalk-deploy의 버전20에 해당하는것을 사용한다고 나와있다.
이는 [Github Action 플러그인](https://github.com/marketplace/actions/beanstalk-deploy)이다. 
AWS CLI로 배포를 진행할 수도 있지만, 이렇게 스크립트안에 알집 압축과 배포까지 한번에 진행시키기 위해 사용한다. 링크를
클릭하고, Use latest Version을 클릭해보면      

<br>

```yml
- name: Beanstalk Deploy
  uses: einaregilsson/beanstalk-deploy@v20
```

와 같은 코드를 얻을 수 있다. 현재(글 작성일 기준으로) 최신버전은 v20이다.

<br>

> 위에 적어놓은 코드 그대로만 적용해주면 플러그인 사용이 가능한것이다. 따로 설치하거나 할 필요가 없다.

<br>

그 뒤로 나와있는 코드를 보겠다.

<br>

```yml
aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

우리가 기존에 받아놓았던, IAM 발급키이다. 여기서는 위의 코드처럼 적어주면 된다.
그러면, Github Action에서는 우리가 깃헙 레포지토리의 secrets로 등록한 secrets.AWS_ACCESS_KEY_ID 키와
secrets.AWS_SECRET_ACCESS_KEY 키를 불러와 각각 적용하게 된다.

<br>

> 앞에서 secrets.AWS_ACCESS_KEY_ID와 secrets.AWS_SECRET_ACCESS_KEY를 정확하게 입력해야 했던 이유가 여기서 적은 키워드와
> 동일해야지 Github Action에서 값을 매치시켜주기 때문이다.

<br>

```yml
application_name: dev-real-test
environment_name: Devrealtest-env
```

그 다음 코드는 갖고있는 IAM키로 접근할 빈스톡 환경을 지정해주는것이다. 보면, application name과
environment name을 적어주어서 어느 빈스톡 환경으로 연동할지를 설정하는것이다. 

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152470883-ec2f15c3-a2b7-4cb7-bf48-4c53c61effd8.png">
</p>

우리가 이전 글에서 만들어놓은 환경을 들어가 보면, 어플리케이션 이름이 dev-real-test이고, 환경이름이 Devrealtest-env라는걸 알 수 있다.
이 명칭들을 적어서 넣어주면 된다.

<br>

```yml
version_label: github-action-${{steps.current-time.outputs.formattedTime}}
```

그 다음 볼 코드는 version_label이다. version label은 말 그대로 어떠한 버전인지 라벨을 붙였다고 생각하면 된다. 우리가
이전 Github Action에서 빌드하는 글을 보면, Get Current Time에 대해 설명한적이 있다. 위의 Deploy.yml 풀코드를 보아도 알 수 있는데
빌드를 완료하고 난 후 시점을 확보하고 그 시점을 배포하려는 어플리케이션의 버전 라벨로 사용한다는 것이다. 즉, github-action-${{steps.current-time.outputs.formattedTime}}
코드는 github-action-빌드한시간 으로 버전 라벨로 사용한다는 의미이다.

<br>

> deploy.yml에서 Show Current Time에서도 이 ${{steps.current-time.outputs.formattedTime}} 코드를 사용하여 빌드된 시점을 보여줬었다.

<br>

```yml
region: ap-northeast-2
deployment_package: deploy/deploy.zip # (2)
```

나머지 코드들을 한번에 정리하도록 하겠다.     
region: ap-northeast-2부터 보면, ap-northeast-2는 서울 리전을 의미한다. 그렇다면 왜 리전을 적어주어야 하는걸까?
바로 빈스톡 환경이름 즉, 환경name은 각 리전별로 고유한 이름이여야 한다. 기본적으로 어플리케이션 내에 여러환경이 있을 수 있기에 어플리케이션은 중복해서
빈스톡 환경을 생성할 수는 있지만, 빈스톡 환경의 이름은 각 환경마다 고유한 이름을 갖어야 된다는것이다. 고유한 이름의 범위는 바로 region내로 한정짓기 때문에
접근하려고 하는 빈스톡이 어느 리전에 있는지 구체화해서 적어주어야 하는것이다.

<br>

> region이 다르면 중복된 빈스톡 환경이름으로 생성할 수 있다.

<br>

deployment_package는 우리가 이전에 빈스톡으로 서버에 배포하기위해 Jar파일을 압축하여 zip파일로 만들었다고 했다.
이 zip파일을 배포하겠다는 의미이다.

이로써, deploy.yml작성을 마무리하고 다른 설정파일들을 보도록 하겠다.

#### 🪁 Reference
* 참조링크 : [deploy.yml 작성하기](https://jojoldu.tistory.com/549)

<br>



### 2. 00-makeFiles.config, Procfile을 작성해보고 개념에 대해 알아보겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152473761-9c06c854-0f78-4014-9092-97e47dc72dc3.png">
</p>

00-makeFiles.config, Procfile를 작성하기전에 파일이 위치한 구조부터 보고가겠다.
보이는 바와같이, 00-makeFiles.config는 루트디렉토리에서 .ebextensions디렉토리를 만들어서 작성을 해주면 되고,
Procfile은 루트디렉토리에 바로 만들어 주면 된다.

<br>

```config
files:
    "/sbin/appstart" :
        mode: "000755"
        owner: webapp
        group: webapp
        content: |
            #!/usr/bin/env bash
            JAR_PATH=/var/app/current/application.jar

            # run app
            killall java
            java -Dfile.encoding=UTF-8 -jar $JAR_PATH
```

00-makeFiles.config파일부터 보도록 하겠다. 처음 드는 궁금증은 왜 .ebextensions라는 디렉토리안에 이 파일을 생성하느냐는 것이다.
그 이유는 바로 빈스톡은 EC2를 대신해서 생성해주기 때문에, 우리가 직접 EC2를 직접 생성할때처럼은 사용할 수가 없다. 
그래서 직접생성해서 사용했을때처럼 커스터마이즈를 하고싶을 때 이 .ebextensions라는 디렉토리아래에 설정파일을 두어서 적용시키는 것이다.

00-makeFile.config라는 .config 확장명을 갖은 파일안에  YAML이나 JSON형식으로 코드를 작성하면, 빈스톡에 배포시
해당 코드들이 사용이 되는것이다. 그럼, 이 .config라는 파일 확장자는 사실 크게 의미는 없지만 설정파일이라는 의미를 주기위해 .config로
적어주는것이고 그 안의 코드들은 YAML이나 JSON으로 적어주어 적용하게 되는것이다. 이 00-makeFiles.config 파일에서 00-xxx.config형태로
쓰는이유는, 파일의 나열이 alphabetical order 순서이기에, 적용 순서가 중요한대로 00,01로 붙여서 사용을 하는것이다.

<br>

>[.ebextension과 00-xxx.config파일명의 이해](https://techblog.woowahan.com/2539/)

<br>

이제 00-makeFiles.config안의 코드들에 대해 보도록 하겠다. 우선,
files로 시작을 하는데, 이는 파일을 만들거나 다운로드를 받게하는 코드이다. 여기서는
만드는 기능으로 적용이 된다. 

<br>

> [00-makeFiles.config 문법](https://techblog.woowahan.com/2539/)

<br>

우리가 배포한 zip파일이 압축이 풀리고, 어느 파일을 실행할지를 설정하는 어플리케이션 실행
스크립트를 생성하는것이다.

<br>

```config
    "/sbin/appstart" :
        mode: "000755"
        owner: webapp
        group: webapp
        content: |
            #!/usr/bin/env bash
            JAR_PATH=/var/app/current/application.jar

            # run app
            killall java
            java -Dfile.encoding=UTF-8 -jar $JAR_PATH
```

/sbin 아래에 스크립트 파일을 두면, 이는 전역에서 사용이 가능하기에 /sbin/appstart 라고 작성하여 appsatrt 스크립트 파일을 생성한다.
권한은 755를 갖으며, owner는 webapp이고, content를 갖은 스크립트 파일이 생성된다.

content안에 있는 JAR_PATH는 압축이 해제된 zip파일내에 있던 jar파일의 경로를 의미한다.

<br>

> 우리가 이전 글에서 build한 후 jar파일들과 여러 설정파일들을 압축해주는 과정에서 
> cp build/libs/*.jar deploy/application.jar 로 생성된 jar파일명을 application.jar로
> 통일해준것을 알 것이다. 이렇게 통일해 준 이유는 위의 JAR_PATH에서 jar파일명을 그대로 명시해주어야 하기 때문이다. 만약,
> build할 때 jar파일명을 application.jar로 통일해주지 않으면 JAR_PATH의 jar파일명도
> 매번 그에 맞게 수정해주어야 하기때문에 번거로워 진다. 그래서 앞선 글에서 JAR파일을 통일시켜준것이다.

<br>

그 아래, killall java는 기존에 실행중이던 어플리케이션을 종료하라는 의미이다. 하지만, 우리는
추가 배치를 이용한 롤링으로 새로운 인스턴스를 생성하여 배포해주고 기존의 인스턴스를 대체하는것이기 때문에
필요한 코드는 아니지만, 적어주어도 상관은 없다.

java -Dfile.encoding=UTF-8 -jar $JAR_PATH는 해당되는 Jar파일을 실행시킨다는
의미이며, -Dfile.encoding=UTF-8을 함으로써 파일 인코딩을 UTF-8로 실행하라는 의미이다. 

설정파일 00-makeFile.config로 인해 생성된 실행 스크립트의 content를 모두 보았다면,
이제 이렇게 생성한 appstart 스크립트 파일을 실행시키는 일만 남았다.
스크립트 파일의 실행은 Procfile에서 담당한다.

<br>

```text
web: appstart
```

Procfile에 작성한 내용이다.     
위에서 만든 어플리케이션 실행 스크립트를 실행시키는 일은 Procfile에서 하고
또, Procfile을 실행시켜야 Procfile이 작동하는 것이니 결국에는 배포된 어플리케이션의 실행은
Procfile을 실행시킨다는 의미와 같다.

#### 🪁 Reference
* 참조링크 : [00-mkaeFile.config, Procfile 작성](https://jojoldu.tistory.com/549)

<br>



### 3.마지막 nginx.conf 파일을 보도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152483197-fe279aef-46bc-4a51-8b4e-f5d4bd19c091.png">
</p>

Nginx는 무중단 배포로도 사용이 되지만, 여기서는 Nginx가 받은요청을 스프링부트 임베디드 톰캣으로
보내는 역활만 하는 리버스 프록시로 보도록 하겠다.

<br>

> 무중단 배포를 Nginx로 설정할 때도, 리버스 프록시 기능을 사용한다.

<br>

Nginx.conf내에 쓰는 코드들을 설명하기에 앞서, 빈스톡 환경에서 리버스 프록시로써의 역활에 대해
필수적으로 알아야 할 기본개념들을 보고 가도록 하겠다. 꼭 참고하기를 바란다.

<br>

> 빈스톡에서는 로드벨런서(Application Load Balancer)가 무중단 배포를 대신해서 수행한다.

<br>

> 리버스 프록시란, 내부망의 서버 앞단에서 요청을 처리해주는 서버를 의미한다. 조금 더
> 쉽게 설명하자면, EC2 인스턴스 내부에는 리버스 프록시 역활을 하는 Nginx가 있고, 우리가 배포한
> 스프링부트 프로젝트인 WAS(웹 어플리케이션 서버)가 존재한다. EC2 인스턴스로 오는 요청을 WAS가 직접받지않고
> 리버스프록시인 Nginx가 대신받은 후에 다시 이를 WAS로 보내는것이다.

> [nginx와 리버스 프록시란](https://juneyr.dev/nginx-basics)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152486577-aa3176b8-8786-4923-b70f-3dd1e98f9b91.png">
</p>

보는바와 같이 EC2내부에 Nginx가 먼저 요청을 받게된다. 그렇다면, 궁금증이 생긴다. 왜 바로 WAS에서 받지않고
번거롭게 리버스 프록시를 거쳐서 요청을 받을까 ?    

그 이유는 바로 보안때문이다. WAS는 대부분 DB서버와 연결 되어있기에, WAS가 최전방에 있으면 보안이 취약해진다. 그 때문에
리버스 프록시인 nginx를 앞 단에 두고 사용하는 것이다.

<br>

> [Nginx를 리버스 프록시로 사용하는 이유](https://juneyr.dev/nginx-basics)    
> [Nginx 그림 참조](https://stackoverflow.com/questions/54612962/502-bad-gateway-elastic-beanstalk-spring-boot)

<br>

그런데, 위 사진을 보면 뭔가 이질적인게 느껴진다. 로드벨런서로 부터 클라이언트의 요청을 Nginx가
받는것 까지는 이해가 되는데, 그 요청을 그대로 WAS에 보내는게 아니라 5000포트로 허공에 보내는걸 알 수 있다.
왜 이러는 걸까 ? 

Nginx는 빈스톡에서 사용될 때 별다른 설정을 해주지 않으면 받은 요청을 다시 5000포트로 보내게 설정되어있다.
즉, 정상적으로 작동하게 하려면 우리는 이를 8080포트(WAS로)로 보내주거나, 아니면 스프링부트 어플리케이션을 5000포트로
실행하여야 한다.

<br>

> 5000포트로 진행하는 방법에 대해서도 글 맨 마지막 추가부분에 잠깐 설명하겠지만, 필자는 8080포트로 진행하도록 하겠다.

<br>

스프링부트 어플리케이션은 8080포트에서 실행시키고, Nginx는 받아온 요청을 다시 8080포트인 WAS로 보내는
설정을 바로 .platform/nginx/nginx.conf 에 있는 nginx.conf에서 설정하여 사용한다.

<br>

```conf
user                    nginx;
error_log               /var/log/nginx/error.log warn;
pid                     /var/run/nginx.pid;
worker_processes        auto;
worker_rlimit_nofile    33282;

events {
    use epoll;
    worker_connections  1024;
    multi_accept on;
}

http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;

  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  include       conf.d/*.conf;

  map $http_upgrade $connection_upgrade {
      default     "upgrade";
  }

  upstream springboot {
    server 127.0.0.1:8080;
    keepalive 1024;
  }

  server {
      listen        80 default_server;
      listen        [::]:80 default_server;

      location / {
          proxy_pass          http://springboot;
          proxy_http_version  1.1;
          proxy_set_header    Connection          $connection_upgrade;
          proxy_set_header    Upgrade             $http_upgrade;

          proxy_set_header    Host                $host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
      }

      access_log    /var/log/nginx/access.log main;

      client_header_timeout 60;
      client_body_timeout   60;
      keepalive_timeout     60;
      gzip                  off;
      gzip_comp_level       4;

      # Include the Elastic Beanstalk generated locations
      include conf.d/elasticbeanstalk/healthd.conf;
  }
}
```

nginx.conf에 작성된 코드이다. 이제부터 각 문단이나 중요키워드를 중심으로
설명해 나가도록 할텐데, 그전에 부수적으로 알아야 할 개념들에 대해서도 설명하니, 천천히 따라오면
된다.

1. 먼저 nginx 설정파일의 문법에 대해서 보도록 하겠다.

* **Directives(지시어)** : nginx.conf처럼 설정파일에서는 디렉티브라는것으로 옵션설정을 한다. 즉,
위의 코드중에 user, error_log, pid, http, server모두 디렉티브이다. 또한, 디렉티브는 블록(혹은 컨텍스트)디렉티브와
심플 디렉티브로 나뉜다.
 
* **심플디렉티브** : 세미콜론(;)으로 끝나는 디렉티브이다. 위의 디렉티브중 USEr,error_log,pid가 심플디렉티브이다.
 
* **블록 디렉티브** : 세미콜론 대신에 중괄호({})로 끝난다. 또한, 블록 디렉티브는 중괄호 안에 다른 디렉티브를 가질 수 있다.
위의 디렉티브중 http, server가 블록 디렉티브이다..

<br>

> 블록은 컨텍스트라고도 불린다. 그렇기에 컨텍스트안에는 다른 디렉티브들이 있을 수 있는데, user, error_log, pid처럼
> 다른 컨텍스트에 속해있지않은 디렉티브들을 메인 컨텍스트안에 있는것으로 보며, server디렉티브는 http 컨텍스트안에
> 있는것으로 보면 된다.

<br>

> [심플 디렉티브와 블록(컨텍스트) 디렉티브 (1)](https://kscory.com/dev/nginx/install)      
> [심플 디렉티브와 블록(컨텍스트) 디렉티브 (2)](https://architectophile.tistory.com/12)

<br>

2. 다음으로, nginx.conf의 파일이 어디있는지 그리고 디렉터리 구조는 어떠한지 보도록 하겠다.
(모두 필요한 내용이니 가볍게 읽어보면 도움이 된다.)

nginx.conf파일은 기본적으로 /etc/nginx/ 폴더안에 위치하게 된다.

즉, 조금 더 풀어서 말하자면, 우리가 EC2인스턴스에 ssh로 접속하게 되면 제일 먼저 홈디렉토리에서 시작하게 된다.
그곳에서 명령어 cd /etc/enginx로 들어가보면 각종 nginx 설정파일들과 폴더들이 있고 거기에 nginx.conf파일도 있는것이다.
그 중에 nginx.conf파일이 가장 주요한 설정파일이다.

<br>

> 루트 디렉토리란, 최상위 디렉토리로 모든 디렉토리들의 시작점을 의미한다. 루트 디렉토리를 '/'로 표시한다.
> 홈 디렉토리란, '\~' 로 물결모양의 디렉토리명으로 표시된다. 통상 EC2에 ssh로 접속하면 시작하는 위치이다. 이
> 디렉토리는 최상위 디렉토리 '/' 하위의 home디렉토리 안에 사용자 디렉토리를 의미하며, ec2인스턴스는 단일 인스턴스이건
> 빈스톡을 이용한 인스턴스이건 ec2-user로 디렉토리명이 지정되어있다. 실제로 해당 홈 디렉토리를 들어가보면 디렉토리가
> '\~' 로 표시된것을 알 수 있다.    
> [루트 디렉토리 '/'와 홈 디렉토리'~'](https://dana-study-log.tistory.com/entry/Linux-%EB%A6%AC%EB%88%85%EC%8A%A4-%ED%8C%8C%EC%9D%BC-%EC%8B%9C%EC%8A%A4%ED%85%9C-%EA%B5%AC%EC%A1%B0-%EB%A3%A8%ED%8A%B8-%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC-%ED%99%88-%EB%94%94%EB%A0%89%ED%86%A0%EB%A6%AC)

<br>

> [nginx.conf 파일의 위치 (1)](https://kscory.com/dev/nginx/install)      
> [nginx.conf 파일의 위치 (2)](https://architectophile.tistory.com/12)

<br>

```conf
user                    nginx;
error_log               /var/log/nginx/error.log warn;
pid                     /var/run/nginx.pid;
worker_processes        auto;
worker_rlimit_nofile    33282;
```

(1).**user** : 프로세스가 실행되는 권한을 설정하는거다. root혹은 nginx와 같은 값을 지정할 수 있다.

(2). **error_log** : nginx에서 일어나는 에러 로그파일이 존재하는 곳이다. warn은 로그 레벨을 의미하며, 해당 로그레벨 이상만 기록한다. error로 변경할 수도 있다.

<br>

> 우리가 빈스톡으로 배포한 ec2인스턴스에 ssh접속하여 vim /var/log/nginx/error.log를 하면 nginx의 에러로그를 볼 수 있다.
> 그러나, 조금 더 편한 방법으로는 빈스톤 콘솔에서 로그를 클릭하고, 전체 로그를 요청해서 다운받은다음에, nginx폴더의 error텍스트를
> 열어보면 더 편하게 볼 수 있다.(같은 로그다.)

<br>

(3).**worker_processes** : nginx에서 몇 개의 워커 프로세스를 생성할 것인지를 지정하는 지시어이다. 1이면 모든 요청을 하나의
프로세스로 실행하겠다는 의미이다. CPU 멀티코어 시스템에서 1이면 하나의 코어만으로 요청을 처리하겠다는 의미이다. 보통 명시적으로 서버에
장착되어 있는 코어 수 만큼 할당하는 것이 보통이며, 그렇기에 auto로 주로 설정한다.

<br>

> EC2인스턴스는 인스턴스 종류마다 코어의 개수가 다르다. 그렇기에 auto로 설정하는 편이 좋다.
> auto는 사용가능한 CPU 코어를 자동탐지하여 적용해준다.

<br>

(4).**worker_rlimit_nofile** : 위의 워커 프로세스가 최대 열 수 있는 파일 수를 제한하는것이다.

<br>

추가로,    
1. nginx 프로세스는 마스터(master)와 워커(worker) 프로세스로 나뉜다. 우리가 설정한 프로세스는
워커 프로세스이며, user에서 지시한 값은 해당 워커 프로세스의 권한 지정이다. 만약 user에서 권한을 루트사용자(최고사용자)로
지정해놓았는데, 악의적인 사용자가 제어권을 갖게되면 최고 사용자의 권한으로 원격제어당하는것이기 때문에 루트사용자로 지정하지않고
nginx로 사용하는것이다.

2. nginx성능 튜닝에 관해서이다. 
통상 worker_processes는 auto로 설정하지만, 운영체제를 위해서 CPU core수의 10\~20%는 남겨두는경우가 있다.
예를 들면, 24core를 갖은 CPU라면 Nginx에서 사용할 코어수를 1\~2-정도 할당하고 2\~4개는 OS용으로 남겨두는것이다.
지금 당장은 적용할 필요는 없지만, 참고하도록하자.

<br>

> [user nginx의 의미 (1)](https://whatisthenext.tistory.com/123)    
> [user nginx의 의미 (2)](https://narup.tistory.com/209)    
> [nginx 에러 로그](https://prohannah.tistory.com/136)    
> [worker_processes의 개념 (1)](https://whatisthenext.tistory.com/123)    
> [worker_processes의 개념 (2)](https://narup.tistory.com/209)    
> [nginx에서 worker processes의 auto 의미](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)    
> [nginx 성능튜닝에 관한 이야기](https://couplewith.tistory.com/entry/%EA%BF%80%ED%8C%81-%EA%B3%A0%EC%84%B1%EB%8A%A5-Nginx%EB%A5%BC%EC%9C%84%ED%95%9C-%ED%8A%9C%EB%8B%9D-2-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%B2%98%EB%A6%AC%EB%9F%89-%EB%8A%98%EB%A6%AC%EA%B8%B0)

<br>

이후, pid는 필요시 nginx.org 공식홈에서 설명해주고 있으니,
궁금하다면 읽어보도록 하자.

<br>

```conf
events {
    use epoll;
    worker_connections  1024;
    multi_accept on;
}
```

(1).**user epoll** : Linux커널 2.6이상인경우에 쓰이는 효율적인 이벤트 처리 방식이다.

<br>

> FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0 및 MacOS에서 순차적인 처리를 위한 방식으로 kqueue가 사용되고 있으나
> Linux 2.6+ 이상에서 사용하는 효율적인 이벤트 처리 방식으로 epoll가 사용되고 있다. 여기서 말하는 순차적인 처리와 효율적인 이벤트
> 처리 방식을 multi_accept에서 다시 설명하도록 하겠다.

<br>

(2).**worker_connections** : 하나의 프로세스가 처리할 수 있는 동시 접속 최대 수를 의미한다. 
그렇기에, 한번에 받을 수 있는 요청은 worker_processess(프로세스 수) X worker_connections수로
정해진다. 통상 512, 1024를 기준으로 적어진다.

<br>

> 여기서 고려해야 할 사항은, 이 커넥션의 의미는 클라이언트와의 커넥션 뿐만이 아니라, 프록시 서버들간의 연결이나 아니면 다른 연결들에대해서도
> 포함한 총 커넥션의 수라는것이다. 또한, 실제 동시 연결 커넥션 수는 오픈 되는 파일의 최대값을 넘을 수 없다는 거다. 이 값은
> 위에서 설명한 worker_rlimit_nofile을 의미한다. 지금 당장은 고려하지 않아도 되지만, 서비스의 규모가 커지면 고려해야 할 부분이다.    
> [worker_connections의 추가내용](https://nginx.org/en/docs/ngx_core_module.html#worker_connections)

<br>

(3).**multi_accept** : 순차적으로 요청(커넥션)을 받지 않고 동시에 요청을 접수하는 방식이다. 디폴트값은 off이다.

<br>

> 위에 user가 epoll로 설정되어야만 사용할 수 있다. 만약 kqueue로 설정되어 있으면 해당 디렉티브(multi_accept on)는 무시되는데
> 그 이유는 kqueue방식은 애초에 커넥션들을 순차적으로 받아들여기 위해 사용되는 방식이기 때문이다.

<br>

> [use epoll, kqueue (1)](https://couplewith.tistory.com/entry/%EA%BF%80%ED%8C%81-%EA%B3%A0%EC%84%B1%EB%8A%A5-Nginx%EB%A5%BC%EC%9C%84%ED%95%9C-%ED%8A%9C%EB%8B%9D-2-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%B2%98%EB%A6%AC%EB%9F%89-%EB%8A%98%EB%A6%AC%EA%B8%B0)       
> [use epoll, kqueue (2)](https://nomaddream.tistory.com/19)   
> [worker_connections에 관하여 (1)](https://couplewith.tistory.com/entry/%EA%BF%80%ED%8C%81-%EA%B3%A0%EC%84%B1%EB%8A%A5-Nginx%EB%A5%BC%EC%9C%84%ED%95%9C-%ED%8A%9C%EB%8B%9D-2-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%B2%98%EB%A6%AC%EB%9F%89-%EB%8A%98%EB%A6%AC%EA%B8%B0)   
> [worker_connections에 관하여 (2)](https://nomaddream.tistory.com/19)   
> [worker_connections에 관하여 (3)](https://whatisthenext.tistory.com/123)   
> [multi_accept 개념 (1)](https://nginx.org/en/docs/ngx_core_module.html#worker_connections)    
> [multi_accept 개념 (2)](https://couplewith.tistory.com/entry/%EA%BF%80%ED%8C%81-%EA%B3%A0%EC%84%B1%EB%8A%A5-Nginx%EB%A5%BC%EC%9C%84%ED%95%9C-%ED%8A%9C%EB%8B%9D-2-%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4-%EC%B2%98%EB%A6%AC%EB%9F%89-%EB%8A%98%EB%A6%AC%EA%B8%B0)

<br>

```conf
http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;

  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  include       conf.d/*.conf;

  map $http_upgrade $connection_upgrade {
      default     "upgrade";
  }
}
```

http 블록은 웹 서버에 대한 동작을 설정하는 영역으로, server, location 블록을 포함한다.
또한, 여기서 선언된 값은 하위블록에 상속된다.

<br>

> http 블록을 여러개 생성하여 관리할 수 있지만, 권장사항은 아니다. http블록을 하나만 생성하여 이용하는것이
> 권장사항이다.     
> [http블록 갯수 권장사항](https://taewooblog.tistory.com/74)

<br>

> [http 블록에 관하여 (1)](https://prohannah.tistory.com/136)
> [http 블록에 관하여 (2)](https://taewooblog.tistory.com/74)
> [선언된 값에 대한 하위 블록의 상속](https://juneyr.dev/nginx-basics)

<br>

그 다음, include 디렉티브는 추가적인 설정파일(conf)을 포함시켜주거나 MIME 타입 목록을
지정하는데 사용된다.

  include       /etc/nginx/mime.types;    
  include       conf.d/*.conf;

이곳에서도, mime.types 파일을 읽어들이거나 *.conf로 설정파일을 불러오고있다.

<br>

> [include 지시어에 관하여 (1)](https://narup.tistory.com/209)    
> [include 지시어에 관하여 (2)](https://aimaster.tistory.com/11)    
> [mime.types에 관하여](https://kscory.com/dev/nginx/install)

<br>

default_type의 default_type  application/octet-stream;는 옥텟 스트림 기반의 http를 사용한다는 의미이며,
MIME 타입 설정이다.

<br>

> [default_type에 관하여 (1)](https://narup.tistory.com/209)    
> [default_type에 관하여 (2)](https://kscory.com/dev/nginx/install)

<br>

log_format은 nginx의 access 로그의 형식을 지정해준다.

<br>

> 뒤에 나오는 access_log의 로그 형식을 지정한다. 또한, 앞서 언급한 error로그의 형식에는
> 영향을 주지 않는다. 

<br>

```conf
  upstream springboot {
    server 127.0.0.1:8080;
    keepalive 1024;
  }
```

이제는 upstream 블록 디렉티브를 보겠다.    
upstream은 origin은 WAS 즉, 웹 어플리케이션 서버를 의미한다. nginx와 연결한 웹 어플리케이션 서버를 지정하는데
사용된다. 하위에 있는 server 지시어는 연결할 웹 어플리케이션 서버의 'IP주소(호스트주소):포트'로 지정해준다.

upstream은 여러개를 만들 수 있으며,a

<br>

> nginx서버를 가리키는것은 downstream에 해당한다.

<br>

> upstream 지시어 안에, server 지시어를 여러개 적어서 로드벨런싱으로써 작용하게 할 수도 있다.    
> upstream test{    
>         server 호스트주소:포트    
>         server 호스트주소:포트    
> }    
> 와 같이 써주며, 순서대로 요청을 돌아가며 처리해주는 라운드로빈 방식과 서버마다 가중치를 주고 가중ㅊ치가 높은곳 부터 부하를 보내는 가중치
> 라운드 로빈 방식이 있는데, 다른 설정이 없다면 라운드 로빈 방식으로 작동한다.       
> [엔진엑스로 로드벨런싱 이용하기](https://cantcoding.tistory.com/77)

<br>

> [upstream 지시어 (1)](https://narup.tistory.com/209)
> 







<br>

```conf
  server {
      listen        80 default_server;
      listen        [::]:80 default_server;

      access_log    /var/log/nginx/access.log main;

      client_header_timeout 60;
      client_body_timeout   60;
      keepalive_timeout     60;
      gzip                  off;
      gzip_comp_level       4;

      # Include the Elastic Beanstalk generated locations
      include conf.d/elasticbeanstalk/healthd.conf;
  }
```

다음은, server 블록 지시어다.    
server 블록의 역활을 간단히 말하면, 하나의 웹사이트를 선언하는 데 사용되며, 가상 호스팅(Virtual Host)의 개념이다.
아래 listen 지시어와 server_name지시어의 역활을 알아가면서 더 쉽게 이해해보도록 하겠다.

<br>

```conf
  server {
      listen         80 default_server;
      listen         [::]:80 default_server;
  }
```

(1).**listen지시어** : listen지시어 다음에는 포트번호가 온다. 예를 들어, "listen 80처럼 되어있으면
EC2에 들어온 요청중에 포트가 80번에 해당하는것을 받겠다." 라는 의미이다.(Nginx는 EC2 인스턴스 내부에 있다.)

바로 다음에는 default_server 인자가 나오는걸 볼 수 있다.
이 default_server는 프로토콜(http, https, ftp등) 별로 단 하나의 
server 블록에만 존재해야한다. 즉, listen의 값이 80인 server블록이 여러개 있다면
단 하나의 server블록내의 listen 80 지시어에 대해서만 default_server로 지시할 수 있다는것이다.

이 default_server의 기능은, 들어온 특정 포트(여기선 80으로 보자)에 대한 모든 요청이
real-test.com:80, real-test.net:80, www.real-test.com:80와 같은 형태로 요청이 들어온다 할때,
server_name이 매칭되는 도메인이 없는경우 해당 요청의 포트 즉, 여기서는 80번 포트에 대해 모든요청이
listen지시어 값이 80 default_server로 지정된 server블록에서 처리 하게 되는 것이다.    
(바로 아래 server_name 지시어에 대한 설명이 나와있다.)

<br>

> listen HostName 혹은 IP주소 / 포트(port)로 적게 되어있는데, 'HostName 혹은 IP주소'에 대해서는
> 언급하지 않고 가도록 하겠다.

<br>

> 만약, default_server가 따로 지정되어있지 않은경우 기본적으로 가장 먼저 정의된
> 특정 포트에 대한 server블록이 default_server로 지정이 된다.     
> [default_server의 자동 지정](http://i5on9i.blogspot.com/2016/01/nginx-server.html)

<br>

> listen [::]:80 와 같이 사용되면, 이는 IPv6형식의 요청을
> 처리한다는 의미이다.    
> [listen \[::\]:80에 관하여 (1)](https://architectophile.tistory.com/12)     
> [listen \[::\]:80에 관하여 (2)](https://swiftcoding.org/nginx-routing)    

<br>

```conf
  server {
      listen         80 default_server;
  }
```

또한, default_server가 명시된 server 블록의 경우, server_name 지시어를
써주던 써주지 않던 그대로 정상적으로 기능을 한다. 예를 들면, 위처럼 listen 80 default_server;로
지정해놓고, server_name지시어가 없거나 아니면 server_name 지시어 값이 real-test.com인 경우 여부에 상관없이
www.real-test.com:80, m.real-test.com:80, real-test.net:80, real-test.co.kr:80와 같은 요청이
오면 모두 listen 80 default_server; 가 있는 server에서 요청을 모두 받아간다.(이 경우에 server블록이 하나밖에
설정을 안한경우였다.)

<br>

> 위의 도메인들이 Route53으로 DNS 도메인설정을 했을때 얘기이다. 빈스톡 환경에서 
> Route53을 이용한 도메인 연결은 다음 글에서 설명할것이다.

<br>

> [listen의 개념](https://architectophile.tistory.com/12)
> [default_server의 개념 (1)](https://swiftcoding.org/nginx-routing)
> [default_server의 개념 (2)](https://architectophile.tistory.com/12)

<br>

```conf
  server {
      listen         80 default_server;
      listen         [::]:80 default_server;
      server_name    real-test.com   #추가된 지시어
  }
```

(2).**server_name지시어** : server_name지시어는 클라이언트가 특정 포트로 요청을 하되, 어느 도메인으로
요청을 했는지에 따라 매칭해주는 지시어이다. 예를 들면, listen 80; 이지만, server_name지시어의 값을 
real-test.com으로 했으면 클라이언트가 브라우저 주소창에 real-test.com으로 입력해야지 해당 server 블록
지시어로 요청이 매칭이 된다는 의미이다. 만약 www.real-test.com이나 혹은 real-test.net처럼 요청이 들어오면
server_name 지시어값과 달라서 해당 server 블록에는 매칭되지 않는다.

이를 조금 더 기능적으로 기술하자면,    
클라이언트의 요청(request)의 header에 명시된 도메인값이 server_name값과 일치하는 경우 server블록에 분기해준다는 의미이다.

<br>

> 여기서 server_name에 들어갈 수 있는 값은 하위도메인(서브도메인, ex)www.도메인.com, m.도메인.com)이나 최상위 도메인이
> 다른 도메인 ( ex)real-test.net, real-test.co.kr)이 들어갈 수 있다.

<br>

> server_name에 대한 개념을 보기위해 지시어를 넣어서 적어주었으나, 우리는 server_name지시어를 적지않고
> 배포하도록 하겠다. 뒤에 하위도메인에 대한 처리나 리다이렉팅을 위해서 필요한 개념이니 반드시 알고가자.

<br>

> [server_name의 개념 (1)](https://swiftcoding.org/nginx-routing)    
> [server_name의 개념 (2)](https://narup.tistory.com/209)      

<br>

```conf
  server {
      listen        80 default_server;
      listen        [::]:80 default_server;

      access_log    /var/log/nginx/access.log main;
  }
```

여기서는 access_log 지시어에 대해 보도록 하겠다.
위 access_log는 nginx로 들어오는 요청이 해당 server에서 받아서 처리할 경우
그에 대한 로그을 담을 파일의 저장위치를 지정해 주는것이다.

access_log 지시어 값의 맨뒤 main은 이 전 log_format  main
에서 log출력형식을 지정하는 지시어에서 main 이름(alias)으로 설정된 format
을 사용하겠다는 의미이다.

<br>

> 만약 server블록 내에서 access_log를 쓰지 않고, http블록 바로 하위나 아니면 루트 컨텍스트에서
> access_log 지시어를 쓰면 모든 server블록에 대한 로그가 한 파일에 담기게 된다. 그렇게 되면, 나중에 로그분석을
> 할 경우 어려움이 많아지니 server블록 단위로 access_log 지시어를 사용하여 각기 다른 파일에 저장해 주는게 좋다.

<br>

> [access_log 지시어에 관하여 (1)](https://kscory.com/dev/nginx/install)
> [access_log 지시어에 관하여 (2)](https://youngwonhan-family.tistory.com/93)

<br>

```conf
  server {
      listen        80 default_server;
      listen        [::]:80 default_server;

      access_log    /var/log/nginx/access.log main;

      client_header_timeout 60;
      client_body_timeout   60;
      keepalive_timeout     60;
      gzip                  off;
      gzip_comp_level       4;

      # Include the Elastic Beanstalk generated locations
      include conf.d/elasticbeanstalk/healthd.conf;
  }
```

나머지 코드들을 정리해 보도록 하겠다.



<br>

정리하자면,      
그렇기에, server블록은 하나의 웺사이트를 선언하는데 사용되며, server 블록이 여러개이면, 한대의 머신(호스트)에 여러 웹사이트를 서빙할 수
있게되는것이다.(server_name으로 여러 도메인을 지정할 수 있기때문) 여기서 호스트란 EC2로 보아도 되고, Nginx 웹서버로 볼 수도 있다.

이렇게, 실제로 호스트는 한대지만, 여러 웹사이트를 서빙하기에 마치 가상으로 호스트가 여러개 존재하는 것처럼
동작하게 되기에 이런 개념을 가상 호스트라고 한다. server 블록 자체가 가상 호스팅을 가능하게 하는것이다. 

<br>

> [server 블록이란 (1)](https://prohannah.tistory.com/136)    
> [server 블록이란 (2)](https://juneyr.dev/nginx-basics)

<br>aaaaaaaaaaa

다음, access log가 어디 저장될지 보이는건데, 이는 이 서버에 해당하는 거에 대해서만
로그를 남긴다. 이것도 상속의 개념aaa

<br>

server 블록 또한 여러개 만들 수 있는데, 그렇게되면 한대의 머신(=호스트)에
여러 웹사이트를 서빙할 수 있게 되어 실제로 호스트는 한대지만, 가상으로 마치 호스트가 여러개 존재하는 것처럼 동작하게 할 수 있기에 가상 호스트라고 한다.

조금 더 쉽게 말하자면, http 컨텐스트 내에 아래와 같이    

그리고 http 블록도 server, location이런거 끝내고 개념 한번 더 정리

```conf
server {
    server_name test123.com
}

server {
    server_name test456.com
}
```

<br>

> 위에는 적혀져 있지 않지만, server_name이라는 지시어를 server 컨텐스트내에 사용할 수 있다.
> 

<br>

```conf
      location / {
          proxy_pass          http://springboot;
          proxy_http_version  1.1;
          proxy_set_header    Connection          $connection_upgrade;
          proxy_set_header    Upgrade             $http_upgrade;

          proxy_set_header    Host                $host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
      }
```

마지막으로, location 블록 지시어를 보겠다.
a

<br>

아니 근데, 이거하고 디렉티브니 이런 기본 개념도 정리해야해, 블럭이니

(3). http 블록 : 웹서버에 대한 동작을 설정하는 영역으로 server블록과 location블록 그리고 upstream블록의 루트 블록이다. 여기서 선언된
값은 하위블록에 상속되어, 서버의 기본값이 된다.

(4). server블록과 location블록 : server블록은 하나의 웹사이트를 선언하는데 사용되며, 가상 호스팅의 개념이다. location블록은 server블록 내에서
특정 URL을 처리하는 방법을 정의한다. 

(5). upstream블록 : origin서버라고도 하며, 여기서는 WAS를 의미한다. nginx는 downstream에 해당한다.


<br>

> 위의 nginx.conf설정 그대로 빈스톡 환경 구성의 소프트웨어 환경 속성에서 PORT로 8080 혹은 
> 5000으로 설정하고 해도 정상적으로 작동한다.(특히, PORT 5000은 빈스톡 환경을 생성하자마자 자동으로 소프트웨어
> 환경 속성에 적혀져있는데, 위의 nginx.conf설정이 담긴 프로젝트를 배포하면 해당 환경 속성 PORT 5000이 없어진다.)      
> [PORT 8080 적용](https://stackoverflow.com/questions/54612962/502-bad-gateway-elastic-beanstalk-spring-boot)

<br>

```conf
user                    nginx;
error_log               /var/log/nginx/error.log warn;
pid                     /var/run/nginx.pid;
worker_processes        auto;
worker_rlimit_nofile    33282;

events {
    use epoll;
    worker_connections  1024;
    multi_accept on;
}

http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;

  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

  include       conf.d/*.conf;

  map $http_upgrade $connection_upgrade {
      default     "upgrade";
  }

  upstream springboot {
    server 127.0.0.1:8080;
    keepalive 1024;
  }
  server {
      listen 80;
      server_name celebmine.net www.celebmine.net;
      return 301 http://celebmine.com$request_uri;
  }

  server {
      listen 80;
      server_name celebmine.co.kr www.celebmine.co.kr;
      return 301 http://celebmine.com$request_uri;
  }

  server {
      listen        80 default_server;

      if ($host = www.celebmine.com) {
          return 301 http://celebmine.com$request_uri;
      }

      location / {
          proxy_pass          http://springboot;
          proxy_http_version  1.1;
          proxy_set_header    Connection          $connection_upgrade;
          proxy_set_header    Upgrade             $http_upgrade;

          proxy_set_header    Host                $host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
      }

      access_log    /var/log/nginx/access.log main;

      client_header_timeout 60;
      client_body_timeout   60;
      keepalive_timeout     60;
      gzip                  off;
      gzip_comp_level       4;

      # Include the Elastic Beanstalk generated locations
      include conf.d/elasticbeanstalk/healthd.conf;
  }
}
```

마지막으로 정리된 nginx.conf파일을 보자면 위와같이 적어주면 된다.

<br>

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152488115-f6e4b0d7-2953-4d84-ac7e-e122c588f229.png">
</p>

위에 보이는것처럼 Nginx가 80번포트로 받은 외부의 요청들을 5000포트로 그대로 요청을 보내며, 스프링부트 어플리케이션을 5000포트로 실행시키는 방법에 대해서도 보고 가도록
하겠다.

첫번째는, 스프링부트 설정파일인 application.properties 또는 application.yml에     
server.port:5000 (properties설정파일) 또는     
server:   
  port: 5000 (yml설정파일)    
와 같이 적어주면, 스프링부트 프로젝트가 실행될때 해당 5000포트에서 실행이 된다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152491989-a3c18aa9-7581-4be7-8cfc-2b1fe0b3f23e.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152515745-47e37589-6117-41b5-9e72-b54ad47e83ec.png">
</p>

두번째 방법은, 바로 빈스톡 환경 생성을 할 때 추가 옵션 구성에서 소프트웨어 편집을 누르고
그 안의 환경 속성에서 SERVER_PORT와 5000을 적어주고 적용하면, 이 또한 스프링부트 어플리케이션이 5000포트에서
실행되게된다.

<br>

> 그런데, properties나 yml설정파일을 사용하는것이나, 빈스톡 환경 구성에서 직접 SERVER_PORT를 5000으로 잡아주는 방식은
> 어디까지나, 위에 작성한 nginx.conf파일처럼 포트를 직접 설정해주지 않았을 때 이야기다. nginx.conf 파일도 그대로 사용하면서 바로 위의
> 설정들도 함께 사용하지 말길 바란다.

<br>

#### 🪁 Reference

* 참조링크 : [Nginx 포트 5000 그대로 사용하는 방법 (1)](https://browndwarf.tistory.com/66)    
* 참조링크 : [Nginx 포트 5000 그대로 사용하는 방법 (2)](https://stackoverflow.com/questions/54612962/502-bad-gateway-elastic-beanstalk-spring-boot)    
* 참조링크 : [Nginx 포트 5000 그대로 사용하는 방법 (3)](https://wky.kr/6)

<br>



태그 : #deploy.yml, #.ebextensions, #nginx.conf, #00-makeFiles.config, #Procfile, #하위블록으로 상속, # 



위에 있는거 로그 스토리지 s3까지 버켓생성하고 다시봐야 한다.

그거랑 ssl할때 보안그룹에 443 포트도 이해해야

그리고 보안그룹에 8080포트도 같이 정리해야한다.

용량에서 인스턴스 유형에 플릿의 구성해서 t2.micro랑 t2.small있는거 같이 다시 정리할 필요가 있다.
실제 이거 정확하게 봐야하는게 비용절감면에서도 좋을거같다.

배포 바익은 추가 롤링이 아닌, 변경 불가능에 대해서도 정리하자.

ㅁㅁㅁ 이거 그거도 해야해, 이거 배포하기 Version label 앞에서 시간확보한거 사용하는거
이거는 파일 작성할 때 말고 실제 빈스톡 환경에서 콘솔창 다룰때 상세하게 다루자.

그 443 ssl적용할때, nginx.conf파일은 물론이고 그 리스터 부분도 다시 정리해야해