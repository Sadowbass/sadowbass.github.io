---
title: "첫번째 프로젝트 : Gamesite 만들어보기"

categories:
- GameSite-Project 
tags:
- SpringBoot
- JPA
- GameSite Project

last_modified_at: 2021-06-16T08:07:00+09:00

---

블로그 생성후 보름정도 지났습니다.  
블로그에 올릴 첫 주제는 무엇을 할까, 무슨 프로젝트를 할까 고민하고 어느정도 작업이 진행되어 슬슬 글을 하나씩 올려보려고 합니다.  
사실 작업만 했으면 며칠이면 끝날 수준의 작업이었는데 회사일도 하고 최근에 백신도 맞고 슬렁슬렁 놀기도하고 게을러서 그렇습니다.\..  
  
그래서 처음 시작할것은 무엇이냐.  
<mark style='background-color: #fff5b1;'>SpringBoot + JPA (Hibernate)</mark>를 이용한 아주아주 간단한 게임사이트를 빙자한 일반 게시판입니다.  
  
최근 김영한님의 JPA 강좌를 보면서 단순히 따라치는게 아니라 내 나름의 것을 만들어 봐야겠다는 생각이 들어서 해보기로 하였습니다.  
  
프로젝트에 Spring Data JPA를 포함시킬것이지만 이는 복잡하고 귀찮은 xml 파일의 생성을 안 하기 위한 방법일뿐  
`JpaRepository`의 기능은 전혀 이용하지 않고 `EntityManager`를 이용해서 직접 persist 시킬것입니다.  
일단 기본적인 JPA 자체의 사용법을 익힌 후 점차 Spring Data Jpa로 확장시키면서 진행합니다.  
같은 맥락으로 `QueryDSL` 역시 사용하지 않습니다 (추후 이로인해 게시판 Filter, Sort에서 끔찍한 상황을 만들게됩니다)  
  
    
여기가 제 일기장 같은 블로그이고 강의할 실력과 말솜씨가 없어서 여기서 jpa의 모든 어노테이션들과 설명을 하지는 않을 것입니다.  
그냥 이렇게 개발하고 글쓰는 사람이 있구나, 이렇게 하면 이런게 얼추 되는구나 정도로만 봐주시고 jpa를 잘 알고싶은 분들은  
김영한님의 교재와 강의, Documentation을 이용하시기 바랍니다.
  
그럼 첫번째 글에서 다시 뵙겠습니다

```java
    public static void main(String[] args) {
        SpringApplication.run(GamesiteApplication.class, args);
    }
```