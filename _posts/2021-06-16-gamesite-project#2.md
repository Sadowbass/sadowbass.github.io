---
title: "첫번째 프로젝트 : Gamesite 프로젝트 생성"

categories:
- GameSite-Project
tags:
- SpringBoot
- JPA
- GameSite Project

last_modified_at: 2021-06-16T10:38:40+09:00

---

본격적으로 프로젝트를 시작해보겠습니다.
일단 제가 프로젝트를 진행할 환경을 먼저 소개를 하자면

1. M1 Macbook Air (macOS Big Sur 11.2.3)
2. Intelli J Ultimate (2021.1.1)

맥과 인텔리제이를 이용하여 진행을 할 것입니다.

제일 먼저 SpringBoot 프로젝트 생성부터 하겠습니다.
프로젝트에서 사용할 자바와 스프링 및 디펜던시들을 먼저 확인하고 가겠습니다.

1. JDK 11
2. Spring Boot 2.5.1 (현재 나온 Snapshot이 아닌 버전중 가장 최근 버전, 사실 제가 블로그 준비하면서 작업할 때는 2.5.0이었는데 그새 올라갔네요)
3. Gradle
4. Lombok

이 정도만 이용할 예정이며 당연히 starter web이나 data jpa는 사용합니다.


# 프로젝트 생성
저는 여기서 인텔리제이를 이용해서 바로 생성할것이며 Ultimate 버전이 아니신분은 [여기](https://start.spring.io)에서 만드셔도 됩니다. 똑같습니다.

![project setting](/assets/images/gamesite/2/1.png)
1. Location은 자신에게 편한 위치로 설정.
2. Maven과 Gradle중 편한 걸 쓰면 되는데 요즘 다들 Gradle을 쓰는 추세라고 해서 Gradle로 진행합니다. (사실 저도 쓸 줄 몰라서 고생 중입니다.\.. 공부할게 끝도 없어요)
3. Group, Artifact 역시 본인에게 편한대로 설정해주세요.
4. Java는 11버전 쓰도록 하겠습니다. 8이상이면 크게 문제는 없는데 새롭게 시작하는 프로젝트에서 충분히 안정화가 되어있는 신버전이 아닌 구버전을 쓰는 건 개인적으로 좋아하지 않습니다.

![dependencies setting](/assets/images/gamesite/2/2.png)
1. Developer Tools에서는 DevTools와 Lombok 추가
2. Web은 Spring Web
3. Template Engines는 Thymeleaf
4. SQL에서 Spring Data JPA와 H2 Database (개발과 테스트 시에 이용하고 추후 원하는 DB로 변경하셔도 다 작동됩니다. 전 나중에 mariaDB로 진행하겠습니다)
5. 마지막 IO에서 Validation을 추가합니다. (예전 버전엔 기본적으로 들어있었는데 어느 순간 빠졌습니다.)

![build.gradle](/assets/images/gamesite/2/3.png)
*이것과 같은 모양의 build.gradle이 생성됩니다*

여기서 저희는 Tomcat을 빼고 Undertow를 넣어보겠습니다. 안 써본 거 다 써보는 게 개인프로젝트의 최장점 아니겠습니까?  
Undertow에 관한 링크
[#1](https://zepinos.tistory.com/35)
[#2](https://zepinos.tistory.com/50)

![exclude tomcat](/assets/images/gamesite/2/4.png)  

25라인의 `'org.springframework.boot:spring-boot-starter-web'`부분을 괄호로 감싸고 중괄호를 쳐서 메소드로 만든 뒤
기본 tomcat을 exclude 후 undertow를 implementation 시켜줍니다.

![exclude tomcat detail](/assets/images/gamesite/2/5.png)
```groovy
	implementation ('org.springframework.boot:spring-boot-starter-web') {
		exclude module: 'spring-boot-starter-tomcat'
	}
	implementation ('org.springframework.boot:spring-boot-starter-undertow')
```
결과는 잠시 후에 확인하겠습니다.

# application.yml 설정
`.properties` 파일의 반복되는 설정이 번잡스러우니 application.properties를 삭제하고 같은 경로에 application.yml을 만들겠습니다.
yml파일의 주의점은 반드시 콜론(:) 뒤에 1칸의 여백이 있어야 하며 상위 property의 하위 property는 앞에 2칸의 여백이 있어야 합니다.
자세한 것은 구글링을 통해 확인하시면 됩니다.

![application.yml](/assets/images/gamesite/2/9.png)  
*org.hibernate.type.descriptor.sql: trace <- descripter 오타입니다 descriptor로 해야 파라미터가 보입니다*  

1. server
    * port: 포트설정입니다.
2. spring
    * datasource
        * driver-class-name: 사용할 DB의 드라이버를 설정합니다. 여기서는 H2를 이용합니다.
        * username: DB username입니다.
        * password: DB password입니다.
        * url: DB접속을 위한 주소입니다. 여기서는 H2 메모리 DB가 아닌 tcp 접속을 이용합니다.
    * devtools
        * livereload.enabled: 매번 프로젝트를 껐다가 켰다가 너무 귀찮음으로 개발시 이를 간편히 rebuild만 해줘도 적용되게 해줍니다.
    * thymeleaf
      * cache: 마찬가지로 스크립트나 CSS 등의 수정 시 캐시에 남아있는 것들을 이용하지 않게 설정한다. devtools를 넣기만 해도 캐시 사용을 안 한다고 하는데 본인은 가끔 계속 캐시에 있는 걸 꺼내와서 명시적으로 설정했습니다.
    * jpa (jpa 부분은 주석을 쳐놨습니다)
3. logging
    * level
        * hibernate가 만들어낸 SQL을 확인하기 위해 로깅설정  

여기까지 했으면 기본적인 설정은 모두 완료되었다고 보면 됩니다.
H2 DB를 실행시키고 스프링부트 프로젝트를 실행하여 보겠습니다.

![run h2 database](/assets/images/gamesite/2/6.png)  
*linux, mac 사용자는 쉘을 실행시키면 되고 윈도우는 bat 파일을 실행시키면 됩니다*

최초 실행 시 `org.h2.jdbc.JdbcSQLNonTransientConnectionException: Database "/Users/seungchulyoo/gamesite" not found, either pre-create it or allow remote database creation (not recommended in secure environments) [90149-200]` Exception이 발생할 수 있습니다.
이는 H2 DB에 필요한 파일을 생성하지 않았으므로 localhost:8082/에 접속.

![jdbc url](/assets/images/gamesite/2/7.png)  
*파란색 JDBC URL에 다음과 같이 작성하고 연결*

그러면 home 디렉토리에 다음 파일이 생성된 것을 확인할 수 있습니다.
![mv file](/assets/images/gamesite/2/8.png)  
*이런 파일이 생성되었다면 다시 한 번 스프링부트로 접속을 시도해봅시다. h2가 정상적으로 실행되었다면 아무런 Exception 없이 실행됩니다.*

실행 로그를 확인해보면 익숙한 톰캣이 아닌 Undertow의 실행 로그도 확인됩니다.
![undertow](/assets/images/gamesite/2/10.png)  

여기까지 최초 설정을 마치고 다음부터 하나씩 만들어보겠습니다.