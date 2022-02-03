<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152286573-635c26c0-33a9-4413-861a-cbda2822d98a.png">
</p>

# 📖 [스프링부트] Github Action과 Beanstalk으로 CI/CD 하기 A부터 Z까 (4) [완]

* deploy.yml, 00-makeFiles.config, nginx.conf, Procfile의 작성
* 

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

```yml
- name: Beanstalk Deploy
  uses: einaregilsson/beanstalk-deploy@v20
```

와 같은 코드를 얻을 수 있다. 현재(글 작성일 기준으로) 최신버전은 v20이다.

<br>

> 위에 적어놓은 코드 그대로만 적용해주면 플러그인 사용이 가능한것이다. 따로 설치하거나 할 필요가 없다.

<br>

그 뒤로 나와있는 코드를 보겠다.

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
environment name을 @@@@

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152080729-0e3fcf8b-e5b1-4cc3-a771-2deda9e99a99.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152080729-0e3fcf8b-e5b1-4cc3-a771-2deda9e99a99.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152080729-0e3fcf8b-e5b1-4cc3-a771-2deda9e99a99.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152080729-0e3fcf8b-e5b1-4cc3-a771-2deda9e99a99.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152080729-0e3fcf8b-e5b1-4cc3-a771-2deda9e99a99.png">
</p>



#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="">
</p>

ㅁ

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #


위에 있는거 로그 스토리지 s3까지 버켓생성하고 다시봐야 한다.

그거랑 ssl할때 보안그룹에 443 포트도 이해해야

그리고 보안그룹에 8080포트도 같이 정리해야한다.

용량에서 인스턴스 유형에 플릿의 구성해서 t2.micro랑 t2.small있는거 같이 다시 정리할 필요가 있다.
실제 이거 정확하게 봐야하는게 비용절감면에서도 좋을거같다.

배포 바익은 추가 롤링이 아닌, 변경 불가능에 대해서도 정리하자.

ㅁㅁㅁ 이거 그거도 해야해, 이거 배포하기 Version label 앞에서 시간확보한거 사용하는거