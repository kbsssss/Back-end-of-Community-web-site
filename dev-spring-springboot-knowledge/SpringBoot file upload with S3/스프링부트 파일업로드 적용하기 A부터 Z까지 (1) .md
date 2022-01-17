태그 : #파일업로드 #보안그룹 #디폴트 보안그룹 #

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148671233-623b59f6-6ec4-49f7-90a8-8ee1606a42c6.png">
</p>

# 📖 스프링부트로 파일업로드 적용하기 A부터 Z까지 (1)

* ec2 인스턴스 생성
* rdb 인스턴스 생성

> 모든 코드는 [깃헙](https://github.com/sooolog/dev-spring-springboot)에 작성되어 있습니다.
* * *



### 1.EC2인스턴스를 생성해보겠다.(ec2인스턴스 생성, rdb인스턴스 생성을 한경우 글 (2) 로 넘어가면 된다.)

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925274-36458791-639b-4f58-8f58-164633c9db28.png">
</p>

필자는 커널 4.14 Linux2를 선택해주었다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925256-89e2da07-4a62-4a40-987a-a2b87e477465.png">
</p>

t2.micro가 프리티어에서 무료로 사용할 수 있는 인스턴스이니 선택해주었다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925270-4c0beacb-2e09-4892-8413-50833fb20930.png">
</p>

그 후의, 단계3의 인스턴스 세부 정보 구성은 '다음:스토리지추가'를 눌러주면 위의
화면과 같은것이 뜬다.

여기서, 우리는 Gib(기가비트)를 프리티어에서는 30GiB까지 사용할 수 있다.
조금 더 얘기하자면, 통상 이 스토리지의 크기에 따라서 IOPS(초당 입출력 처리량)도
증가 하게 되는데, 그러면, 조금 더 빠른 성능 개선을 도와주게 된다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925275-71987e2b-3568-4270-a0a5-6329b71fbfee.png">
</p>

다음으로 가서 태그 추가를 눌러주고, 키에 Name이라 적어주며 값에는 

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925280-2a95cf13-88fc-4fd7-b645-48473e5e53bf.png">
</p>



<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148925281-5fca576a-5d2b-4ab2-8075-d64dfe63c156.png">
</p>

#### 🪁 Reference
* 참조링크 : []()
* 참조링크 : []()

<br>



### 2-1.그다음은 파일과 그 외의 기본내용들을 저장하려 DB을 만들어보겠다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148872538-9c24fbba-76a0-4e05-889a-2d10ce462fa2.png">
</p>

데이터베이스 인스턴스를 생성하는 방법은 핵심만 심플하게 알아보고 넘어가도록 하겠다.<br>
해당 데이터베이스는 MariaDB를 사용하였으며, 프리티어를 사용하였다. 이외에 자세한 설명은 다른 웹사이트를
참조하길 바란다.<br><br>

위에 보여지는 DB 인스턴스 식별자는 DB 인스턴스의 이름이다. 그 외의 마스터 사용자 이름과 마스터 암호는 후에
해당 DB 인스턴스에 접근하려면 알고있어야 하니 기억해 두도록 하자.는

> DB인스턴스를 만들어보면, 맨처음 적는란의 DB 인스턴스 식별자 외에, 데이터베이스 옵션의 초기 데이터베이스
> 이름을 적는란이 있다. 그 차이점은 DB 인스턴스 식별자 현재 AWS리전의 DB인스턴스 자체에 대한 이름으로 해당 AWS
> 리전에서 고유한 이름이여야 하고, 초기 데이터베이스 이름은 우리가 실제 접근해서 사용하는 실 데이터베이스의 이름이다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/147551855-1ec80def-5a7f-46b0-83c8-d2f3c715b9a2.png">
</p>

데이터베이스를 생성한 이후에는, 파라미터 그룹을 생성해보도록 하겠다. 그림처럼 rds목록에서 파라미터
그룹을 클릭하면 된다. 그 후에 파라미터 그룹 생성을 클릭해주자.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148872252-750e1d58-6530-47ed-8e72-df532aaaa0e5.png">
</p>

파라미터 그룹이름은 rds와 동일하게 해주었다. 또한, 데이터베이스의 버전도 이전 DB인스턴스의
버전과 동일하게 해주어야 한다. 

완료되었다면, 파라미터 그룹을 생성해준다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714048-b9a362ce-0788-4f88-bd1f-7308aafc319b.png">
</p>

생성된 파라미터그룹을 들어간 후에 파라미터 편집을 눌러준다. 그래야, 파라미터의 여러 설정들을 바꿔줄 수 있다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714054-82b57430-28ac-4fd6-b7ad-ed9eb7870914.png">
</p>

총  종류의 설정을 할것인데, 첫번째가 time_zone이다. 이를 Asia/Seoul로 바꿔준다.
궁금증이 생길 수 있다. 이것을 왜 바꾸어주어야 하는지. 아래 참조링크를 보면 알겠지만, DB의
기준이 되는 시간은 Seoul에 맞춰진 시간이 아니다. NOW()라는 쿼리문으로 실행해보면, DB의
기준시간과 Seoul이 다르기 때문에, 나중에 시간을 다루는 쿼리문을 할때 오차가 생길 수 있기 때문이다.
자세한것을 아래 참조링크를 참고하도록 하자.

> 참조링크 : [time_zone 설정 이유](https://programforlife.tistory.com/52)

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714115-8a7d0065-19f7-4c8c-b63e-995c9d99ac55.png">
</p>

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714130-15e92cd2-3006-49ca-a438-71c21ebd5961.png">
</p>

그 다음 바꾸어주어야 할 것이 Character Set이다. 총 6개의 character set을 바꾸어주면된다.     
character-set-client    
character-set-connection    
character-set-database     
character-set-filesystem    
character-set-results    
character-set-server    

값은, utfmb4로 지정해준다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714149-a4bc505d-9dde-4966-8b37-0bb4863246b6.png">
</p>
<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714168-6bec70ec-a8f5-4347-9abe-876d523bb777.png">
</p>

세번째로 바꾸어주어야 할 것이 collation이다.         
collation_connection  
collation_server     
이렇게 두개를 utf8mb4_general_ci 혹은 utf8mb4_unicode_ci로 바꾸어준다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148872257-67b7c3e4-196f-4632-a4b8-23b23fe4ff92.png">
</p>

변경된 파라미터 그룹을 저장하고 다시 DB목록으로 돌아온다음에, 처음 만들었던 db를 선택하고
수정을 누른다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148872255-861c6a49-ad44-49fa-979a-47b097b2caba.png">
</p>

그 이후에, DB파라미터 그룹으로 이동해서, 옵션 그룹은 그대로 두고 DB 파라미터 그룹만
방금전에 만든 파라미터 그룹으로 변경해서 저장만 해주면 된다.

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148714192-70727e83-0622-4df5-b71a-2c5ad02ea681.png">
</p>

그 이후에, 예약된 다음 유지 관리 기간에 적용을 하지 않고, 즉시 적용을 선택한다.
예약된 다음 유지 관리 기간이란 새벽시간대 등에 적용되기 때문에, 현재 실제 서비스 배포중이 아니니
바로 적용을 해주는게 맞다.(적용하는 시간에는 DB가 작동하지 않기 때문이다.)

DB 인스턴스 수정을 클릭하면, DB가 수정중이라 나오는데, 끝나고 나서도 재부팅을 한 번 해주자.
간혹 적용되지 않는 경우가 있다. 

> 추가로, DB인스턴스나 아니면 파라미터 그룹을 생성한다고 해서 보안그룹이 생성되는것은 아니다.
> 바로 뒤의 보안그룹에 대해 설명하도록 하겠다.

#### 🪁 Reference
* 참조링크 : [AWS 콘솔](https://console.aws.amazon.com/)

<br>



### 2-2.DB에 

2.unicode 안하는이유
3.my.cnf이거 확인k

5.새로운 test로 셋팅하기 aws
6.collation이랑 이런거 체크해서 보여줘서 추가 utf8mb4설

#### 🪁 Reference
* 참조링크 : [AWS 콘솔](https://console.aws.amazon.com/)

<br>



### 3.



### 🚀 추가로

<p align="center">
<img src="https://user-images.githubusercontent.com/59492312/148872254-e4b0946b-6017-4bcc-b12a-ca7a5bbc7243.png">
</p>

보이는바와 같이, 해당 default보안 그룹은 aws계정을 만들고나서 바로 생겨져있는
보안그룹이다. 즉, ec2나 rds 인스턴스를 생성했다고 생기는 보안그룹이 아니다. 또한, 삭제도
불가능하다. 인지하고 있으면 좋을 것 같다.
