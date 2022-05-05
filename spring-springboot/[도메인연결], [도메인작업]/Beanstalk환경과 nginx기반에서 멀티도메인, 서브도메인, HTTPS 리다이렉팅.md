<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166934711-797918c8-e3e8-4bc9-a44d-e717cd2a073c.png">
</p>

# 📖 Beanstalk환경과 nginx기반에서 멀티도메인, 서브도메인, HTTPS 리다이렉팅

* 
* 

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *

<br>

### 1.

<p align="center">
<img src="">
</p>

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

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="">
</p>

ㅁ

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #

