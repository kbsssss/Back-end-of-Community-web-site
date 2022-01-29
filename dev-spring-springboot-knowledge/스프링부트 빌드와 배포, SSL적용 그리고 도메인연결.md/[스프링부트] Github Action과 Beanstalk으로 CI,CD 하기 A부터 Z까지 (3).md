<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151650030-15d1cafa-0f30-4803-8ea7-ecbc60d5a596.png">
</p>

# 📖 [스프링부트] Github Action과 Beanstalk으로 CI/CD 하기 A부터 Z까지 (3)

* 
* 

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *



<br>

### 1.Benastalk 사용을 위한 환경 생성하기

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652653-19bdcb04-9460-44d8-a580-4c82b28c4dab.png">
</p>

빈스톡 사용을 위한 환경생성을 진행해 본다. aws 검색창에 beanstalk를 치면 Elastic Beanstalk가 나온다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652649-d30182dd-8d28-411e-bb01-146cd2bd741e.png">
</p>

그 후 환경생성을 누르고 나면, 바로 환경 티어 선택이라는 창이 나온다.      
웹 서버 환경은 기존 EC2 환경을 그대로 사용하는것으로 일반적인 웹 서버를 사용하려 할 때 선택해주면 된다. 즉, 우리도
웹 서버 환경을 선택해서 진행해주면 된다.  

그렇다면, 작업자 환경은 무엇일까?   
웹 서버 환경에 메세지 큐 (SQS) 수신이 가능한 리스너 데몬이 별도 구축된 환경으로
보통의 HTTP API 처리가 아닌 프로세스들(배치/메시지워커 등)을 위한 환경이다. 

<br>

> 작업자 환경은 후에 SQS를 구축하여 서버를 사용하고자 할 때 다시보면 된다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652647-09aa657b-819d-4a6b-b7aa-b55243c3cadb.png">
</p>

애플리케이션의 이름은 필자는 보통 프로젝트의 이름과 같게한다. 여기서는 연습용 환경구축이니 필자는
real-test라 했지만, 이 글을 읽는 독자분들은 프로젝트 이름으로 써주거나 원하는 이름으로 어플리케이션 이름을
적어주면 된다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652640-306072d1-c9e1-476a-96a9-82cd6d8a57f8.png">
</p>

1개의 환경은 1개의 빈스톡이라고 보면된다. 규모가 큰 프로젝트의 경우 하나의 어플리케이션에 여러 환경을 구축하여 사용하기에
보통은 환경과 어플리케이션을 구분지어 놓는다. 그렇기에, Realtest-env나 Realtest-env1로 써주도록 하자. 도메인은 후에 빈스톡을
사용하여 ec2에 배포하는 경우, 배포 내용을 볼 수 있는 임시 도메인으로 봐도 된다.

도메인은 아무것도 채워넣지않아도 자동으로 채워지니 비워두고 그 다음 진행하도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652638-da74cfa1-514b-47a1-adb7-fec6d9bcf6bf.png">
</p>

Corretto는 AWS에서 지원하는 무료로 사용할 수 있는 Open Java Development Kit(OpenJDK)의 멀티플랫폼 배포판이다.
즉, Corretto 8은 자바 8기반의 멀티플랫폼 배포판인것이다. 일반 자바 8 SDK도 선택해서 사용할 수 있지만, deprecated된다고 되어있으니
Corretto 8을 이용하도록 하겠다. 

<br>

> 필자는 자바8로 프로젝트를 진행하지만 자바11이 익숙한 독자는 Corretto 11이나 자바11로 진행해주면 된다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652657-a8011dd1-fe08-4382-bfd2-922866e35c8c.png">
</p>

샘플 어플리케이션을 선택하여 진행하도록 하겠다. 그 아래 코드 업로드는 직접 프로젝트 파일을 jar파일이나 zip파일로 build된것을 직접
서버에 업로드하여 배포하겠다는 의미이다. 우리는 샘플 어플리케이션을 선택하겠다. 

> 다음은, 추가 옵션 구성을 눌러준다. 환경 생성을 눌러주면 안된다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652656-6388ba27-7a14-4ed5-8c2e-85aab42353c9.png">
</p>

추가 옵션 구성을 클릭하면 맨위에 이런 화면이 뜬다.

>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652654-930b5456-1cad-4618-8a91-d8b1299c3cbe.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652858-5c340932-670f-4101-bb06-0fbaf32c2db0.png">
</p>

ㅁ 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151653188-2dca1ecb-e073-4116-a3a1-7267c6980143.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652651-f4faaf49-e88d-4579-8618-4adddc127e70.png">
</p>

ㅁ 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652648-f148a032-45c6-4d2c-a967-cbebdc2074c2.png">
</p>

ㅁ 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151653123-e9ef2992-9a68-4438-bcf9-37be3d1ee574.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652644-1038fd6b-2571-4091-b63c-d58ae2bba0e3.png">
</p>

ㅁ 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151653127-cad86afc-717e-48e7-9f6d-62658823591c.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652857-16a1dbe5-99a7-4077-8646-5476c536d4f0.png">
</p>

ㅁ 

#### 🪁 Reference
* 참조링크 : [빈스톡 환경 구성하기 (1)](https://jojoldu.tistory.com/549)
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="">
</p>

ㅁㅁㅁ 이거 그거도 해야해, 이거 배포하기 Version label 앞에서 시간확보한거 사용하는거

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #




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
    
11.탄력적 ip관련해서도 정리해야한다. 로드벨런서에서 하는건데, 

12.rds도 연결해보고 있는데, rds추가할때 무엇이 문제가 될때 빈스톡이 연결이 안되는건지 보도록 하겠다.
    (1). 