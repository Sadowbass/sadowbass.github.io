---
title: "Java를 이용한 Discord Bot 만들기 #1 - 개발을 하게된 이유"

categories:
- Discord-Bot
tags:
- Java
- JDA
- Discord

last_modified_at: 2022-12-01T20:18:42+09:00

---

먼저 JDA 공식 GitHub부터 둘러보겠습니다.  
[JDA GitHub](https://github.com/DV8FromTheWorld/JDA)  
<br>
gradle로 받을수있는 방법과 documentation등 친절한 설명이 되어있습니다.  
저는 beta 버전이지만 가장 최근버전인 JDA5를 이용하였습니다.  
별도로 프로그램을 유지시키도록 할 코드도 필요없이 main메소드에서 JDA를 이용해서 연결하면 새 스레드에서 웹소켓이 연결되면서 프로그램이 꺼지지 않습니다.
<br>
저는 이미 완성을 시키고 깃에 커밋을 하였기에 제가 만든 최초 main 메서드를 보겠습니다.
```java
public static void main(String[] args) throws InterruptedException {
    jda = JDABuilder.createDefault("token")
            .enableIntents(GatewayIntent.MESSAGE_CONTENT)
            .enableIntents(GatewayIntent.GUILD_MEMBERS)
            .build();
    
    jda.getPresence().setStatus(OnlineStatus.ONLINE);
    jda.awaitStatus(JDA.Status.CONNECTED);
    ...
}
```
전 static field에 JDA를 선언 해놓은 상태여서 main 메서드에서 초기화를 해줍니다.  
`createDefault("token")`에서 token 부분은 디스코드 개발자 페이지에서 앱을 만들고 봇을 만들면 발급해 줍니다.  
<br>
제 깃 첫커밋을 보면 토큰이 나와있는데 이미 새로 받아서 그 토큰은 유효하지 않으니 사용하셔도 동작하지 않습니다.  
<br>
`enableIntents()`부분은 documentation을 보겠습니다. [link](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/JDABuilder.html#enableIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...))  
Intent(의도)를 활성화 시킨다고 합니다.  
<br>
Intent가 무엇인지 해당 클래스의 doc를 확인합니다. [link](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html)  
대충 번역을 해보면 봇에게 필요한 권한들만 골라서 활성화/비활성화를 시키는 enum입니다.  
메세지 관련 intent를 활성화 시키지 않으면 메세지 내용을 봇이 읽을 수가 없으며 길드멤버 intent가 없으면 길드멤버들의 정보나 멘션등이 먹지를 않습니다.  
제가 다 경험해봤으니 넣어줍니다.  
<br>
혹시라도 몰래 intent를 넣어서 나쁜짓을 하실 분들이 있을까봐 말씀드리자면 디스코드에서 봇 프로필을 보면 봇이 갖고 있는 intent가 다 표시되니 나쁜마음은 접어두세요.
![intent](/assets/images/discord_bot/2/2-1.png)  
<br>
`JDABuilder.build()`로 JDA 객체를 만들어주고 presence를 Online상태로 만들면 서버와 봇이 알아서 connection이 되면서 온라인 상태로 봇이 실행되게 됩니다.  
![bot online](/assets/images/discord_bot/2/2-2.png)  
<br>
이제 디스코드에서 발생하는 이벤트에 반응하는 ListenerAdapter를 구현해보겠습니다.

### 참고 링크
1. [JDA GitHub](https://github.com/DV8FromTheWorld/JDA)
2. [JDA5 Documentation](https://ci.dv8tion.net/job/JDA5/javadoc/index.html)
3. [제작한 봇 GitHub](https://github.com/Sadowbass/shotgun-bot)