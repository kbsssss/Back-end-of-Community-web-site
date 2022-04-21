<p align="center">
<img src="">
</p>

# 📖 

* 
* 

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 1.

<p align="center">
<img src="">
</p>

ㅁ

<br>



### 2.

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

<br>



### 🚀 추가로

<br>



태그 : #