<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151650030-15d1cafa-0f30-4803-8ea7-ecbc60d5a596.png">
</p>

# 📖 [스프링부트] Github Action과 Beanstalk으로 CI/CD 하기 A부터 Z까지 (3)

* 빈스톡 환경 생성
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

<br>

> 다음은, 추가 옵션 구성을 눌러준다. 환경 생성을 눌러주면 안된다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652656-6388ba27-7a14-4ed5-8c2e-85aab42353c9.png">
</p>

추가 옵션 구성을 클릭하면 맨위에 이런 화면이 뜬다.
단일 인스턴스를 사용하게 되면, 로드벨런서를 사용할 수 없게된다. 그러니 사용자 지정 구성을 클릭해주자.
그 외에 고가용성에서 가용성이란 의미는 사용가능 시간을 의미한다. 즉, 간단하게 말하면 서비스가 끊이지 않고 얼마나
연속해서 이어지는 서비스를 제공하냐를 의미한다. 현재 단계에서는 보지않아도 되니, 사용자 지정 구성을 클릭해 준다.

단, 스팟 인스턴스, 온디맨드 인스턴스, 예약 인스턴스는 꼭 알아야 할 기본 개념이니 한번 살펴 보고가도록 하겠다.
(이미 개념을 안다면 그 다음 사진으로 넘어가도 된다.)

**스팟 인스턴스** : 스팟 인스턴스를 이해하려면, 스페어 EC2용량에 대해 먼저 알아야 한다. 남는 EC2에 대해, 수요와 공급에 의해 스페어 EC2의 가격이 정해지는데,
내가 인스턴스 입찰 가격의 범위를 지정해놓고, 해당하는 범위 내에 스페어 인스턴스 가격이 내려오면 입찰이 되어 이용가능해져 인스턴스가 시작된다. 다시 가격이 올라가면
이용을 할 수 없게 되어 인스턴스가 종료되는 방식이 바로 스팟 인스턴스이다. 그러다 보니 언제 종료될지 알 수 없게되는 단점이있다. 그 외에도 온디맨드 방식에 비해서는 90%까지 저렴한 가격을 제공하기에 장단점이
각각 존재한다. 주로 Batch작업에 많이 사용하며, 이 스팟 인스턴스에도 종류가 4가지나 있으니, 더 자세히 알고싶다면 아래 참조링크를 참고하도록 하자.

<br>

> batch 작업이란, 배치작업은 데이터를 실시간으로 처리하는게 아니라, 일괄적으로 모아서 한번에
> 처리하는 작업을 의미한다. 가령, 은행의 정산작업의 경우 배치 작업을 통해 일괄처리를 수행하게 되며 
> 사용자에게 빠른 응답이 필요하지 않은 서비스에 적용한다.

<br>

> [스팟 인스턴스의 개념과 4가지 종류](https://blog.leedoing.com/178)
> [스팟 인스턴스의 개념](https://wooono.tistory.com/86)
> [Batch란 무엇인가 ?](https://gold-jae.tistory.com/46)

<br>

**온디맨드 인스턴스** : 이용한 만큼만 지불하는 방식으로, 1시간단위이며 단1분만 써도 1시간 단위로 계산된다.
선결제가 불가능하고, 다른 인스턴스들에 비해 비싸지만 제일 대중적으로 사용되는 인스턴스이다. 사용하지 않을때는 비용이
지불되지 않는다.

<br>

> Lamda 혹은 aws instance scheduler를 통하여 서버의 시작 및 종료지정하여 온디맨드 방식의 인스턴스의 사용량을 줄여 비용을 절감하기도 한다.

<br>

> [온디맨드 인스턴스란 (1)](https://ssue95.tistory.com/4)
> [온디맨드 인스턴스란 (2)](https://codingmoondoll.tistory.com/entry/EC2-%EB%B9%84%EC%9A%A9-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

<br>

**예약 인스턴스** : 일반적인 AWS EC2의 요금은 한 달 기준으로 인스턴스를 사용한 시간에 따라 요금이 책정되는 방식인데(온디맨드 인스턴스), 
예약 인스턴스를 이용하면 최소 1년 이상 인스턴스를 사용해야 하는 대신 시간당 요금을 할인받을 수 있다.
한 번에 많은 돈을 지불해야 하지만 전체적으로 봤을 때 훨씬 싸게 이용 가능한 헬스장 회원권과 비슷한 개념이다.
별다른 특이사항 없이 장기간 EC2 인스턴스를 사용해야 한다면 예약 인스턴스를 이용하는 게 비용적인 측면에서 훨씬 이득이다.

<br>

> 이렇게 보면, 당연히 온디맨드 방식이 아닌 예약 인스턴스를 사용해야할것같지만, 최소 계약 단위가 1년이고 Scale up을 할 수 없기에
> 사용하지 않게되더라도 환불할 수 없으니 잘 생각해보고 이용해야 한다.

<br>

> [예약 인스턴스의 개념 (1)](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=rnjsrldnd123&logNo=221492606516)
> [예약 인스턴스의 개념 (2)](https://ssue95.tistory.com/4)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652654-930b5456-1cad-4618-8a91-d8b1299c3cbe.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652858-5c340932-670f-4101-bb06-0fbaf32c2db0.png">
</p>

인스턴스의 편집을 클릭하면 위와같이 인스턴스 보안 그룹을 선택하는 란이 보인다.
필자는 미리 만들어둔 보안그룹이 있기에, 해당 보안 그룹 test-service-ec2을 선택했다.

<br>

> 인스턴스의 보안그룹 생성에 관한것은 필자가 적어둔 [파일업로드 적용하기 A부터 Z까지 (1)](https://github.com/sooolog/dev-spring-springboot-knowledge/blob/master/dev-spring-springboot-knowledge/SpringBoot%20file%20upload%20with%20S3/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%20%ED%8C%8C%EC%9D%BC%EC%97%85%EB%A1%9C%EB%93%9C%20%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0%20A%EB%B6%80%ED%84%B0%20Z%EA%B9%8C%EC%A7%80%20(1)%20.md)글에
> 적어 놓았다. 또한, 보안 그룹만 따로 생성할 수도 있으니 참고하도록 하자.

<br>

여기서, 추가로 짚고 넘어가야 할 부분이 있다. 뒤에 나올 nginx에서 자세하게 다룰 내용이지만 미리 알고가면 더
이해가 잘 되기에 nginx와 로드벨런서, ec2 그리고 보안그룹의 간단한 구조에 대해 알아보고 왜 포트80을 열어주어야 하는지에 대해서도 알고 넘어가도록 하겠다. 
아래 사진들을 보자.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101959-50f08e62-e2dd-4e77-987f-a70c0b25092f.png">
</p>

Internet Gateway에서 로드벨런서가 먼저 요청을 받고 이를 Nginx에 80포트로 요청을
전달 하고, 그 뒤에 어플리케이션으로 요청을 전달하게 된다. 또한, Nginx는 EC2내부에 있다.

<br>

> 여기에 포트 5000이라 적혀져있는것은 뒤에 설명할것이니 지금은 신경쓰지 않도록 하자.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101955-9efdfb27-fb66-4d3c-a4f6-4dfe8c9d0bd5.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101954-625e2162-0f72-4e82-8f57-cd1a8c1c4f3d.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101953-d1a59d88-e1da-4604-b0cd-28acdc3da974.png">
</p>

세 개의 그림 모두, 처음 말했던 구조를 갖고 있다. 이 그림을 보여준 이유는 Nginx는 EC2 내부에 있고, EC2 내부에 있다면,
로드벨런서로부터 80포트로 요청을 받을때 ec2 보안그룹을 통과하여야 한다. 즉, EC2의 보안그룹에도(빈스톡의 EC2 보안그룹) 80포트에 대해 
허용을 해주어야지 Nginx가 요청을 받을 수 있는것이다.

<br>

> 나머지 5000포트와 8080포트 그 외에 관한 내용은 뒤에서 자세히 다룰 것이다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152107605-84ecfb27-031e-44dc-a171-44c5c95f4e9f.png">
</p>

그렇다면, 빈스톡 환경생성을 할 때 보안그룹을 어떻게 구성해야 하는지 보도록 하겠다. 위 사진은 위에서 선택한 보안 그룹 test-service-ec2이다. 
ssh에 대해서는 보안상 내ip만 접속하도록 해놓았고, 443과 8080포트에 대해서는 전부 열어놓았다. 그렇다면, 의문이 든다. 80번 포트에 대해서도 보안그룹 인바운드 규칙을 추가해주어야 웹 브라우저에서
접근할 수 있을텐데 이게 어떻게 가능할까 ? 아래 그림을 보도록 하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101949-d3d9fcdb-9372-4ec3-a89a-25f9e6b17715.png">
</p>

빈스톡 환경을 생성해주면 위와같은 디폴트 보안그룹이 2개가 자동으로 생성이 된다. 각각 어떠한
인바운드 규칙이 있는지 보도록 하겠다.

<br>

> 디폴트 보안그룹 2개는 빈스톡 환경 생성을 할때, 보안그룹을 선택(기존에 내가 만들어 놓은 보안그룹) 하던 선택하지 않던
> 디폴트로 생기는 보안그룹이다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101950-e91a36aa-49ec-4012-9ead-8e999e1728d2.png">
</p>

첫번째 디폴트 보안그룹은 이렇게 80포트로 IPv4에 대해서 전부 열려있다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152101957-c453f35c-f580-4306-add8-2c41ff197338.png">
</p>

다른 두번째 디폴트 보안그룹은, ssh에 대하여 포트가 전부 열려있고, 방금 전 보았던 첫번째 보안그룹의 보안그룹ID에 대해 80포트로 받아들이고 있다.
필자의 [파일업로드 적용하기 A부터 Z까지 (1)](https://github.com/sooolog/dev-spring-springboot-knowledge/blob/master/dev-spring-springboot-knowledge/SpringBoot%20file%20upload%20with%20S3/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%20%ED%8C%8C%EC%9D%BC%EC%97%85%EB%A1%9C%EB%93%9C%20%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0%20A%EB%B6%80%ED%84%B0%20Z%EA%B9%8C%EC%A7%80%20(1)%20.md)
의 글을 참고해보면, 보안그룹을 추가할때 다른 보안그룹ID로 추가하는 경우, 추가한 보안그룹ID에 해당하는 보안그룹이 적용된 서버로 들어온 요청이 다시 보안그룹ID를 인바운드에 추가한 서버로의 요청에 대해
허용 한다는 말이다. 아래 사진을 보면서 다시 정리하겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152114275-145df6f5-f986-45a9-909d-c07901253b70.png">
</p>

이 사진은 빈스톡 환경 구성을 끝내고 환경을 생성한 후 다시 인스턴스 카테고리를 편집하여 인스턴스 보안 그룹 설정을 보여주는것이다.
필자가 위에서 생성한 보안그룹만 체크하고 넘어갔는데, 자동으로 디폴트 보안그룹중 하나가 체크된 모습이다. 이 체크된 디폴트 보안그룹은 두번째 디폴트 보안그룹이다.
그렇기에, 기존 필자가 만든 보안그룹에서 80번 포트 HTTP 설정을 해주지 않아도, 정상적으로 작동이 가능하게 되는것이다.

<br>

> 실제로, 두번째 디폴트 보안그룹의 80번포트를 허용하는 첫번째 보안그룹ID를 지우니 정상적으로 배포되지않고, 배포 후에
> 보안그룹ID를 두번째 디폴트 보안그룹 목록에서 삭제하면, 브라우저에서 정상적으로 열리지 않았었다.
> 추가로, 굳이 디폴트로 등록된 보안그룹ID로 80번 포트에 대해서 요청을 허용하도록 적지않고, HTTP port 80, 0.0.0.0/0로만 보안그룹 목록을
> 추가하면, 이 또한 정상적으로 작동한다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152131390-4dde48c0-2709-487b-9fb7-5f3aaa5751ca.png">
</p>

그 다음, 수정해 주어야 할 부분이있는데 우리가 ec2 단일 인스턴스를 만들 때 ssh 접근해 대해서도 보안그룹으로 설정을 하여 사용한다.
ssh접근에 대해서는 aws에서 pem키를 발급하여 pem키가 있어야만 인스턴스에 ssh접속을 할 수 있는데, 그 외에도 보안그룹에서 22번 포트에 대해
접근 가능 ip를 제한해 둔다. 기존 단일 EC2인스턴스를 생성할 때도 디폴트 보안그룹에 ssh가 전체열림(0.0.0.0/0)으로 되어있기에, 이를 내IP만
접속할 수 있도록 바꿔놓는다. 빈스톡에서도 디폴트 보안그룹에서는 전체열림으로 되어있으니 우리는 이를 보안강화를 위해 내ip로만 접속할 수 있도록 
바꾸어 줄 필요가 있다. 필자가 위의 작성하여 보안그룹에는 이미 ssh에 대하여 내 ip로만 접속이 가능하다고 적어놓았으니, 두번째 디폴트 보안그룹에 
적혀있는 인바운드 규칙의 ssh에 대해서는 보안상 삭제하는게 좋다.

<br>

> 위 사진은 두번째 디폴트 보안그룹으로 ssh 목록을 삭제한 뒤의 자료이다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/152134332-76a73032-38d3-4403-bb6f-7da8ec8c733c.png">
</p>

마지막으로, 첫번째 디폴트 보안그룹에 80번 포트로의 접근에서 ::/0도 설정해주어, 혹시 모를 IPv6에 대한
접근도 가능하게 해주자.

<br>

> (EC2에 Nginx 설치)[https://dev-jwblog.tistory.com/42]      
> (EC2 내부에 Nginx (1))[https://jojoldu.tistory.com/549]      
> (EC2 내부에 Nginx (2))[https://browndwarf.tistory.com/66]       
> (EC2 내부에 Nginx (3))[https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/concepts-webserver.html]     
> (EC2 내부에 Nginx (4))[https://tipsfordev.com/502-bad-gateway-elastic-beanstalk-spring-boot]     

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151653188-2dca1ecb-e073-4116-a3a1-7267c6980143.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/151652651-f4faaf49-e88d-4579-8618-4adddc127e70.png">
</p>

용량에서 편집을 눌러준다.   
그러면, Auto Scaling 그룹이 보이는데, 필자는 최대 인스턴스 수를 1로 적어주었다.
이것의 의미는, 아무리 트래픽이 많이 생기고 CPU 가동률이 올라가도 최대 1개의 인스턴스 외에는
Auto Scaling을 적용하지 않겠다는 의미이다.

> 당연히, 실제 배포되는 서비스나 규모가 어느정도 생긴다면 인스턴스의 최소 최대를 유연하게 설정해주어야 하는게
> 맞으나 우리는 프리티어로 무료로 사용해야하니 최대 1개로 진행하는 것이다.

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

태그 : #온디맨드 인스턴스, #예약 인스턴스, #스팟 인스턴스, #Nginx 80포트, #ssh 보안그룹, #




1.빈스톡 connect fail해서 한 경우도 인스턴스가 바뀌지않음
2.이상하게 그 디그레이드 되서 기다리게 된거 있잖아 배포는 됬는데, 그거는 EC2가 바뀌었는데 ? 직접해봄 - 그니까, 실제 인스턴스도 바뀌고 배포도 완료되어 들어가보면 변경된 restcontroller의 String값이 브라우정 나왔다.
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
13. 온디맨드 인스턴스는 람다나 scheduler를 이용하여 비용을 절약할 수도 있다고 한다.
    근데, 이게 로드벨런서랑 무슨 차이일까 흠 ?
    https://ssue95.tistory.com/4

14.그 db연결할 때 우선 build.gradle에 의존모듈     implementation('org.mybatis.spring.boot:mybatis-spring-boot-starter:2.1.3')
이 있으면 빈스톡에 배포가 안된다. 그러고 보니, 일반적으로 인텔리제이에서 실행할때도 안됬던게 있었는데 이거 의존모듈 때문이였나 ?
그거 비교해서 다시 봐보고 정리하자.

15.탄력적ip하려면 로드벨런서인가 거기에서 VPC해야 하는거 간단히 정리

19.혹시 ssl적용하기 위해 리스너 추가하면 어떻게 달라지는지 보자.

21.꼭 ssl할때 리스너에 따라서 이런거 어떻게 바뀌는지 꼭 정리하고 보자.