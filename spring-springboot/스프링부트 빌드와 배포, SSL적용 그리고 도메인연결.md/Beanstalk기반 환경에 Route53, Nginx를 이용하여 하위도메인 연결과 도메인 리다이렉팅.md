<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153751897-1f53ab1c-3e98-4c42-acb6-d6925fb77eef.png">
</p>

# 📖 Beanstalk기반 환경에 Route53, Nginx를 이용하여 하위도메인 연결과 도메인 리다이렉팅

* 하위도메인(www.도메인)연결과 리다이렉팅
* 도메인.net, 도메인.co.kr을 도메인.com으로 리다이렉팅
* 하위도메인(m.도메인)연결과 설계

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *

<br>



### 1.하위도메인(www.도메인)연결과 리다이렉팅

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786539-3a375a17-591d-419b-836f-3ef32edcdcd6.png">
</p>

기존에 만들어 두었던 real-test.com 호스팅영역을 클릭해보았다. 기존의 도메인(real-test.com)이
A레코드로 로드벨런서에 라우팅값이 지정된 것을 볼 수 있다. 하지만, 해당 레코드값만 지정해서는 www.real-test.com에
대해서는 따로 라우팅해주는 값이 없기에 브라우저에 www.real-test.com 입력시 아무 화면도 띄어주지않는다.

<br>

> 호스팅 영역이란, route53 aws 리소스에서 생성한 것이다.    
> [빈스톡 환경에서 route53을 이용하여 도메인 연결하기](https://github.com/sooolog/dev-spring-springboot-knowledge/blob/master/dev-spring-springboot-knowledge/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%20%EB%B9%8C%EB%93%9C%EC%99%80%20%EB%B0%B0%ED%8F%AC%2C%20SSL%EC%A0%81%EC%9A%A9%20%EA%B7%B8%EB%A6%AC%EA%B3%A0%20%EB%8F%84%EB%A9%94%EC%9D%B8%EC%97%B0%EA%B2%B0.md/Beanstalk%EA%B8%B0%EB%B0%98%20%ED%99%98%EA%B2%BD%EC%97%90%20Route53%EC%9D%84%20%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC%20%EB%8F%84%EB%A9%94%EC%9D%B8%20%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0.md)의 연장글이니
> 아직 도메인연결을 진행하지 않은분들은 참고하고 오면 된다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786538-75e0527a-1973-4cd3-a98b-a63ad8aea921.png">
</p>

그래서 우리는 레코드 이름에 www를 입력하고 A레코드를 선택하며, 별칭으로 이전에 만들어 두었던 빈스톡의
로드벨런서를 선택해주면 된다.

<br>

> 왜 빈스톡이 아닌 로드벨런서를 선택해야하는지도 바로 직전에 말한 이전글에 설명되어있으니 참고하도록 하자.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786521-6cb69823-e61f-40e0-9292-8ca7cb237989.png">
</p>

www 하위도메인에대한 레코드 지정까지 끝내면 real-test.com 호스팅 영역에 위와같이 나타나게 된다.
여기까지 하면 route53에 대한 설정은 끝이다.

<br>

```conf

```

여기 엑세스로그도 전부 따로 지정해줘야
이거 따로 지정하는 방식 vompressor.com/nginx-log/를
참고하자.

보니까, 엔진엑스에서 404에러같은거 에러페이지 보여주게 하는 방식 지정할 수 있다.
기본 프로젝트는 안되도 그냥 다른 도메인 연결한거에 대해서는 지정할 수 있을듯
dev-jwblog.tistory.com/42

그리고 default_server도 전부 다시 지정해야해 보안강화게
https://kamang-it.tistory.com/entry/WebServernginx%EB%A1%9C%EA%B7%B8%EB%93%A4-%EB%B3%B4%EA%B8%B0

아니 잠깐만, Www.도메인.com을 그냥 cname으로 도메인.com으로 돌리는 경우도 있는데 ?
https://jsikim1.tistory.com/153

<br>

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.도메인.net, 도메인.co.kr을 도메인.com으로 리다이렉팅

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786550-53963f11-6d02-4b65-a0b8-3ea3b02856c3.png">
</p>

a

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786548-c095ce49-2a53-4ac3-8359-6a87425bf2fb.png">
</p>

aa

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153787368-508968a6-4752-46b6-aad3-5a73f895a043.png">
</p>

a

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786545-a8dcf279-c191-4e4d-a43f-2f65dc304686.png">
</p>

ab

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786544-a124b056-57b3-4fb8-9cea-7739c145d90e.png">
</p>

abc

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786543-435a0910-dca2-4fbc-ad32-4b196b7c124a.png">
</p>

a

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786551-bd3756ca-158a-43ec-a848-423d2575df0f.png">
</p>

a

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/153786549-98397fa6-6081-4bc6-8af2-afdc2dd83861.png">
</p>

a

<br>

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 3.하위도메인(m.도메인)연결과 설계

<p align="center">
<img src="">
</p>

ㅁ

<br>

<p align="center">
<img src="">
</p>

a

<br>

<p align="center">
<img src="">
</p>

a

<br>

<p align="center">
<img src="">
</p>

a

<br>

<p align="center">
<img src="">
</p>

a

<br>

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #

> 처럼 띄우는거에는 항상 위아래로 <br> 붙여주기


2.혹시 첫번째 DNS 쿼리에 대해서 성능이 좋다는게, 어차피 로드벨런서나 빈스톡은 매번 TTl없이 진행해야하는데, 서브도메인 호스팅영역으로
 그냥 NS만하면 그 값은 안바뀌잖아 그러니까 그거는 캐시 즉,ttl을 적용해서 네임서버 부하를 덜어주는거지.
 
 엔진엑스는 반영이안되나, 그 배포는 됬는데 건강체크해서 실패한 경우, SSh 접속해서 보면 좋을텐
 이거도 전부 반영이됬다.
 
 그거도 해야해, 내가 배포실패해도 파일은 올라갔는지, NGinx.conf파일 볼려고 SSh 접속하려하는데 안됨
 근데, 이거 빈스톡 버전 최신꺼로하니 됬다.; 시
 
 그리고 이상하게 내 CELEBMINE.COM해서 들어가도 다시 클릭해보면 Www안보인다.
 그러나, WWW.Celebmine.com으로 들어가면 보인다. 흠..다른데는 무조건 WWw가 보이는데
 Https의 영향인건가.