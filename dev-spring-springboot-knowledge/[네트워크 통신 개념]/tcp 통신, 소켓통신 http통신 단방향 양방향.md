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



정리하자면,,

1.TCP통신은 4 layer, http통신은 7 layer이다.
2.TCP 기반 구성을 할 때 소켓을 이용하여 실시간 통신이되며, 양방향 통신이다.
https://intrepidgeeks.com/tutorial/clean-the-socket-and-web-socket-at-one-time-2-the-difference-between-socket-and-web-socket-everything-about-web-socket-and-the-relationship-between-http-tcp-socket
3.근데 http는 애초에 tcp 기반으로 만들어진것이기에 소켓을 이용하여 통신을 하지만 양방향이 아닌 단방향이라고 한다.
    그 이유는, 애초에 TCP 통신은 4계층에서 이루어지고 http통신은 7계층에서 이루어 지기에 계층에서 차이가 나며,
    아무리 http 프로토콜이 tcp 기반의 프로토콜이라 해도 서로 다른 프로토콜에서 통신이 이루어지는것이기 때문에, 
    소켓이 http 통신에 사용된다하더라도 단방향으로 사용이 되는거다. 
4.그렇기에, http통신에서도 양방향 소켓이 사용되기 위해 웹소켓이 나오게 된거다.
    그전에는 단방향을 polling으로 해결하고 있었
5.웹 소켓 통신은, WS, WSS 프로토콜 기반이며 tcp포트 80,443에서  동작하도록
    설계되었다. 그렇기에 HTTP 의 프록시를 지원할 수 있게 하기 위하여 HTTP 포트 80, 443에서 동작하도록 설계되었다. 이 때문에 HTTP 프로토콜과의 호환도 가능하다. 웹소켓을 사용하기 위해서는 HTTP Upgrade header라는 걸 사용하여 HTTP 프로토콜에서 WebSocket 프로토콜로 전환된다. 이 과정을 WebSocket HandShake라고 함.
    WebSocket 프로토콜을 이용하면 이전에 서버와 클라이언트 간의 실시간, 양방향 통신을 가능하기 위해서 HTTP Polling 방식을 사용했던 것보다 overhead가 확실히 작아진다. 이는 클라이언트가 먼저 서버에게 요청하지 않고, 서버가 클라이언트에게 contents를 보내고 이 연결이 계속 열린채로 유지되기 때문에 가능하다.
   https://velog.io/@imacoolgirlyo/web-socket%EA%B3%BC-socket.io
6.웹소켓은 HTTP로 Handshake를 한 후 ws로 프로토콜을 변환하여 웹소켓 프레임을 통해 데이터를 전송합니다
  출처: https://kellis.tistory.com/65 [Flying Whale]





앞으로, 이 이미지 앞에 <br> 붙이기

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

> 처럼 띄우는거에는 항상 위아래로 <br> 붙여주기
