---
title: "Java를 이용한 Discord Bot 만들기 #3 - ListenerAdapter 구현"

categories:
- Discord-Bot
tags:
- Java
- JDA
- Discord

last_modified_at: 2022-12-02T11:58:08+09:00

---

이제 본격적으로 디스코드에서 발생하는 이벤트에 대응하는 ListenerAdapter를 구현해보겠습니다.  
<br>
JDA는 굉장히 여러가지 이벤트에 대응하는 메서드들을 제공하고 있습니다.  
메시지가 왔을 때, 수정 되었을 때, 누군가 가입했을 때, 모든 이벤트 등등  
저희는 그저 어느 타이밍에 원하는 기능을 구현만 하면 됩니다.  
![override methods](/assets/images/discord_bot/3/3-1.png)  
세션, 특정 이벤트등 원하는 이벤트에 해당하는 메서드를 오버라이드하면 끝!  
<br>
백문이 불여일견으로 간단한 클래스를 직접 만들어 보겠습니다.  
지난번 2번째 포스트에서는 기존 코드를 이용했는데 이번부터는 아예 새로 처음부터 만들고 그것을 어떻게 수정하였는지는 기존 코드를 이용해서 설명하는 것으로 살짝 노선을 바꾸었습니다.  

### Main
```java
package example;

import net.dv8tion.jda.api.JDA;
import net.dv8tion.jda.api.JDABuilder;
import net.dv8tion.jda.api.OnlineStatus;

public class BotMain {

    public static void main(String[] args) throws InterruptedException {
        JDA jda = JDABuilder.createDefault("token")
                .build();

        jda.getPresence().setStatus(OnlineStatus.ONLINE);
        jda.awaitStatus(JDA.Status.CONNECTED);

        jda.addEventListener(new TestListenerAdapter());
    }
}
```  
토큰은 보시는 분들이 discord developer 페이지에서 발급받아 넣으시면 됩니다.  
Intent 없이 최대한 단순히 만들어줍니다.  
그리고 밑에서 만들어줄 첫 ListenerAdapter를 jda에 addEventListener로 추가시켜줍니다.
### First ListenerAdapter
```java
package example.listener_adapter;

import net.dv8tion.jda.api.events.GenericEvent;
import net.dv8tion.jda.api.events.message.MessageReceivedEvent;
import net.dv8tion.jda.api.hooks.ListenerAdapter;
import org.jetbrains.annotations.NotNull;

public class TestListenerAdapter extends ListenerAdapter {

    @Override
    public void onGenericEvent(@NotNull GenericEvent event) {
        System.out.println("TestListenerAdapter.onGenericEvent");
        System.out.println("event.getClass() = " + event.getClass());
    }
  
    @Override
    public void onMessageReceived(@NotNull MessageReceivedEvent event) {
        System.out.println("TestListenerAdapter.onMessageReceived");
        System.out.println("event.getMessage().getContentRaw() = " + event.getMessage().getContentRaw());
    }
}
```  
원래는 sout이 아니라 slf4j로 출력해야 하지만 예제에서는 편의를 위해 System.out을 이용합니다.  
모든 이벤트에 반응하는 `onGenericEvent`와 메시지가 왔을 때 반응하는 `onMessageReceived` 두 가지를 Override 합니다.  
그리고 자신의 채널에 봇이 추가되어있는지 확인하고 메시지를 보내보겠습니다.  
![check add bot](/assets/images/discord_bot/3/3-4.png)  
TestExampleBot이 새로 만드는 봇입니다.  
그 후 아무런 메시지를 보내고 콘솔을 확인합니다.  
![console result](/assets/images/discord_bot/3/3-5.png)
원하는 메시지는 거의 나왔는데 `onMessageReceived`에서 이상한 문구가 출력되었습니다.  
간단하게 메시지를 확인해보면  
`Discord now requires to explicitly enable access to this using the MESSAGE_CONTENT intent.`  
이제 메시지를 확인하려면 MESSAGE_CONTENT intent가 필요하다는 메시지입니다.  
jda를 초기화하는 부분으로 돌아가서 intent를 추가합니다.  
```java
        JDA jda = JDABuilder.createDefault("token")
                .enableIntents(GatewayIntent.MESSAGE_CONTENT)
                .build();
```  
다시 실행해서 메시지를 보내보겠습니다.  
새로운 에러가 뜨면서 아예 작동 자체가 되지 않습니다. 
![new error](/assets/images/discord_bot/3/3-6.png)  
MESSAGE_CONTENT intent가 없다면서 안 되더니 추가했더니 허용되지 않는다는 에러가 발생합니다.  
이건 discord 개발자 페이지에서 허용해주어야 합니다.  
![allow intent](/assets/images/discord_bot/3/3-7.png)  
Privileged Gateway Intents 항목을 전부 허용해주겠습니다.  
지금 보니 public bot으로 설정되어있었으니 겸사겸사 꺼주도록 합니다.  
그리고 다시 실행합니다.  
정상적으로 실행된 것을 확인하고 메시지를 보내봅니다.  
![complete](/assets/images/discord_bot/3/3-8.png)  
이제 저희가 보낸 메시지가 확인됩니다.  
해당 메시지에 응답을 보내보겠습니다.  
```java
    @Override
    public void onMessageReceived(@NotNull MessageReceivedEvent event) {
        System.out.println("TestListenerAdapter.onMessageReceived");
        if (!event.getAuthor().isBot()) {
            event.getMessage().reply("안녕하세요. 봇이 답변합니다.").queue();
        }
    }
```  
응답에 주의할 것이 하나 있는데 `onMessageReceived`는 모든 메시지에 작동합니다.  
보낸 이가 유저이든 봇이든 어디서 보냈든 모조리 실행되기 때문에 봇이 보낸 메시지에는 반응하지 않도록 처리합니다.  
![reply](/assets/images/discord_bot/3/3-9.png)  
이렇게 답장을 보내는 봇을 만들었습니다.  
<br>
작동하는 것은 기쁘지만, 문제점이 한두 개가 아닌 것을 느끼신 분이 많으실 겁니다.  
그 부분은 차후 제가 어떻게 처리하였는지 말씀드리면서 설명하겠습니다.  
다음엔 버튼 혹은 특정 요소를 선택하는 discord 봇의 핵심인 Interaction에 대해서 써보겠습니다.

### 참고 링크
1. [JDA GitHub](https://github.com/DV8FromTheWorld/JDA)
2. [JDA5 Documentation](https://ci.dv8tion.net/job/JDA5/javadoc/index.html)
3. [제작한 봇 GitHub](https://github.com/Sadowbass/shotgun-bot)