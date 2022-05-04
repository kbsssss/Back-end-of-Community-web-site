<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166184112-67d8e456-bad2-46c0-8df5-01528b404275.png">
</p>

# 📖 Beanstalk기반 환경에 도메인 SSL 설정하기

* HTTPS 작동원리와 SSL/TLS 인증서의 개념
* ACM에서 SSL 인증서 발급받기
* Beanstalk 환경에 ACM 인증서 등록하기

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 1.HTTPS 작동원리와 SSL/TLS 인증서의 개념

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166401664-c65fc2ba-6ce0-4490-a021-3134a550e923.png">
</p>

HTTPS란 무엇이고 SSL 인증서란 무엇일까 ?     
이 글을 읽고나면 빈스톡기반 서비스의 도메인에 https가 적용되는것을 볼 수 있을것이다.
다만, 기본적인 원리와 개념들은 알고있어야 후에 이를 응용하거나 아니면 리다이렉트(http -> https)에 대한
개념을 볼 때 더 잘 이해할 수 있으니 꼭 읽어보고 가도록 하자.

* HTTPS : 말 그대로 기존 HTTP에서 보안이 더 강화된 버전이다. 조금 더 쉽고 자세하게 얘기하자면, 인증된 서버에 한해서
  데이터를 전송할 때 암호화를 적용하여 진행한다. 아래 예시를 보면 완전히 이해할 수 있다.

* SSL/TLS 인증서 : 클라이언트와 서버 간의 통신의 안정을 보증해주는 문서이다. 조금 더 쉽게 설명하자면 클라이언트(브라우저)에서
  서버에 데이터를 보내려하는데 해당 서버가 신뢰할 수 있는 서버인지 확인하려 할 때 이 SSL/TLS 인증서를 확인하게 된다. 이 또한 아래
  구체적인 예시를 보면서 다시 한번 설명하겠다.

> 우리가 HTTPS 연결을 위해 가장 많이 들어본 인증서는 SSL(Secure Socket Layer) 인증서이다. 그렇다면, TLS(Transport Layer Security) 인증서는 
> 무엇일까? TLS는 SSL의 업데이트된 버전이며 명칭만 바뀐것이다. 현재 우리가 사용하는 인증서는 이 둘을 함께 사용하고 있다. 우리가 앞으로 받을 인증서도 SSL/TLS 인증서이다.
> 하지만, SSL/TLS 인증서나 TLS인증서를 모두 그냥 SSL 인증서라고 혼용해서 부르기도 한다.(하지만, SSl과 TLS는 기능적인 면에서 차이점이 존재한다. 자세한 내용을 알고싶다면 
> SSL과 TLS의 차이점에 대해 찾아보길바란다.)    
> [SSL과 TLS에 관하여 (1)](https://kanoos-stu.tistory.com/46)     
> [SSL과 TLS에 관하여 (2)](https://ssungkang.tistory.com/entry/Web-SSL-%EC%9D%B8%EC%A6%9D%EC%84%9C%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%ED%95%B4%EC%82%AC%EC%A0%84-%EC%A7%80%EC%8B%9D-%EC%A0%95%EC%9D%98-%EB%8F%99%EC%9E%91%EC%9B%90%EB%A6%AC-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EB%B9%84%EA%B5%90)    

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166401675-33281ada-1054-432a-a193-176003d196b2.png">
</p>

브라우저, 인증기관 그리고 서버와의 관계에 대해 알아보기전에 다른 예시를 보고 가겠다.

성인이되면 동사무소에서 주민등록증을 발급한다. 술집에서는 발급된 주민등록증을 확인하고 술을 주는데
이 주민등록증이 과연 진짜 주민등록증인지 확인을 해야한다. 술집은 주민등록증을 발급해주는 동사무소에 연락을
하여(실제로는 해당 주민등록증의 지문을 단말기로 대조 한다고 한다.) 진짜 주민등록증인지 확인을 하고 손님에게
최종적으로 술을 건내주게 된다.

HTTPS보안과 SSL/TLS인증서 서버와 브라우저사이의 관계도 이와 같다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166401683-9abcb994-bc80-4dd7-9281-48046b159e6f.png">
</p>

서버는 자신의 서버가 신뢰할만한 사이트인지를 알려주기위해 SSL/TLS 인증서를 발급받는다.
해당 SSL/TLS 인증서는 인증기관(CA)에서 발급받아 사용하게된다. 그리고나서 브라우저(클라이언트)가 해당 사이트의 서버에
접속하려 할 때 이 서버가 안전하고 신뢰할만한지 알기 위해서는 해당 서버가 발급받은 SSL/TLS인증서를 확인하고 또한 이 인증서를
발급해주는 인증기관(CA)에 이 인증서가 진짜인지 확인 과정을 거치게 된다. 최종적으로 신뢰할만한 인증서라는 확인을 거치게 되면
브라우저는 이제 서버에 보낼 데이터들에 대해 어떻게 암호화를 할것인지 서버와 함께 협상을 진행 하게 된다.

정리하자면, 클라이언트(브라우저)가 HTTPS를 사용하게 되면 신뢰할만한 서버에만 데이터를 보내게 되고, 데이터를 보낼때도
암호화를 설정하여 데이터를 보내게 된다. SSL/TLS 인증서는 서버가 자신이 신뢰할만하고 안전한 서버인지를 보여주는것이다.

기본적인 개념과 원리에 대해 알았으니 이제는 실제로 빈스톡 환경 기반의 도메인에 SSL/TLS 인증서를 발급받고
HTTPS가 정상적으로 작동하도록 설정해 보겠다.

> [HTTPS와 SSL 그리고 브라우저(클라이언트)와 서버의 작동원리](https://aws-hyoh.tistory.com/entry/HTTPS-%ED%86%B5%EC%8B%A0%EA%B3%BC%EC%A0%95-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0%EC%9A%B0%EB%A6%AC%EB%8A%94-%EA%B5%AC%EA%B8%80%EC%97%90-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%93%A4%EC%96%B4%EA%B0%80%EB%8A%94%EA%B0%80)

<br>



### 2.ACM에서 SSL/TLS 인증서 발급받기

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397052-138ae7c6-df2a-4216-8ce9-2ccbe0f6c833.png">
</p>

AWS의 ACM(AWS Certificate Manager)을 접속해보면 위와같은 화면이 나온다. 
이곳에서 우리는 SSL/TLS 인증서를 요청하고 발급받

> Amazon ACM에서 발급해주는 SSL/TLS 인증서는 무료인 대신에, 해당 도메인의 DNS 서버로 Route53을 사용해야 한다. 그러니 route53을
> 이용하여 미리 도메인을 연결해놓자. 아직 연결해놓지 않았다면 필자가 적어놓은 [Beanstalk기반 환경에 Route53을 이용하여 도메인 연결](https://sooolog.dev/Beanstalk%EA%B8%B0%EB%B0%98-%ED%99%98%EA%B2%BD%EC%97%90-Route53%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0/)
> 을 참고하도록 하자.

> [](https://wwlee94.github.io/category/blog/aws-eb-https/)    

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166397059-6fbb60b0-7218-46ba-98ca-1e875424a81e.png">
</p>

그것도 해야해 보니까 tls인가 인증서 버전을 1.3이상 올려야 한다. 등 이런얘기 하고 있어
이거 정책아님 ?
https://www.google.com/search?q=ssl+tls+%ED%95%A8%EA%BB%98+%EC%82%AC%EC%9A%A9&rlz=1C5CHFA_enKR982KR982&oq=ssl+tls+%ED%95%A8%EA%BB%98+%EC%82%AC%EC%9A%A9&aqs=chrome..69i57j33i160l2.8432j0j1&sourceid=chrome&ie=UTF-8


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