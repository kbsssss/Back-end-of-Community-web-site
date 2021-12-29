

# 🗝 스프링부트 파일업로드 S3 연동하기 A부터 Z까 (1)

* 
* 

> 필자는 코드 및 개념설명시 핵심내용에 대해서만 설명합니다. 자세한 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)을 참조해주세요.

* * *

### 1.S3를 연동하기 위해서는 S3 버켓 생성과, IAM설정부터 알아보도록 하겠다.



<br>

### 2.다음은 스프링부트 프로젝트에서 의존성에 대해 설명하겠다.

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

> 참조링크 : [(2)코드에 관해](https://jojoldu.tistory.com/300)
> 
> 참조링크 : [POM과 BOM에 관한설명 1](https://simongs.tistory.com/49)
>
> 참조링크 : [POM과 BOM에 관한설명 2](https://findmypiece.tistory.com/101)
>
> 참조링크 : [(3)코드에 관해](https://bgpark.tistory.com/127)

<br>

###3.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147551855-1ec80def-5a7f-46b0-83c8-d2f3c715b9a2.png">
</p>

ㅁ

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147551867-fec6c9ad-6842-4ceb-a9fd-e2943dd2d837.png">
</p>



### 3.



🚀 추가로
1. 이 devtools의 원래기능은 5가지이나 우리가 본것은 Property Defaults, Automatic Restart, live Reload이다. 템플릿의 캐쉬 설정이나 live Reload등을 사용하겠다 하는 모든 설정은 devtools 의존성을 추가함으로써 자동 설정되는 부분이니 건드리지 않아도된다.
2. Devtools의 작동원리의 개념에 대한것을 알고싶다면 아래 참조링크를 보도록 하자.   
[[SpringBoot Devtools 원리]](https://iksflow.tistory.com/57))


## 🪁 Reference
* [스프링부트 Devtools 설정1](https://velog.io/@bread_dd/Spring-Boot-Devtools)
* [스프링부트 Devtools 설정2](https://otrodevym.tistory.com/entry/spring-boot-설정하기-10-dev-tools-설정-및-테스트-소스)
* [스프링부트 Devtools 설정3](https://otrodevym.tistory.com/entry/spring-boot-설정하기-10-dev-tools-설정-및-테스트-소스)
* [스프링부트 시작하기 책](https://velog.io/@sooolog)