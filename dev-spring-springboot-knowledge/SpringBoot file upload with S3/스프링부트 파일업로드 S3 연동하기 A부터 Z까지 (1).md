<p align="center">
<img src="">
</p>

# 🗝 스프링부트 파일업로드 S3 연동하기 A부터 Z까지 (1)

* IAM 설정
* S3 버켓 생성
* 

> 필자는 코드 및 개념설명시 핵심내용에 대해서만 설명합니다. 전 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)을 참조해주세요.

* * *



### 1.IAM설정부터 알아보도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147724488-d348165e-a543-42d8-9458-f9e239d232f4.png">
</p>

IAM사용자 이름을 만들어줍니다. 필자는 s3에 할당하는 권한을 설정할것임으로 s3에 기존 s3버켓 이름이 real-test였으므로
s3-real-test로 IAM이름을 만들어 주었다.

아래에 엑세스 키를 선택해준다. '암호 - AWS 관리 콘솔 엑세스'는 IAM 계정에 관련된것이므로 지금 볼 필요는 없다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147724499-47957921-f40b-470b-b676-16106ade09b9.png">
</p>

다음의 권한 설정에서 기존 정책 직접 연결을 선택한다음에, AmazonS3FullAcess를 선택해준다. 이것이 있어야 S3에 파일을 업로드할 수 있다.

그 외에 설정들에 대해서는 건들지말고 계속 다음으로 가주면, 액세스 키 ID와 비밀 엑세스 키 ID가 나온다.
이 키들을 갖고 S3에 접근할 수 있게 해주는것이다. IAM을 만들때만 보여지고 다시는 볼 수 없으니, 어딘가에 메모를
해놓아야 한다.

> 이 KEY값들이 노출되면, 다른 사람들도 마음데로 이용할 수 있기에 조심해야 한다. 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147725487-6e3a61b1-9d79-418f-9286-7c92f29fc61d.png">
</p>

해당 IAM 정책을 클릭하고, AmazonS3FullAccess를 클릭해보면 위와 같이 JSON 형태의 entity가 있고 Statement 내부에 Effect, Action, 
Reource 등이 허용범위가 미리 설정되어 있다. 만약에 직접 커스터마이징 하고 싶다면 해당 JSON의 형식을 IAM JSON policy reference를 참고하여 수정하면 된다.

#### 🪁 Reference
* 참조링크 : [IAM에 관하여 1](https://devlog-wjdrbs96.tistory.com/323)
* 참조링크 : [IAM에 관하여 2](https://bgpark.tistory.com/127)

<br>



### 2.S3 연동하기 위한 S3 버켓 생성 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147722815-c4f3aeeb-0a94-4464-b6c0-04b16b89f964.png">
</p>

S3 버켓 생성시에, 버킷 이름은 파티션별로 고유해야 한다. S3에서 파티션이란 리전을 의미한다.
아래 s3 버켓 이름생성규칙에 관한 aws 공식문서를 링크해놓았다. 이를 참고하면 되겠다. 또한, 정적호스팅을
s3로 하는 경우, 이 버킷이름과 도메인이름이 같아야 하지만, 우리는 파일업로드를 위해서 사용하는것이니 같게하지않아도 된다.
필자가 test-service로 했지만, 리전에서 중복이 된다고 real-test-service로 버켓이름을 변경해주었다. 

> [링크](https://yuda.dev/251) 이곳에서는 도메인명과 버켓이름을 갖게해주어선 안된다고 말한다. 참고용으로만 보도록 하자.

리전이란, S3 서비스 자체는 글로벌 성격이지만, 버킷 자체는 특정 지역을 기반으로 하기에 지정을 해주어야 한다. 즉, 내가 주로 서비스하는 지역으로
선택해주면 된다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147722820-dddb33f6-aa38-4699-affb-d0527ce79426.png">
</p>)

외부에서 파일을 접근할 수 있도록 퍼블릭 액세스 차단을 비활성화한다. 즉, 위의 이미지처럼 모든 퍼블릭 엑세스차단을 해제해주면된다.

주어진 설정외에는 모두 기본값으로 하고 버켓을 생성해준다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147730558-8d816f36-7f78-4bb3-823d-655b8ecd1055.png">
</p>)

추가로, 버켓을 생성하고 버켓을 클릭하고 권한을 들어가서 버켓 정책 편집을 눌러주면 위와같은 화면이 뜬다. 이 버켓정책을 사용하는 이유는
기본적으로는 버킷의 접근을 허용하지만 정책을 통해 권한을 막겠다라는 의미이다. 이는 나중에 필요할 때 더 자세히 보도록 하겠다.

#### 🪁 Reference
* 참조링크 : [S3리전에 관하](https://blog.naver.com/minza1215/222541017141) 
* 참조링크 : [버켓 정책 편집에 관하여 1](https://devlog-wjdrbs96.tistory.com/323)
* 참조링크 : [버켓 정책 편집에 관하여 2](https://victorydntmd.tistory.com/334)

<br>



### 3.다음은 스프링부트 프로젝트에서 의존성에 대해 설명하겠다.

(1)
```java
   dependencies {
       implementation 'org.springframework.cloud:spring-cloud-starter-aws:2.1.0.RELEASE'
   }
```

(2)
```java
repositories {
    maven { url 'https://repo.spring.io/libs-milestone'}
}

dependencies {
    compile('org.springframework.cloud:spring-cloud-starter-aws')
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework.cloud:spring-cloud-aws:2.0.0.RC2'
    }
}
```

(3)
```java
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-starter-aws'
}

dependencyManagement {
    imports {
        mavenBom 'org.springframework.cloud:spring-cloud-dependencies:Greenwich.RELEASE'
    }
}
```

보통, S3를 연동하기 위해 스프링부트에 의존성을 추가하면 위와 같은 세가지 경우를 볼 수 있다.   
<br>
(2)번 코드 spring-cloud-aws의 버전이 해당 코드를 쓰는 당시에 2.0.0.RC2가 최신버전이였고, 해당 의존성을 사용하려면 당시에는 원격저장소에 maven { url 'https://repo.spring.io/libs-milestone'}를 추가해서 받아와야
했기에, 해당 코드를 사용한것이다. 즉, 지금은 원격저장소에 maven { url 'https://repo.spring.io/libs-milestone'}를 작성하여 갖고올 필요가 없다.(mavenCentral()만으로 충분하다.)     
또한, 추가로 mavenBom 'org.springframework.cloud:spring-cloud-aws:2.0.0.RC2'에서 이 Bom의 의미를 먼저 알아야 하는데, 그러면 Pom에 관해서도 알아야 한다. 이 Pom 이란 메이븐의 Pom.xml을일파일 의미하며, 만약
메이븐에서 이 pom.xml에 dependencyManagement만을 명시하고 Packaging을 pom으로 해서 원격저장소에 배포하게되면 이 dependencyManagement만을 명시한 pom.xml을 bom파일이라 한다. 즉, bom은 pom의 일종이다.
'org.springframework.cloud:spring-cloud-aws:2.0.0.RC2'와 같은 코드를 쓰는이유는 maven { url 'https://repo.spring.io/libs-milestone'} 메이븐저장소에서 가져다 쓰기위함이며, 고전방식의 연장선인것이다.

(3)번 코드는 원래는 'org.springframework.cloud:spring-cloud-starter-aws'이 아닌, 'org.springframework.cloud:spring-cloud-aws-autoconfigure' 이였다. 여기서 dependencyManagement를 쓰는 이유는
spring cloud aws는 애초에 spring cloud의 버전에 의존하기에, spring cloud의 버전을 맞춰주면 spring cloud aws의 버전도 그와 맞게 맞춰지는거다. 실제로 'org.springframework.cloud:spring-cloud-aws-autoconfigure'이
아닌 'org.springframework.cloud:spring-cloud-starter-aws'로 코드를 바꿔서 했는데도, spring-cloud-starter-aws의 버전이 이 'org.springframework.cloud:spring-cloud-dependencies:Greenwich.RELEASE' 버전을
따라갔었다. 이렇게 하는 이유는 사실, 스프링부트 버전에 맞는 spring cloud 및, spring-cloud-aws를 사용하기 위함이다. 더 자세히 보자면, 스프링부트는 해당 버전에 맞는 spring cloud버전이 있는데, 구글 검색창에 spring cloud version
for spring boot 2.1.7(필자 버전)을 치면 스프링 공식홈에서 스프링부트 버전에 따른 spring cloud버전이 명시되어있다. 이에 맞는 버전으로 필자는 위의 (3)코드에서 Greenwich라고 적어서 적용한것이다. 그러면, spring-cloud-starter-aws
의 버전이 2.1.0으로 된것은 dependencies에서 확인할 수 있다.

결론을 들자면 (1)코드처럼 깔끔하게 쓰면된다. 대신에, (3)번 코드에서 배운것을 기반으로 자신의 스프링부트 버전에 해당하는 spring-cloud-starter-aws 버전을 맞게찾아서 의존모듈을 추가해주면된다.

#### 🪁 Reference
* 참조링크 : [(2)코드에 관해](https://jojoldu.tistory.com/300) 
* 참조링크 : [POM과 BOM에 관한설명 1](https://simongs.tistory.com/49)
* 참조링크 : [POM과 BOM에 관한설명 2](https://findmypiece.tistory.com/101)
* 참조링크 : [(3)코드에 관해](https://bgpark.tistory.com/127)

<br>



🚀 추가로
1. 이 devtools의 원래기능은 5가지이나 우리가 본것은 Property Defaults, Automatic Restart, live Reload이다. 템플릿의 캐쉬 설정이나 live Reload등을 사용하겠다 하는 모든 설정은 devtools 의존성을 추가함으로써 자동 설정되는 부분이니 건드리지 않아도된다.
2. Devtools의 작동원리의 개념에 대한것을 알고싶다면 아래 참조링크를 보도록 하자.   
[[SpringBoot Devtools 원리]](https://iksflow.tistory.com/57))