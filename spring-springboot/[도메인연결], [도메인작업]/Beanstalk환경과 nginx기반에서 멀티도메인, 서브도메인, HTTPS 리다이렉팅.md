<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166934711-797918c8-e3e8-4bc9-a44d-e717cd2a073c.png">
</p>

# 📖 Beanstalk환경과 nginx기반에서 멀티도메인, 서브도메인, HTTPS 리다이렉팅

* 멀티도메인과 서브도메인 Route53 설정
* Nginx 설정파일 nginx.conf에서 멀티도메인, 서브도메인 리다이렉팅 설정

하위도메인(www.도메인)연결과 리다이렉팅
* 도메인.net, 도메인.co.kr을 도메인.com으로 리다이렉팅
* 하위도메인(m.도메인)설계와 리다이렉팅
* http:// -> https:// 리다이렉팅

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *


<br>

아 뭔데 뭔 NS(네임서버)야, 그러니까 왜 m.celebmine.com에 대해 CNAME이
아닌 NS로 설정하라고 AWS에서 얘기하는거지 ?

### 1.멀티도메인과 서브도메인 Route53 세팅

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/167347840-1d28e2b8-dc54-4221-b425-fda0a80bc873.png">
</p>

멀티도메인과 서브도메인의 리다이렉팅에 대해 알아보도록 하겠다.

흔히 우리는 도메인A.com을 주 도메인으로 사용하고싶은데 나머지 도메인A.co.kr이나 도메인A.net을 구매하여
이를 통한 클라이언트의 요청이 들어올시 다시 도메인A.com으로 접속되어 보여지게 하고싶어한다. 이를 리다이렉팅이라고
하는데, www.도메인A.com이나 m.도

> 흔히 우리는 도메인A.com으로 서비스를 내면, 도메인A.net이나 도메인A.co.kr도 한번에
> 구매를 하여 도메인A.com으로 리다이렉팅을 한다. 이때 여러 도메인을 하나의 서버로 연결해서
> 사용하는데, 이 때 사용된 모든 도메인을 멀티도메인이라고 한다. 또한 최상위도메인(com, co.kr, net)이 
> 다른것 외에 도메인B.com 같이 도메인명 자체가 다른 경우에도 다른 도메인들과 같은 서버를 가리키게 되면
> 도메인B.com도 멀티도메인이라 한다.     
> [멀티도메인이란](https://post.naver.com/viewer/postView.nhn?volumeNo=24092785&memberNo=11287836)

> m.도메인도 서브도메인 리다이렉팅에 속하긴하나, 이는 모바일 혹은 태블릿을 인지하고
> 디바이스별 리다이렉팅이 필요하기 때문에 
> 


<br>

<img src="https://user-images.githubusercontent.com/59492312/167347848-5c9459c3-3ca6-4586-906d-0b6184a2700d.png">
</p>

ㅁ

<br>

<img src="https://user-images.githubusercontent.com/59492312/167347850-7053e250-826b-49d7-985a-14f7f3e9e910.png">
</p>

ㅁ

<br>

<img src="https://user-images.githubusercontent.com/59492312/167347852-a80b09db-e48f-45ee-b491-bfa8c39ecdf4.png">
</p>

ㅁ

<br>

<img src="https://user-images.githubusercontent.com/59492312/167347854-0b027007-1e80-492c-be84-47b409fed769.png">
</p>

ㅁ

<br>





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

그 기능도 넣어야해, pc에서는 naver.com하면 당연히 PC버전 나오고 pc에서 m.naver.com하면 모바일버전도
들어가진다. 근데 모바일에서는 m.naver.com하면 당연히 들어가지고 naver.com해도 m.naver.com으로 들어가진다. 그러니
pc버전으로 보기를 따로 만들어야 한다. 모바일 버전에서
그러면 pc nginx에서 모바일이나 태블릿 체크해주고 이를 m.naver.com 리다이렉트 시켜주어야 한다는건데 ?

잠깐, 이 redirect도 301 status code인가, 그럼 302는 뭐고 bitly 이거 원리는 뭐지

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


여기 그것도 해야돼, DEFAULT_SERVER을 이용해서 맞는 URI가 아니면 반려하는거
https://xetown.com/tips/1172256

지금 이거 보니 listen 443을 안하고 그냥 지금있는 그대로 쓰면
불안정한 https연결이 될수도 있다는데 ?
http://linforum.kr/bbs/board.php?bo_table=security&wr_id=104





2.혹시 첫번째 DNS 쿼리에 대해서 성능이 좋다는게, 어차피 로드벨런서나 빈스톡은 매번 TTl없이 진행해야하는데, 서브도메인 호스팅영역으로
 그냥 NS만하면 그 값은 안바뀌잖아 그러니까 그거는 캐시 즉,ttl을 적용해서 네임서버 부하를 덜어주는거지.
 
 엔진엑스는 반영이안되나, 그 배포는 됬는데 건강체크해서 실패한 경우, SSh 접속해서 보면 좋을텐
 이거도 전부 반영이됬다.
 
 그거도 해야해, 내가 배포실패해도 파일은 올라갔는지, NGinx.conf파일 볼려고 SSh 접속하려하는데 안됨
 근데, 이거 빈스톡 버전 최신꺼로하니 됬다.; 시
 
 그리고 이상하게 내 CELEBMINE.COM해서 들어가도 다시 클릭해보면 Www안보인다.
 그러나, WWW.Celebmine.com으로 들어가면 보인다. 흠..다른데는 무조건 WWw가 보이는데
 Https의 영향인건가.
 
 그럼 https://celebmine.com은 어떻게 매치하는걸까
 이거는 맞는 listen도 없고 맞는 server_name도 없으면 그냥
 default_server로..? 이건 말이 안되는데..?
 
 그리고 listen으로 포트먼저 듣고 그 다음 server_name으로 매칭한다는데
 https://jgj1018.github.io/server/2017/02/11/server1.html
 일치하는 listen 포트가없으면 server_name에 맞게 매치하는것같다.
 
 그 nginx에서 server별로 access_log가 중복되는게 한개 있었는데 그거때문에
 배포 오류나는거였나 ? 14분씩 걸린다
 access_log    /var/log/nginx/access1.log main;
 이런거
 이게 맞았던거 같다. 바로 에러나도 단축확되서 나왔다.
 아 아닌가 이상한 에러다.
 
 다음, access log가 어디 저장될지 보이는건데, 이는 이 서버에 해당하는 거에 대해서만
 로그를 남긴다. 이것도 상속의 개념aaa + 이거 리다이렉팅에 넣자.
 
 /var/log/nginx/access.log이거 형식 있었는데 main;
 이거 서버마다 로그 이름 다르게 하는거 있었다.
 
 우선은
 리다이렉트하는거 자체가 nginx에서 하는거 그거 301이다., 302아님
 그리고 301은 영구한거로 검색 엔진에 점수들이 전부 바뀐거로 업데이트 되는데 302를 하면 요새는 막 광고 그런거도 많이되서 오히려 점수 깍인다고한다.
 https://nsinc.tistory.com/168
 그리고 request_uri는 그 파라미터인가 그거 적용시켜주는거고,
 실제로 이 리다이렉팅하면 엔진엑스에서 움직이는게 아니라 애초에 브라우저에서 url쳤을때 dns거치고 하는 과정을 그대로 거치게 된다. 네이버로 해봄
 그리고 예상한 바 다른 DNS에서 내 도메인을 지정하여 트래픽을 보낼수 있는데 그건 막을 수 없다고 한다. 그 윈도우10 리다이렉팅에 나와있다. 그래서
 대책을 세워야 할거같긴하다.
