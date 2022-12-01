---
title: "Java를 이용한 Discord Bot 만들기 #1 - 개발을 하게된 이유"

categories:
- Discord-Bot
tags:
- Java
- JDA
- Discord

last_modified_at: 2022-12-01T19:28:42+09:00

---

길드 관리 어플리케이션에 관한 글을 올리던 중 새로운 시리즈의 글을 올리게 될 줄은 몰랐습니다.  
<br>
대학 기말고사 준비와 개인적으로 따로 공부를 하면서 관리 애플리케이션 글을 시간을 내서 길게 쓰기가 힘든 상황입니다.  
그렇지만 길드관리 애플리케이션은 제가 가장 시간도 많이 쓰고 머리도 많이 쓴 프로젝트이기에 꼭 여유가 생기면 계속해서 작성할 예정입니다.  
<br>
디스코드 봇을 만들게 된 계기는 단순한 이유였습니다.  
<br>
대다수의 게임 길드들이 그렇겠지만  
카톡, 디스코드, 인게임 커뮤니티 시스템 등 길드원들이 이용하는 여러가지 커뮤니케이션 도구들이 굉장히 나누어진 상황이었고  
이로인해 여러가지 자동화된 공지나 건의접수등이 불가능하였습니다.  
<br>
1:1 메신저를 통한 건의사항등은 모든운영진에게 공유되지 않고 누락되기도 쉬웠으며  
파티신청은 카톡으로, 건의는 카톡과 디코 모두 이런식으로 나누어져 이용자들도 혼란스러웠고  
무엇보다 그것들을 접수,관리하는 운영진 개개인에 대한 피로감도 많은 상황이었기에 필수적으로 통일된 환경이 필요하였습니다.
<br>
이러한 이유로 여러가지 길드원들과 운영진들을 위한 자동화된 창구를 만들어야했고 두가지의 선택지중에 선택을 해야했습니다.
1. 카톡을 이용한 자동화
2. 디스코드를 이용한 자동화

처음에 가장 먼저 고려하였던것은 카톡을 이용한 봇이었습니다.  
누구나 쓰고있고 가장 접근성이 편리하기에 카카오톡 봇을 고려하였지만 아래와 같은 이유로 개발이 어려웠습니다.  
1. 단순한 방장봇등이 아닌 직접 원하는 기능을 개발해서 사용하는 봇은 카카오톡에서 공식적으로 지원하지 않는다.
2. 안드로이드 에뮬레이터등으로 카톡을 설치하고 개발한 봇 apk를 실행해서 동작하기에 별도의 카카오톡 계정이 필요하다.
3. js기반이 대다수라 내가 당장 개발하기엔 퀄리티의 문제가 우려된다.

상기 사유중 가장 문제는 역시 별도의 계정이 필요한것인데 아쉽게도 저는 현재 단일 계정만 보유중으로 카톡봇은 제외하기로 하였습니다.  
그렇다면 남은것은 디스코드 봇을 이용하는 방법뿐이었고 이미 디스코드엔 수많은 봇들이 여러 서버에 들어와 있기에 분명히 좋은 방법이 있을것이라고 생각했습니다.  
<br>
역시 기대한대로 디스코드는 봇을 위한 공식 API와 SDK를 제공하고 있었고 그중 java기반의 JDA가 있어 JDA를 이용하여 봇을 만들어 보기로 합니다.
<br>
다만 수많은 봇이 디스코드에 분포되어 있지만 국내 사이트중 예제라고 할만한것이 아주 단순한 봇과 디스코드 서버의 연결, 메세지 응답등의 단순 기능들 뿐이라 공식 api docs와 외국 사이트들의 도움을 받아야만 함이 몹시 괴로운 과정이었습니다.  
혹시라도 개발 공부를 처음 시작하신 분들은 시간을 조금이라도 쪼개서 영어를 반드시 공부하세요...  

## 사설은 여기까지하고 바로 개발 시작!

### 참고 링크
1. [JDA GitHub](https://github.com/DV8FromTheWorld/JDA)
2. [제가 만든 봇 GitHub](https://github.com/Sadowbass/shotgun-bot)