<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/166934711-797918c8-e3e8-4bc9-a44d-e717cd2a073c.png">
</p>

# 📖 Beanstalk환경과 nginx기반에서 멀티도메인, 서브도메인, HTTPS 리다이렉팅

* 멀티도메인과 서브도메인 Route53 설정
* HTTPS 리다이렉팅을 위한 기본 세팅
* nginx.conf 설정을 이용하여 멀티도메인, 서브도메인, HTTPS 리다이렉팅


하위도메인(www.도메인)연결과 리다이렉팅
* 도메인.net, 도메인.co.kr을 도메인.com으로 리다이렉팅
* 하위도메인(m.도메인)설계와 리다이렉팅
* http:// -> https:// 리다이렉팅

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.

* * *


<br>



### 1.멀티도메인과 서브도메인 Route53 세팅

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/168206626-c716f392-2c89-46c2-9716-692b7b2f3174.png">
</p>

멀티도메인과 서브도메인의 리다이렉팅에 대해 알아보도록 하겠다.

흔히 우리는 도메인A.com을 주 도메인으로 사용하고싶은데 나머지 도메인A.co.kr이나 도메인A.net을 구매하여
이를 통한 클라이언트의 요청이 들어올시 다시 도메인A.com으로 접속되어 보여지게 하고싶어한다. 이를 멀티도메인 리다이렉팅이라고
하고, www.도메인A.com 처럼 서브도메인을 다시 도메인A.com으로 리다이렉팅하는것은 서브도메인 리다이렉팅이라고 한다.

이제부터, 멀티도메인과 서브도메인 리다이렉팅을 위한 기본 Route53 세팅부터 보도록 하겠다.
기본적으로 Route53과 도메인 설정을 안다는 가정하에 필요한 부분만 보고 넘어가도록 하겠다.
위 화면을 보면, 기본 도메인A.com(도메인A라고 통힐하겠다.)을 A레코드로 설정하였고(로드 밸런서로 설정하였다.), www.도메인A.com
또한 도메인A.com과 같은 라우팅 값을 설정해준것이다.(m.도메인A.com을 제외한 SOA, NS 유형은
기본적으로 route53 호스팅영역을 만들면 생성되는 유형이고 위의 CNAME 유형은 SSL/TLS 인증서를 받기위한
DNS 검증 레코드이다.)

> 흔히 우리는 도메인A.com으로 서비스를 내면, 도메인A.net이나 도메인A.co.kr도 한번에
> 구매를 하여 도메인A.com으로 리다이렉팅을 한다. 이때 여러 도메인을 하나의 서버로 연결해서
> 사용하는데, 이 때 사용된 모든 도메인을 멀티도메인이라고 한다. 또한 최상위도메인(com, co.kr, net)이
> 다른 도메인 외에 도메인B.com 같이 도메인명 자체가 다른 경우에도 다른 도메인들과 같은 서버를 가리키게 되면
> 도메인B.com도 멀티도메인이라 한다.     
> [멀티도메인이란](https://post.naver.com/viewer/postView.nhn?volumeNo=24092785&memberNo=11287836)

> m.도메인도 서브도메인 리다이렉팅에 속하긴하나, 이는 모바일 혹은 태블릿을 인지하고
> 디바이스별 리다이렉팅이 필요하기 때문에 별도의 설정이 필요하다. 또한, DNS인 Route53에서
> 불필요한 리소스 낭비를 막기위한 추가설정에 관해서도 설명해야하니 필요시 필자가 작성한 글을 보도록 하자.    
> [Beanstalk환경과 nginx기반에서 디바이스별 리다이렉션](https://sooolog.dev/Beanstalk%ED%99%98%EA%B2%BD%EA%B3%BC-nginx%EA%B8%B0%EB%B0%98%EC%97%90%EC%84%9C-%EB%94%94%EB%B0%94%EC%9D%B4%EC%8A%A4%EB%B3%84-%EB%A6%AC%EB%8B%A4%EC%9D%B4%EB%A0%89%EC%85%98/)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/168217003-90ef1421-cc26-4e9f-9658-fca00db63abc.png">
</p>

위의 것과 동일한 도메인A이나 최상위도메인이 com이 아닌 net이다. 
Route53 호스팅영역을 생성할 때 기본적으로 최상위도메인이 다르면 하나의 최상위도메인당
하나의 호스팅영역을 생성해주어야 한다.

동일하게 레코드 유형이 NS,SOA인것은 호스팅영역을 생성할 때 기본적으로 생성되는 레코드 유형이고
CNAME 두개는 SSL/TLS 인증서를 발급받기 위한 DNS검증을 위해 사용된 레코드 이다. 나머지는 도메인A.net
과 www.도메인A.net이 빈스톡의 로드 밸런서를 가리키는 A유형의 레코드이다.

> 최상위 도메인당 하나의 호스팅영역을 생성해주어야 하는것과는 반대로, 서브도메인은
> 주로 도메인.com의 호스팅영역 내에서 추가적으로 설정하여 사용한다. aws 에서도 서브도메인에
> 대해서 따로 독립적인 호스팅영역을 만드는걸 권장하지 않는다. 하지만, 예외가 있는데 바로 
> m.도메인.com의 경우이다.(이는 디바이스별 리다이렉션에 자세하게 정리해 놓았다.)       
> [Beanstalk환경과 nginx기반에서 디바이스별 리다이렉션](https://sooolog.dev/Beanstalk%ED%99%98%EA%B2%BD%EA%B3%BC-nginx%EA%B8%B0%EB%B0%98%EC%97%90%EC%84%9C-%EB%94%94%EB%B0%94%EC%9D%B4%EC%8A%A4%EB%B3%84-%EB%A6%AC%EB%8B%A4%EC%9D%B4%EB%A0%89%EC%85%98/)

<br>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/167347850-7053e250-826b-49d7-985a-14f7f3e9e910.png">
</p>

위의 자료는 최상위 도메인 co.kr에 대한 설정이다.
방금 전 설명한것과 동일한 레코드들이다.

여기까지하면, 서브도메인과 멀티도메인에 대한 Route53 설정은 끝이 난다.

<br>



### 2.HTTPS 리다이렉팅을 위한 기본 세팅

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/167347854-0b027007-1e80-492c-be84-47b409fed769.png">
</p>

여기서는 따로 빈스톡기반 환경에 도메인 HTTPS 연결을 다루지 않고 이미 연결이 되있다는 가정하에 진행하도록 하겠다.
빈스톡환경에서 SSL/TLS 인증서를 발급받아 HTTPS를 적용하는 방법은 필자가 작성한 글을 보면 쉽게 따라할 수 있다.
(아래 참조링크가 필자가 작성한 글이다.)

이곳에서는 HTTP로 접속했을 시 HTTPS로 리다이렉트하여 접속할 수 있도록 설정하려 한다.

위와같이 도메인에 접속하였을 때 자물쇠가 채워져 있으면 HTTPS가 정상적으로 작동하고
있다는 것이다.

> [Beanstalk기반 환경에 도메인 SSL 설정하기](https://sooolog.dev/Beanstalk%EA%B8%B0%EB%B0%98-%ED%99%98%EA%B2%BD%EC%97%90-%EB%8F%84%EB%A9%94%EC%9D%B8-SSL-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0/)

<br>



### 3.nginx.conf 설정을 이용하여 멀티도메인, 서브도메인, HTTPS 리다이렉팅

```conf
  server {
      listen 80;
      server_name celebmine.net www.celebmine.net;

      set $mobile_rewrite do_not_perform;

      if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge\ |maemo|midp|mmp|mobile.+firefox|netfront|opera\ m(ob|in)i|palm(\ os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows\ ce|xda|xiino [NC,OR]") {
          set $mobile_rewrite perform;
      }

      if ($http_user_agent ~* "^(1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a\ wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r\ |s\ )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1\ u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp(\ i|ip)|hs\-c|ht(c(\-|\ |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac(\ |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt(\ |\/)|klon|kpt\ |kwc\-|kyo(c|k)|le(no|xi)|lg(\ g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-|\ |o|v)|zz)|mt(50|p1|v\ )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v\ )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-|\ )|webc|whit|wi(g\ |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-) [NC]") {
          set $mobile_rewrite perform;
      }

      if ($mobile_rewrite = perform) {
          rewrite ^ http://m.celebmine.com$request_uri? redirect;
          return 301;
          break;
      }

      return 301 https://celebmine.com$request_uri;

      access_log    /var/log/nginx/access2.log main;
  }


  server {
      listen 80;
      server_name celebmine.co.kr www.celebmine.co.kr;
      return 301 https://celebmine.com$request_uri;

      access_log    /var/log/nginx/access13.log main;
  }

  server {
      listen        80 default_server;
      listen        [::]:80 default_server;

      if ($host = www.celebmine.com) {
          return 301 http://celebmine.com$request_uri;
      }

        set $mobile_rewrite do_not_perform;
        if ($http_user_agent ~* "(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge\ |maemo|midp|mmp|mobile.+firefox|netfront|opera\ m(ob|in)i|palm(\ os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows\ ce|xda|xiino [NC,OR]") {
            set $mobile_rewrite perform;
        }
        if ($http_user_agent ~* "^(1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a\ wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r\ |s\ )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1\ u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp(\ i|ip)|hs\-c|ht(c(\-|\ |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac(\ |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt(\ |\/)|klon|kpt\ |kwc\-|kyo(c|k)|le(no|xi)|lg(\ g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-|\ |o|v)|zz)|mt(50|p1|v\ )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v\ )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-|\ )|webc|whit|wi(g\ |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-) [NC]") {
            set $mobile_rewrite perform;
        }

        if ($mobile_rewrite = perform) {
            rewrite ^ http://m.celebmine.com$request_uri? redirect;
            return 302;
            break;
        }


      location / {
          proxy_pass          http://springboot;
          proxy_http_version  1.1;
          proxy_set_header    Connection          $connection_upgrade;
          proxy_set_header    Upgrade             $http_upgrade;

          proxy_set_header    Host                $host;
          proxy_set_header    X-Real-IP           $remote_addr;
          proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
      }

      access_log    /var/log/nginx/access.log main;

      client_max_body_size  10m;
      client_header_timeout 60;
      client_body_timeout   60;
      keepalive_timeout     60;
      server_tokens         off;
      gzip                  on;
      gzip_comp_level       4;

      # Include the Elastic Beanstalk generated locations
      include conf.d/elasticbeanstalk/healthd.conf;
  }
```

전체 설정코드는 위와같다.
위의 코드의 의미는 

그럼 이제 멀티도메인, 서브도메인, HTTPS 리다이렉트를 위한 각각의 코드들에 대해
자세히 알아보도록 하겠다.

음,, 아예 STatus코드 반환하는게 낫나 아니면 그냥 리다이렉트가 낫나


<br>

```conf
a
```

a

<br>

```conf
a
```

a

<br>



### 4.마지막으로 default_server 설정으로 보안 강화하기

```conf
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # Everything is a 444
        location / {
                return 444;
        }
}

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # Everything is a 444
        location / {
                return 444;
        }
}
```

a

<br>


eturn 301 아래에 expires epoch; 을 붙여주는 것은 301 리다이렉트가 캐싱되지 않도록 하기 위해서
https://xetown.com/tips/1172256
실제로 WWw.celebmine.net이 내 브라우저에 캐싱되어서 계속 네이버 뜬다.
근데 스마트폰은 정상적으로 잘 뜸

https://xetown.com/tips/1172256
여기서는 rewrite를 쓰지 말라한다.

www seo측면이랑 싹 정리하자.
https://xetown.com/questions/1131734

그거도 해야해, 원래 route53할때 서브도메인하지말라해
근데 m.도메인은 해야해

아 뭔데 뭔 NS(네임서버)야, 그러니까 왜 m.celebmine.com에 대해 CNAME이
아닌 NS로 설정하라고 AWS에서 얘기하는거지 ?

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
