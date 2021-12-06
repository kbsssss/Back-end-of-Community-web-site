<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/144818416-a457a8ac-95cf-4fad-8be5-f6d2f612ef6e.png" width="700px" height="350px">
</p>

# 🤔 Java SE,EE 그리고 EJB와 스프링은 어떤 관계일까?    

문뜩, 스프링에서는 Java SE가 쓰이는걸까 아니면 Java EE가 쓰이는걸까 궁금해졌다. 인텔리제이에서 SDK 설정을 하려면 기본적으로 JDK를 다운받아서 설정하여야 하는데, 그렇다면,Java EE는 Enterprise Edition으로 Java SE 스펙기반으로 자바를 이용한 서버측 개발을 위한 플랫폼이라는데, 이것은 무슨말일까?
아래 정리된 내용을 보면서 궁금증을 해결해보자.
- - -

#### 1. Java SE란 무엇인가 ?
자바 SE는 보통 우리가 자바 JDK를 다운로드 받는다거나 자바를 다운로드 받는다 할때 대부분이 이 자바 JDK를 의미한다. 자바 SE = 스탠다드 에디션으로, 핵심 Java 프로그래밍 플랫폼이다. 여기에는 Java 프로그래머가 알아야하는 모든 라이브러리와 API가 포함되어 있다. (java.lang, java.io, java.math, java.net, java.util 등).

자바 관련 프로젝트라면 Java SE를 다운받아서 사용해야한다.

> 즉, 자바 프로젝트를 해보았다면, 우리가 기본적으로 가져다 쓰는 클래스들(java.util, java.lang..등이 모두 이 Java SE에서 가져다 쓴다는 말이다.

<br>       

#### 2. Java EE란 무엇인가 ?
그렇다면 Java EE는 무엇인가? 우선 쉽게 설명하자면 Java EE는 Java Enterprise Edition으로 언뜻보기에 기업용 자바라는 느낌을 준다. 조금 더 자세히 보도록 하겠다.

Java SE 스펙기반으로 자바를 이용한 서버측 개발을 위한 플랫폼이다. 즉, Enterprise Applications에 사용하기 위한 플랫폼이다. Java EE 플랫폼은 Enterprise Application 개발에서 발생하는 여러 문제들을 해결해준다.
[[Enterprise Applications란?]](https://velog.io/@jihoson94/Java-SE-vs-EE)

> 정리하자면, Java EE는 Java SE보다 다소 기업적 차업이나 서버측 측면에서 쓰이는 플랫폼이다. 그렇다면, EBJ와 스프링은 무엇이고 어느 자바 플랫폼 기반일까?

<br>

#### 3. EJB와 스프링
예전 스프링이 없었을 당시에는 자바로 Enterprise Application을 개발하는 가장 일반적인 방법은 EJB(Enterprise Java Beans)였다. 즉, 자바EE를 활용하여 EJB라는 기술로 개발하는것이다. 그러나 EJB에는 여러가지 단점이 있었다.

궁금하다면,
[[EJB의 단점들]](https://velog.io/@outstandingboy/Spring-%EC%99%9C-%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C-Spring-vs-EJB-JavaEE)

그러기에, Spring이 나오게 된것이고, 이는 Java SE기반으로 돌아가며 J2EE 표준의 복잡성에 대응하기 우해 만들어졌다. 스프링은 Java EE인 EJB의 단점들을 보완해서 만들어진 결과물인것이다.

> 즉, EJB는 자바 EE기반 기술이며 단점이 많아 자바 SE기반의 스프링이 이를 대체하였다.

<br>

🎈 추가로,Java SE와 J2SE 그리고 Java EE와 J2EE는 같은 말이다.
[J2SE와 J2EE](https://animal-park.tistory.com/97)

## 🪁 Reference
* [Java SE의 개념](https://linked2ev.github.io/java/2019/04/29/JAVA-1.-JAVA-SE-EE-ME/)
* [Java EE의 개념](https://linked2ev.github.io/java/2019/04/29/JAVA-1.-JAVA-SE-EE-ME/)
* [EJB는 J2EE의 핵심기술](https://okky.kr/article/415474)
* [J2EE에서 핵심역활을 하는 EJB](https://noahlogs.tistory.com/46)
* [J2EE를 대체하는 스프링1](https://velog.io/@jihoson94/Java-SE-vs-EE)
* [J2EE를 대체하는 스프링2](https://velog.io/@outstandingboy/Spring-%EC%99%9C-%EC%8A%A4%ED%94%84%EB%A7%81-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%A0%EA%B9%8C-Spring-vs-EJB-JavaEE)