<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161483356-b3db1b9f-f64f-46cb-97cb-2c0f76995989.png">
</p>

# 🎈 HTTP 구조와 알아두어야 할 기본 요소들

* HTTP에 대한 기본 내용
* HTTP Request와 HTTP Response 구조
* HTTP 메소드들과 Status Code들

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *

<br>



### 1.HTTP에 대한 기본 내용

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161484168-8ce33c96-9910-4ab0-888c-59b23c4ee85a.png">
</p>

HTTP(HyperText Transfer Protocol) 란?         
* HTML(HyperText Markup Language)를 주고받기 위해 만들어진 규약(Protocol)이다.
* 80번 포트를 사용하는 통신 프로토콜이다.
* 비연결성(Connectionless)이자 무상태성(Stateless)이다.
* 비연결성과 무상태성의 단점을 극복하고자 쿠키(Cookie)와 세션(Session)을 사용한다.
* 도메인 + 자원위치(URL), 도메인 + 자원의 식별자(URI) 를 통해서 요청을 하고, 서버가 요청에 따른 HTML 문서응답을 해준다.
* HTML 문서 외에 JSON이나 XML도 주고받을 수 있다.
* HTTP 통신은 요청(Request)과 응답(Response)으로 이루어진다.

<br>

> [HTTP에 대한 기본 내용 (1)](https://velog.io/@doomchit_3/Internet-HTTP-%EA%B0%9C%EB%85%90%EC%B0%A8%EB%A0%B7-IMBETPY)        
> [HTTP에 대한 기본 내용 (2)](https://velog.io/@sehy/Http)      
> [HTTP에 대한 기본 내용 (3)](https://velog.io/@teddybearjung/HTTP-%EA%B5%AC%EC%A1%B0-%EB%B0%8F-%ED%95%B5%EC%8B%AC-%EC%9A%94%EC%86%8C)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161544914-03ff6e7d-8c0d-435d-9d03-765cd80d199b.png">
</p>

비연결성(Connectionless)은 클라이언트와 서버가 한 번 연결을 맺은 후, 클라이언트의 요청에 대해 서버가 응답을 마치면
맺었던 연결을 끊어버리는 특성을 얘기한다.(즉, 서버단이 응답을 마치면 연결이 유지되는게 아니라 바로 끊어지게 되는것을 의미)
또한, HTTP의 무상태성(Stateless)은 통신을 진행할 때 혹은 하고 나서 서버단이 클라이언트단에 대한 상태(다른말로 하면 정보)를 보존하도록 해주는
기능을 제공해주지 않는것을 의미한다. 그렇기 때문에 HTTP 요청을 진행할 때 마다 매번 새로운 요청을 진행하게 되고, 서버단은
같은 클라이언트로부터 요청이 반복적으로 들어와도 해당 클라이언트에 대한 정보를 갖고있지않기에 식별할 수 게 된다.      

실제 예를 보면 더 쉽게 이해된다.    
1.쇼핑몰에 접속    
2.로그인      
3.상품 클릭 -> 상세화면으로 이동     
4.로그인      
5.주문     
6.로그인      
7.....    

이러한 상황을 만들지 않기 위해서 우리는 쿠키와 세션을 사용하여 서버단이 HTTP 요청이 왔을때 특정 클라이언트에 대해
식별할 수 있게 해준다.

<br>

> 네트워크 파트의 쿠키와 세션 개념 글에서 자세하게 다루겠지만, 간단하게만 짚고 넘어가자면 쿠키는 클라이언트단인 브라우저에
> 사용자 정보를 저장하고 이를 HTTP 요청을 보낼 때 함께 보내게하여 서버단에서 해당 정보를 받아볼 수 있도록 한다. 이에 반해,
> 세션은 쿠키를 기반으로 사용되지만 사용자 정보를 브라우저에 저장하는 쿠키와 달리 세션은 서버 측에서 관리한다. 즉,
> 조금 더 쉽게 얘기하자면, session은 비밀번호와 같은 인증 정보를 쿠키에 저장하지 않고, 대신에 사용자 식별자인 session ID를
> 브라우저 쿠키에 저장한다. 이후 서버에 HTTP 요청을 보낼때 session ID를 서버가 받아보게 되고 해당 session ID와 부합하는 것이
> 서버단의 세션에 있으면 서버는 해당 세션의 정보들을 사용하게 된다. 만약 이해를 위해 조금 더 구체적인 예시가 필요하다면 [해당글](https://interconnection.tistory.com/74)을
> 읽어보도록 하자.      
> [쿠키와 세션 (1)](https://velog.io/@cjung5318/%EC%BF%A0%ED%82%A4-%EC%84%B8%EC%85%98-%ED%86%A0%ED%81%B0%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90)          
> [쿠키와 세션 (2)](https://interconnection.tistory.com/74)

<br>

> [HTTP의 비연결성과 무상태성 (1)](https://victorydntmd.tistory.com/286)     
> [HTTP의 비연결성과 무상태성 (2)](https://minjoon950425.tistory.com/109)

<br>



### 2.HTTP Request와 HTTP Response 구조

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161484177-a4fc147c-b430-4636-b31d-a6c98e1fbd33.png">
</p>

위에서 보았듯이, HTTP 요청은 Request(요청)와 Response(응답)로 이루어져 있다.
이제부터 이 HTTP Request(요청)와 HTTP Response(응답)의 구조와 그 안의 요소들에 대해
알아보도록 하겠다.

제일 먼저 알아볼 것은 패킷이다.
패킷은 간단하게 말해서, 데이터를 담은 블록이라고 보면 된다. 패킷 방식의 네트워크 통신에서는
데이터를 주고받을 때 이 패킷을 사용하여 주고 받는다. HTTP 통신 또한 이 패킷을 이용한다.
조금 더 쉽게 말하자면, HTTP 통신시 전송되는 데이터들이 이 패킷이라는 블록안에서 정해진 형식과 구조를 갖추어서 
전송이 되는것이다.

이 블록 형식의 패킷은 HTTP Request(요청)와 HTTP Response(응답)에 따라서
조금은 다른 구조를 갖는데, 이제부터 그 구조와 구조안에 있는 요소들에 대해서 알아보도록 하겠다.

<br>

> [패킷이란 무엇일까 (1)](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%ED%8C%A8%ED%82%B7)       
> [패킷이란 무엇일까 (2)](https://codinggom.github.io/HTTP-%ED%8C%A8%ED%82%B7/)           
> [패킷이란 무엇일까 (3)](https://free-eunb.tistory.com/41)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161484177-a4fc147c-b430-4636-b31d-a6c98e1fbd33.png">
<img src="https://user-images.githubusercontent.com/59492312/161484177-a4fc147c-b430-4636-b31d-a6c98e1fbd33.png">
</p>

**HTTP 요청(Request)**     
클라이언트(사용자)가 서버에 HTTP Request(요청)하는 경우를 의미한다.

HTTP 요청(Request)을 보면 구조가 크게 3부분으로 나누어진다.   
  
* start line
* headers
* body    

3부분을 하나하나 봐보도록 하겠다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161952061-41e20e88-b8a7-4b8d-be80-e1372a440544.png">
</p>

제일 먼저 **start line**이다.    
말 그대로 HTTP Request의 첫 라인이다. start line은 3가지 요소로 구성되어져 있다.    

1. Request URL : 해당 요청(Request)의 목표 url을 의미한다. 바로 위의 이미지를 보면 
  https://www.naver.com이라고 되어 있는것이 보인다.(필자는 크롬 브라우저에서 https://naver.com를
  입력하고 개발자도구의 Network탭에 들어갔기 때문이다.)

2. Request Method :  해당 요청에 대한 종류(액션)를 정의하는 부분이다. 위 이미지 예시를 보면
  GET이라고 적혀져 있다. 주로 GET과 POST가 쓰이며 그 외에 PUT, DELETE, OPTION등이 있다. 이러한
  메서드들에 대해서는 아래에서 다시 얘기하도록 하겠다.

3. HTTP Version : 사용되는 HTTP 버전 (주로 1.1 버전이 널리 쓰인다.)

<br>

> 위의 실제 예로 보여준 이미지와 앞으로 보여줄 이미지들은 모두 크롬 브라우저의 F12(개발자 도구)를 눌러서
> Network탭에 있는 여러요소중 하나를 클릭해서 보여준것이다. 이렇게 실제 사용하는 예를 들어서 알아가면 더 많은 도움이
> 되니 참고하도록 하자.

<br>

> start line을 request-line이라고 부르기도 하고, request-url을 request target이라고 부르기도
> 한다. 또한, Request Method 대신, HTTP Method라고 부르기도 한다.    
> [start line을 request-line으로 (1)](https://velog.io/@doomchit_3/Internet-HTTP-%EA%B0%9C%EB%85%90%EC%B0%A8%EB%A0%B7-IMBETPY)    
> [start line을 request-line으로 (2)](https://blog.wanzargen.me/20)    
> [request target과 HTTP Method (1)](https://velog.io/@teddybearjung/HTTP-%EA%B5%AC%EC%A1%B0-%EB%B0%8F-%ED%95%B5%EC%8B%AC-%EC%9A%94%EC%86%8C)     
> [request target과 HTTP Method (2)](https://velog.io/@sehy/Http)

<br>

> []()     
> []()    

<br>

1.http 그 crlf인가 하는거 구조
+
2.General정리된것도 다시
https://blog.cordelia273.space/11
+
3.소켓도
+
4.content-type등 자세한 것도 있따.



<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161952082-cc529db6-5b0f-4199-b2a4-54adc4e94eb9.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161952087-84959da6-5a12-4c1b-afd9-7294239f321e.png">
</p>
  
<2> **headers** - 
1.    
  
<3> body     
  
<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161689682-1bb606e1-be0b-4ff3-9e64-78d8a05b376e.png">
</p>

**HTTP 응답(Response)**      
서버가 사용자의 요청을 받고 클라이언트(사용자)에 HTTP Response (응답)하는 경우를 의미한다.

HTTP 응답(Response)도 구조가 크게 3부분으로 나누어진다.     
* start line
* headers
* body    

하나하나 봐보도록 하겠다.

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161952061-41e20e88-b8a7-4b8d-be80-e1372a440544.png">
</p>

예시 이미지를 보면 알겠지만 HTTP Request에 대한 내용과 HTTP Response에 대한 내용이
함께 있다.

<1> start line
  
<2> headers    
  
<3> body     

<br>

<awefa~

그런데, 크롬의 네트워크탭은 이 반응과 요청을 다 한꺼번에 보여주는거같다.

<br>



### 3.HTTP 메소드들과 Status Code들

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/161484168-8ce33c96-9910-4ab0-888c-59b23c4ee85a.png">
</p>

a

<br>



### 🚀 추가로

<br>

태그 : #

> 처럼 띄우는거에는 항상 위아래로 <br> 붙여주기
