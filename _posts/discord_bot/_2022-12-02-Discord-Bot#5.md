---
title: "Java를 이용한 Discord Bot 만들기 #5 - 새로운 ListenerAdapter 설계"

categories:
- Discord-Bot
tags:
- Java
- JDA
- Discord

last_modified_at: 2022-12-02T15:28:31+09:00

---

봇의 수정에 앞서 왜 스프링을 쓰는지 생각해보겠습니다.  
스프링이 워낙 강력하니 이유는 셀 수 없이 많겠지만 지금 상황에서 스프링이 필요한 이유는 다음과 같다고 생각했습니다.  

1. DispatcherServlet -> HandlerMapping을 통해 핸들러(핸들러 어댑터)를 찾아서 해당하는 핸들러만 동작 가능
2. 구현만 해놓으면 수작업으로 일일이 Component를 등록하지 않아도 자동으로 scan 해서 등록

사실 2번은 개인적인 욕심일 뿐 중요한 건 1번입니다.  
원하는 핸들러만 정확히, JDA 기준으로는 원하는 InteractionEvent만 실행이 되어야 혹시 모를 불상사를 막을 수 있을 거라 생각했습니다.  