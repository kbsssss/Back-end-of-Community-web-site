<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/159886777-d6c616ac-f01a-4343-8004-3a74017b4d10.png">
</p>

# 🖼 HTTP 통신과 TCP 통신의 차이점과 HTTP 프로그래밍과 소켓 프로그래밍에 대한 개념 정리

* HTTP 통신과 TCP 통신의 차이점, HTTP 프로그래밍과 소켓 프로그래밍

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 1.HTTP통신과 TCP통신의 차이점에 대해

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160339237-6c677536-0e9d-43fc-862d-cca439bd9cae.png">
</p>

우선 HTTP통신과 TCP통신에 대해 알기전에 OSI 7 Layer와 프로토콜이란것에 대해서 알아 가도록 하겠다.

OSI(Open Systems Interconnection) 7 Layer는 ISO(국제표준기구)에서 
만든 네트워크를 7계층으로 만든 모델이고, 프로토콜(Protocol, 통신규약)은 상호간의 
접속이나 전달방식, 통신방식, 주고받을 자료의 형식, 오류 검출 방식, 코드 변환방식, 
전송속도 등에 대하여 이미 정해진 약속이기 때문에 레이어별 프로토콜은 한마디로 
OSI 7 계층의 계층간에 존재하는 네트워크 통신을 위한 규약을 뜻한다.

즉, 우리가 이제 알려고 하는 HTTP(HyperText Transfer Protocol)와 TCP(Transmission Control Protocol)는
모두 특정 layer(계층)의 규약으로 해석할 수 있다.

<br>

> HTTP는 7계층 규약이고, TCP는 4계층 규약이다.    
> [HTTP와 TCP는 서로 다른 계층의 규약](https://itmining.tistory.com/127) 

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160337367-e64ee082-9cf4-4bb4-8dc1-8c2b9391dcba.png">
</p>

그렇기에, TCP통신과 HTTP통신이란 해당 계층에 해당되는 규약(protocol)아래에서 이루어지는
통신을 의미한다.

조금 더 각각의 통신에 대해 자세히 보도록 하겠다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160337367-e64ee082-9cf4-4bb4-8dc1-8c2b9391dcba.png">
</p>

TCP통신은 3-way-handshake라는 과정을 거치고 연결이 이루어진다.
그리고 연결을 종료할때는 4-way-handshake를 거치게 된다.

또한, TCP통신에서는 소켓을 이용한 연결방식을 사용한다.
그로인해, 양방향 통신이 가능하게 되는데 양방향 통신이란, 클라이언트단과
서버단이 서로 연결되어 있을때 실시간으로 양방향(클라이언트 -> 서버, 서버 -> 클라이언트)
으로 요청을 보내 통신을 할 수 있게 해주는 것이다. 또한, 클라이언트단과 서버단의
연결이 끊어지지않고 계속 연결을 유지해주어 실시간 소통이 가능하다.

위와같이 소켓을 이용한 연결은 TCP 프로토콜 기반으로 맺어진 연결이며,
**소켓통신**이라고도 부른다.

마지막으로, 이러한 소켓 통신을 사용하도록 설계하는 프로그래밍을 
**소켓 프로그래밍**이라고 한다.

<br>

> 3-way,4-way handshake에 대해서 잘 모르겠다면, 필자의 네트워크에 관한 [다른글](https://sooolog.dev/3-way,4-way-handshake%EC%99%80-Time_wait-%EC%86%8C%EC%BC%93%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%9C%EB%85%90/)을 참조하도록 하자.      
> [3-way,4-way handshake에 관하여](https://sooolog.dev/3-way,4-way-handshake%EC%99%80-Time_wait-%EC%86%8C%EC%BC%93%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%9C%EB%85%90/)

<br>

> [TCP통신과 3-way-handshake](https://velog.io/@vov3616/Socket%ED%86%B5%EC%8B%A0%EA%B3%BC-TCP-UDP)         
> [TCP통신과 4-way-handshake](https://bangu4.tistory.com/74)       
> [TCP통신과 양방향통신 그리고 소켓통신](https://mangkyu.tistory.com/48)    
> [TCP통신과 소켓 프로그래밍](https://mangkyu.tistory.com/48)    

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160383592-90d4ec99-94cf-4e41-921b-594fc483bc9c.png">
</p>

그렇다면, http통신은 어떻게 이루어지는것일까?    
http통신에 대해 정리하고 대개 고민하고 헷갈리는 부분에 대해 한번에 정리하도록 하겠다.

http통신은 tcp통신과는 다르게 단방향 통신이다. 즉, http통신은 클라이언트의
요청이 있을때만, 서버단이 응답하고 처리를 해주며 해당 응답이 끝나면 연결을 바로 끊게 된다.
(서버단에서 클라이언트단으로 요청을 먼저 보낼 수 있는 양방향 통신과는 다르다.) 예를 들면,
우리가 특정 웹페이지(예를들어, sooolog.dev로 생각해보자.)를 들어가기위해 브라우저에 sooolog.dev를
입력하면 서버단에 요청이 들어오고 sooolog.dev에 해당하는 내용들을 브라우저(클라이언트)에게 보내고
연결이 끊기면서 끝나게된다.

http통신은 또한, 위의 tcp통신에서 사용하던 3-way, 4-way handshake 과정을 그대로 거치며,
소켓을 이용하여 소켓 기반 통신이기도 하다.

그렇다면, 여기서 의문이 든다. 위에서 소켓통신은 양방향통신이며 실시간통신이 가능하다고 했는데
왜 http통신은 단방향 통신이 되는것일까? 우선, HTTP통신이 소켓 기반인 이유는 애초에 http 규약은
tcp 규약에 기반하여 만들어졌다. TCP가 있는 계층 위에서 만들어진것이 http규약이기 때문이다.

(그렇기에, http통신은 소켓통신이라고 하지않으며, 소켓 기반의 통신이라고 볼 수 있다.)

<br>

>
> [HTTP규약은 TCP](https://bitcodic.tistory.com/151)
>

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160383387-058a42c6-29cd-40e8-8c25-d7b81fc83304.png">
</p>

4계층 외에 7계층에서도 사용할 수 있는데 실제 실시간 스트리밍 중계나 실시간 채팅을 그 예로 들 수 있다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/160337367-e64ee082-9cf4-4bb4-8dc1-8c2b9391dcba.png">
</p>

정리하자면,,


그거 설명해야지, 그러면 엔진엑스-was, was-api, was-db 이거모두
http통신이고 그러면 http통신에서 똑같이 소켓기반 연결이되며 이 time_wait소켓도
생성이 되는건가. 즉, http통신은 소켓통신은 아니지만 소켓기반 통신이며 3-way,4-way handshake를
http통신이 사용하는것은 물론이며 time_wait 소켓이 발생하는것도 http통신도 가능하냐 이것이다.

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



그리고 그걸 알아야해 TIME_wait 소켓은 언제나 클라이언트단에서 발생해
그리고 http 통신에서는 브라우저에서는 그런게 안생기는데 nginxdls flqjtm vmfhrtldptj
WAS로 보낼때도 클라이언트,서버단의 관계인데 이때 nginx에서 time_wait소켓이 생성되는거다.
- 조졸두 글중

잠깐, time_wait 소켓에 대한 개념을 다시 보자. 정확하게
이게, 한 연결에 대해서만 가능한건지 아니면 다른데서 끌어와서 사용하는건지 알자.
그리고 핸드쉐이크 언제쯤에 이게 세션을 계속 살려두는지도 알자.

네트워크 HANshake 관련글도 다시 정리해야해, 이거 http통신인지
tcp통신인지에 대해 자세히 다시 구별해서 정

그리고 fin을 보내서 연결을 끊는건 서버단에서도 가능하다하는데, 이거 http통신에서는
대부분 클라이언트단에서 fin을 보내서 time_wait가 클라이언트단에서 생긴다는데 이거도
찾아서 정리하


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
