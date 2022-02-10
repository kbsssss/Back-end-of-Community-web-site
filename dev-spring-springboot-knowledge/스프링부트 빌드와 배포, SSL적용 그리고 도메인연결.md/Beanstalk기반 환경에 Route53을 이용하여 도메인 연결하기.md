<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153342965-a7f7fcc2-96ba-467f-aae8-e22fdb515e43.png">
</p>

# 📖 Beanstalk기반 환경에 Route53을 이용하여 도메인 연결하기

* route53, 호스팅 영역 생성
* 

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *

<br>



### 1.호스팅영역인 route53 생성하기

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153199300-6e5cd361-0d96-4d1b-a0ec-2ca8df85d3e2.png">
</p>

aws 검색란에 route53을 검색하면 위와같은 화면이 나타난다. 우리는 목록에서 호스팅 영역을 클릭해주고, 호스팅 영역을 생성해준다.

<br>

> 여기서 알고가야 할 용어가 있는데, 우리는 지금 DNS 시스템으로 route53을 사용하는거다. DNS는 Domain Name System으 요청받은 도메인 이름을
> 네트워크 주소로 바꾸어주거나, 그 반대의 변환을 시행해주는 시스템이다. 또한, 호스팅영역은 레코드들의 컨테이너이며 레코드에는 특정 도메인과 그 하위 도메인의 트래픽을
> 라우팅하는 방식에 대한 정보가 담겨있다. 뒤에 자세히 설명해놓았으니, 순서대로 읽으면 더 잘 이해가 될것이다.

<br>

> [route53과 DNS](https://wwlee94.github.io/category/blog/aws-eb-https/)    
> [호스팅영역이란](https://docs.aws.amazon.com/ko_kr/Route53/latest/DeveloperGuide/hosted-zones-working-with.html)     

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153199317-ab8976b0-7aef-4d70-b52e-c248b5967f5d.png">
</p>

도메인 이름은 내가 구매한 도메인이나 무료로 발급받은 도메인명을 적어주면 된다.
필자는 test.com이라고 적어놓았지만, 독자분들은 갖고있는 도메인명을 적어주면 된다.

<br>

> 아래 설명 부분은 이름이 같은 호스팅영역을 구분하기 위한것으로 지금 당장은 필요하지않다.
> 또한, 프라이빗 호스팅 영역은 Amazon VPC영역 내에서 트래픽을 라우팅 하는 방식을 의미하기에
> 우리는 퍼블릭 호스팅 영역을 선택해준다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153199323-adea2639-a93f-4604-94a1-1b75245751d7.png">
</p>

호스팅 영역 생성을 클릭해주자.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153206006-94323c3e-79de-4dba-9ffb-378f5bb3ff28.png">
</p>

생성한 호스팅 영역을 들어가보면 위와같은 화면이 나온다.
레코드타입을 보면 NS라고 나오는데, Name Server을 의미하며, DNS(Domain Name Server)와 의미가 같다.
타입이 NS인 부분에 총 4개의 값이 등록되어 있어서 라우팅 대상이 4개나 되는데, 그 이유는 라우팅 대상의 네임서버가
한개밖에 안되고 그 서버가 죽으면 클라이언트들이 우리의 서비스에 들어올 수 없기에 4개까지 지정해준것이다. 이
네임서버를 통해 사용자가 요청을 했을 때 ip정보를 반환해준다.

다음 설명들을 보면 조금 더 쉽게 이해할 수 있다.

<br>

> 위에서는 DNS가 Domain name System이라 했는데, 기본적인 의미는 같으며 S가 server인 DNS는 말 그대로 
> 도메인 이름을 네트워크 주소로 바꿔주거나 그 반대의 역활을 해주는 서버자체를 의미하며, S가 System인 DNS는 
> 그 시스템을 의미한다.

<br>

> 도메인이 이름이 real-test.com인 이유는 test.com은 만들수가 없어서 real-test.com으로
> 도메인명을 입력하여 호스팅 영역을 생성해줬다.

<br>

> [NS타입의 역활과 개념](https://insight-bgh.tistory.com/261)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153209229-b84834fe-ba4b-4159-8c9b-24a8409f5a54.png">
</p>

위 화면은 필자가 구메한 도메인에서 네임서버를 설정하는 부분이다. 네임서버를 설정하는 곳 에서 우리가 방금 위에서
봤던 aws의 4개의 네임서버값을 호스트명에 입력해야, 클라이언트들이 브라우저에서 도메인명을 입력하였을때 aws의 네임서버가
정상적으로 요청을 받을 수 있는것이다.

<br>

> 필자는 가비아를 사용했지만, 도메인을 구입하건 아니면 무료로 발급을 받아도 네임서버를 설정하는 부분이 있다. 

<br>

> 호스트명이란, IP주소를 대신해서 사용할 수 있는 이름이다.    
> [호스트명, hostname이란](https://www.google.com/search?q=%ED%98%B8%EC%8A%A4%ED%8A%B8%EB%AA%85&rlz=1C5CHFA_enKR982KR982&oq=%ED%98%B8%EC%8A%A4%ED%8A%B8%EB%AA%85&aqs=chrome..69i57j35i39j69i59j46i131i433i512j0i3j69i60j69i61l2.1952j0j7&sourceid=chrome&ie=UTF-8)

<br>

> 자세히 보면 1차,2차,3차는 호스트명은 마지막에 '.'이 찍혀있지 않지만, 4차는 찍혀져있다. 호스팅영역의 NS 값을
> 그대로 복붙해오면 '.'도 같이 복사되니 이 점을 지워주어야 한다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153206006-94323c3e-79de-4dba-9ffb-378f5bb3ff28.png">
</p>

이번에는 레코드타입 SOA를 보겠다. SOA는 도메인과 관련된 타이머 설정부분이며 SOA 레코드가 등록되지않을 경우
다른 레코드들을 등록 할 수 없다.

SOA레코드와 NS레코드는 호스팅영역을 생성해주면 AWS에서 자동으로 생성해주는 값들이다.

<br>

> [SOA 레코드에 관하여 (1)](https://server-talk.tistory.com/176)
> [SOA 레코드에 관하여 (2)](https://insight-bgh.tistory.com/261)

<br>

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153199317-ab8976b0-7aef-4d70-b52e-c248b5967f5d.png">
</p>

ㅁ

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #