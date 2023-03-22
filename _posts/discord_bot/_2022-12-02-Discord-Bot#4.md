---
title: "Java를 이용한 Discord Bot 만들기 #4 - Interaction"

categories:
- Discord-Bot
tags:
- Java
- JDA
- Discord

last_modified_at: 2022-12-02T13:36:32+09:00

---

지난 포스트에서는 가장 기본적인 EventListener를 만들어보았습니다.  
눈에 보이는 문제점은 뒤로 미루고 응답을 버튼이나 모달 등으로 받아볼 수 있도록 기능을 더 입혀보겠습니다.  
메시지를 채팅방에 보내는 방법은 대표적으로 2가지가 있습니다.  

1. 들어온 메시지에 답변으로 메시지 보내기.
2. 들어온 메시지와 상관없이 원하는 채널에 메시지 보내기.  

먼저 답변으로 메시지를 보내는 방법은 지난 편에서 작성했던 것처럼 `Message.reply()`를 통해서 하게 됩니다.  
단순한 문자열은 String으로 응답할 수 있지만 버튼이나 각종 셀렉트, 모달은 MessageCreateData라는 객체를 생성해서 보내주어야 합니다.  
![reply() overloading methods](/assets/images/discord_bot/4/4-1.png)  
2가지 parameter를 지원하도록 overload 되어 있습니다.  
CharSequence는 지난번에 String을 사용해보았으니 이번엔 버튼을 넣은 응답을 해보겠습니다.  
```java
    @Override
    public void onMessageReceived(@NotNull MessageReceivedEvent event) {
        Message message = event.getMessage();
        String contentRaw = message.getContentRaw();
        if (!event.getAuthor().isBot()) {
            if (contentRaw.equalsIgnoreCase("!vote")) {
                MessageCreateData buttonReplyMessage = new MessageCreateBuilder()
                        .addContent("여러분의 직업을 선택해주세요")
                        .addActionRow(
                                Button.primary("devil_hunter_female", "건슬링어"),
                                Button.primary("force_master", "기공사")
                        ).build();
                message.reply(buttonReplyMessage).queue();
            }
        }
    }
```  
(잠시 게임 얘기를 하자면 실제로 로스트아크 공식 홈페이지에는 건슬링어가 영어로 별도의 캐릭이 아닌 데빌헌터 여캐라는 이름을 쓰고 있다...)  
지난번처럼 봇의 메시지인지 확인하고, 유저가 보낸 메시지가 !vote라면 우리가 만든 메시지를 답변하도록 만들어 보았습니다.  
<br>
문서를 보면서 한 줄씩 확인하겠습니다. [MessageCreateBuilder docs](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/utils/messages/MessageCreateBuilder.html)  
`.addContent()`는 메시지의 텍스트 부분입니다.  
`.addActionRow()`는 MessageCreateBuilder 의 상위클래스인 MessageCreateRequest 에 구현되어 있습니다. [MessageCreateRequest](https://ci.dv8tion.net/job/JDA5/javadoc/net/dv8tion/jda/api/utils/messages/MessageCreateRequest.html#addActionRow(java.util.Collection))  
ItemComponent 객체들을 하나의 row로 보여줄 수 있습니다.  
만약에 두 개의 버튼을 각각 `.addActionRow()`에 넣을 경우 각각 한줄씩 2줄로 버튼이 나오게 됩니다.  
보기가 좋지 않으니 하나에 밀어 넣겠습니다. Collection 객체로도 넣을수 있습니다.  
<br>
제대로 버튼이 붙어서 답변이 가는지 확인해보겠습니다.  
![button reply](/assets/images/discord_bot/4/4-2.png)  
다른 메시지에는 반응하지 않고 !vote에만 정확히 답변을 해주었습니다.  
원하는 대로 버튼도 달린 보기 좋은 답장입니다.  
이제 버튼을 눌러볼까요?  
![button reply](/assets/images/discord_bot/4/4-3.png)  
상호작용 실패라는 메시지가 출력됩니다.  
무엇이 문제일까요?  
그렇습니다. 저희는 메시지에 반응하는 메서드는 만들었지만, 버튼에 반응하는 메서드는 구현하지 않았습니다.  
바로 구현해보겠습니다.  
```java
    @Override
    public void onButtonInteraction(@NotNull ButtonInteractionEvent event) {
        System.out.println("event.getMessage() = " + event.getMessage());
        System.out.println("event.getButton() = " + event.getButton());
    }
```  
먼저 ButtonInteractionEvent가 무엇인지부터 보기 위해 출력을 해봅니다.  
고맙게도 JDA는 상당수 클래스가 toString이 오버라이딩 되어있어 이것만으로 어느 정도 정보를 확인 할 수 있습니다.  
![button result](/assets/images/discord_bot/4/4-4.png)  
`getButton`만 보면 위에서 영어로 작성한 id과 label이 보입니다.  
버튼에 반응을 해보겠습니다.  
InteractionEvent들은 공통으로 reply를 하지 않으면 내부에서 무슨 처리를 하든 상호작용 실패라는 메시지를 받아보게 됩니다.  
간단한 응답이라도 전달하겠습니다.  
```java
    @Override
    public void onButtonInteraction(@NotNull ButtonInteractionEvent event) {
        String buttonId = event.getButton().getId();
        if (buttonId.equals("devil_hunter_female")) {
            event.reply("건슬링어를 선택하셨습니다").queue();
        } else {
            event.reply("기공사를 선택하셨습니다").queue();
        }
    }
```  
![button interaction reply](/assets/images/discord_bot/4/4-5.png)  
이렇게 버튼 인터렉션까지 만들어 보았습니다.  
`onMessageReceived` 때도 느꼈지만 이번에도 문제가 있습니다.

1. 해당하는 모든 이벤트에 조건을 사용자가 직접 걸어야 한다.
2. 여러 개의 ListenerAdapter를 구현하였다면 해당하는 이벤트를 구현한 모든 메서드가 전부 실행됩니다.
3. id가 전부 String으로 이루어져 있으며 unique 하지 않습니다. 2번 사유에 맞물려서 혹시라도 각기 다른 버튼 이벤트에 같은 id를 넣으면 전혀 엉뚱한 버튼 인터렉션이 같이 실행됩니다.  

1번은 이해할 수 있는 부분이라지만 2번과 3번은 조금 치명적입니다.  
그나마 3번은 모든 인터렉션에 패턴화를 시켜서 조심해서 작성한다면 괜찮습니다만 역시 2번...  
모든 리스너가 동작하는 자체가 마음에 들지 않습니다.  
<br>
 ![get problem](/assets/images/discord_bot/4/4-6.png)  
(한 번의 메시지에 `onMessageReceied`를 구현한 2번의 클래스가 동작하는 모습.)  
위험하고 번거롭습니다.  
그런 상황에 딱 떠오른 것은 역시 Java의 구원자 Spring...  
근데 스프링을 쓰지 않고 스프링처럼 한번 만들어보기로 합니다.  
(이것이 고난의 시작...)
### 다음으로 이어집니다

### 참고 링크
1. [JDA GitHub](https://github.com/DV8FromTheWorld/JDA)
2. [JDA5 Documentation](https://ci.dv8tion.net/job/JDA5/javadoc/index.html)
3. [제작한 봇 GitHub](https://github.com/Sadowbass/shotgun-bot)