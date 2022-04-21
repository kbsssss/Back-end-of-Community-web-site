<p align="center">
<img src="">
</p>

# 📖 Nginx의 keepalive와 ALB의 유휴 연결 시간, client_header,body_timeout과의 관계

* Nginx의 keepalive와 ALB(Application Load Balancer)의 유휴 연결 시간과의 관계
* Nginx의 keepalive와 client_header_timeout, client_body_timeout과의 관계

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 1.

<p align="center">
<img src="">
</p>

1.alb연결 유휴 시간내에 데이터가 계속전송되면 유휴 시간을 늘린다.
2.그런데, keepalive가 어디까지 적용되는거지 ? 이거 ec2와 로드벨런서 그리고 ec2내의것들까지 커버되는거같다.
3.연결 유휴시간이 끝나면 로드벨런서는 프론트와의 연결을 끊는다는데
4.아... 그 nginx의 keepalive는 로드벨런서와 벡엔드사이 연결을 의미하는거같은데..?
5.이해했는데, 그러면 keepalive끝나서 만약 로드벨런서와 서버가 연결을 끊는다고하면, 먼저 끊는게
  서버이니까 time_wait말하려 했는데 로컬포트를 사용하는게 아니니까 상관없구나.
https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/application/application-load-balancers.html#connection-idle-timeout
6.그러면 alb의 연결 유휴 시간이 60초이면 keep alive는 60초보다 길게 해야하나.
7.빈스톡은 연결 유휴 시간 정하는게 따로없던데 그라면..? 못바꾸나?
8.이 유휴 연결 시간은 전송할 데이터가 더 있거나 하면 자동으로 늘어난다고 한다.

그래서 값이 같아도 되는지 아니면 keepalive가 더 커야되는지 모르겠네 우선 브라우저별로 다르지만 요새 브라우저는 최대
keepalive값이 60초라던데

<br>



### 2.

<p align="center">
<img src="">
</p>

client_body_timeout이나 client_header_timeout의 경우, 음..
헤더는 해당안되는걸수도 있는데 연결이 클라이언트와 서버가 오고간다음에 그 사이에도
뭔가를 전송할게 남았는데 그게 제대로 전송이 안되면 에러가 발생하기에 keepalive시간을
이 body나 header timeout보다 길게 잡아야 한다는거같다.
https://intrepidgeeks.com/tutorial/set-nginx-timeout

또한, keepalive는 애초에 클라이언트와 서버가 주고받고나서부터 시작하는 것이다.
연결된 Socket에 I/O Access가 마지막으로 종료된 시점부터

정의된 시간까지 Access가 없더라도 세션을 유지하는 구조이다.
https://goodgid.github.io/HTTP-Keep-Alive/
<br>

그래서 값이 같아도 되는지 아니면 keepalive가 더 커야되는지 모르겠네 우선 브라우저별로 다르지만 요새 브라우저는 최대
keepalive값이 60초라던데

### 🚀 추가로

<br>



태그 : #