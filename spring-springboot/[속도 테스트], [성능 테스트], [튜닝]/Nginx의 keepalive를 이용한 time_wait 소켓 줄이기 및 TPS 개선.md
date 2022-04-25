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

첫번째로, keepalive적용할때 이것도 같이 생각해 worker_connection 개수 1024이고 X worker_process가 코어 개수잖아
근데 이게 nginx뿐만이 아니라 모든 연결의 기준인지 그리고 이걸 몇개로 조절할것인지도

여기 아래에 보면 알겠지만 개발자도구 네트워크 탭에서 keepalive가 쓰였는지 안쓰였는지 알 수 있다.
https://www.lesstif.com/system-admin/nginx-gzip-59343019.html

추가로 크롬 개발자도구 네트워크 탭에서 keepalive사용되는지 응답 헤더인가 볼 수 있는것도 정리하자.

<br>

#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2.

<p align="center">
<img src="">
</p>

https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=noorol&logNo=140199393456
여기보면 Keepalive를 사용하는 경우 메모리도 신경써야한다는 글이다. 꼭 읽자.
근데 아래 반박 링크를 보면, Nginx 개발자에 따르면 10,000개의 유휴 연결(keepalive)은 2.5MB의 메모리만 사용하므로
Nginx는 연결 유지로 인해 유휴 연결을 처리하는데 매우 뛰어나다고 한다. 메모리는 별로 신경안써도 될것같은데 그런데 여기서 nginx만
얘기하는데 WAS와 db와의 연결도 keepalive적용되는거 아닌가 ? 아 맞다. 이거는 keepalive가 아니라 커넥션풀이나 쓰레드 관련인가 ????
https://ciksiti.com/ko/chapters/9149-what-is-keepalive-in-nginx

아니 그것도 해야해, 분명 worker_connection은 분명 was랑 nginx뿐만아니라 was랑 db연결도 관련이 있다했잖아
그럼, 우리가 nginx의 upstream에서 지정한 keepalive가 nginx에서 was가는것만 신경써야할게 아니라 was에서
db로 가는것도 신경써야하는거 아닌가 ? 이다. 그래서 keepalive가 was랑 db랑 소통하는것도 적용되는게 아닌가 ?

아니 우선, keepalive를 하기에 앞서 브라우저에서 nginx는 http통신이고, nginx에서 was 그리고 was에서 외부 api, 그리고 was
에서 데이터베이스 연결은 http 통신이 아니잖아 tcp통신 기반이긴 하지만, 그러면 keepalive를 적용하면 나는 nginx에다가 keepalive를 적용할건데
그러면, 브라우저에서 nginx으로의 통신과 그리고 nginx에서 was로의 통신만 keepalive가 적용되는거아닌가, 데이터베이스나 api는 말고

https://velog.io/@kyle-log/HTTP-keep-alive

클라이언트가 요청을 한 후 응답을 받으면 그 연결을 끊어 버리는 특징
HTTP는 먼저 클라이언트가 request를 서버에 보내면, 서버는 클라이언트에게 요청에 맞는 response를 보내고 접속을 끊는 특성이 있다.
헤더에 keep-alive라는 값을 줘서 커넥션을 재활용하는데 HTTP1.1에서는 이것이 디폴트다.
HTTP가 tcp위에서 구현되었기 때문에 (tcp는 연결지향,udp는 비연결지향) 네트워크 관점에서 keep-alive는 옵션으로 connectionless의 연결비용을 줄이는 것을 장점으로 비연결지향이라 한다.
https://interconnection.tistory.com/74

웹서버 keepalive도 참고

http keepalive에 대해서, 이게 socket i/o가 완전히 끝나고부터 시작되는 시간 기준이라는
등 이러한 자세한 내용에 대해서 한번 정리


#### 🪁 References
* 참조링크 : []()
* 참조링크 : []()

<br>



### 🚀 추가로

<br>

태그 : #

> 처럼 띄우는거에는 항상 위아래로 <br> 붙여주기