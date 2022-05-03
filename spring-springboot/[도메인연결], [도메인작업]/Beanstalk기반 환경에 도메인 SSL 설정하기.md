<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166184112-67d8e456-bad2-46c0-8df5-01528b404275.png">
</p>

# 📖 Beanstalk기반 환경에 도메인 SSL 설정하기

* SSL인증서와 
* ACM에서 SSL 인증서 발급받기
* Beanstalk 환경에 ACM 인증서 등록하기

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 2.ACM에서 SSL인증서 발급받기

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397052-138ae7c6-df2a-4216-8ce9-2ccbe0f6c833.png">
</p>

AWS의 ACM(AWS Certificate Manager)을 접속해보면 위와같은 화면이 등장한다. 
이곳에서 우리는 SSL 인증서를 요청하고 발급받아와서 

<br>



### 2.ACM에서 SSL인증서 발급받기

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397052-138ae7c6-df2a-4216-8ce9-2ccbe0f6c833.png">
</p>

AWS의 ACM(AWS Certificate Manager)을 접속해보면 위와같은 화면이 등장한다. 
이곳에서 우리는 SSL 인증서를 요청하고 발급받아와서 

<br>

> Amazon ACM에서 발급해주는 SSL인증서는 무료인 대신에, 해당 도메인의 DNS 서버로 Route53을 사용해야 한다. 그러니 route53을
> 이용하여 미리 도메인을 연결해놓자. 아직 연결해놓지 않았다면 필자가 적어놓은 [Beanstalk기반 환경에 Route53을 이용하여 도메인 연결](https://sooolog.dev/Beanstalk%EA%B8%B0%EB%B0%98-%ED%99%98%EA%B2%BD%EC%97%90-Route53%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0/)
> 을 참고하도록 하자.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397059-6fbb60b0-7218-46ba-98ca-1e875424a81e.png">
</p>

ㅁ

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397062-a6b3fdc5-1e59-4989-a321-4d3a990be7f5.png">
</p>

ㅁ

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397063-4a7ee18e-682e-4312-8cc9-33fe2d67cbd6.png">
</p>

ㅁ

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397064-73c66903-59da-4f85-b845-b7987d46fb3a.png">
</p>

ㅁ

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397067-80d1c598-ae7a-40a0-9c3b-3777afdf07bb.png">
</p>

ㅁ

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397069-7d05d580-cd76-421a-89e2-d43bd7acceb2.png">
</p>

ㅁ

<br>

지금 이거 보니 listen 443을 안하고 그냥 지금있는 그대로 쓰면
불안정한 https연결이 될수도 있다는데 ?
http://linforum.kr/bbs/board.php?bo_table=security&wr_id=104

tls 정책과 관련해서 default_server에 대해서 얘기하는거 같은데 ?
https://varins.com/library/server/web-server-nginx/

a ssl기한도 있다고 한다.

인증서 수정이 안돼

보니까 https연결이 된다는것은 그냥 엔진엑스까지만 도달해도 된다는거네
왜냐면, https://celebmine.com해도 정상적으로 https가 적용되니까,

그리고 listen으로 포트먼저 듣고 그 다음 server_name으로 매칭한다는데
https://jgj1018.github.io/server/2017/02/11/server1.html
일치하는 listen 포트가없으면 server_name에 맞게 매치하는것같다.

그럼 https://celebmine.com은 어떻게 매치하는걸까
이거는 맞는 listen도 없고 맞는 server_name도 없으면 그냥
default_server로..? 이건 말이 안되는데..?

<br>

https://danidani-de.tistory.com/39
아니 https리디렉션은 로드벨런서에서 해주는데 ?

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153543841-c6377ecb-88be-4037-9fb8-b38a80b8a8fa.png">
</p>

ㅁ 그, 그거 보여주자 DNs가 어떻게 작동하는지 그거보면 우선 도메인에 대한 IP를 받고
그 다음에 http건 Https건 보내주게 되는거다. 즉, routte53은 ssl이랑 상관없고
그저, 로드벨런서에 SSL인증서를 등록하여 사용하기 때문에 ROUTE53에서 라우팅 값을
로드벨런서로 해줘야지 ssl이 정상작동된다는거였다.

즉, 왜 빈스톡이아닌 route53에서 로드벨런서로 지정해줘야 ssl사용이 가능한지 그거 풀기

이게 A레코드로 celebmine.com만 하면, www.celebmine.com은 아무곳도
가리키지않게된다. 근데, default_server 80이 되어있으니 그래도 이거 가리키는거
아님 ? 맞다.(그때 default_server없애니 안들어가졌다.)

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="">
</p>

AWS에서 제공해주는 SSL 인증서는 무료인 대신에, 해당 도메인의 DNS 서버로 Route53을 사용해야한다.

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로, 알아두면 좋을 HTTPS 기본 개념

<p align="center">
<img src="">
</p>

a

<br>



태그 : #