---
title: "첫번째 프로젝트 : Entity Manager를 이용한 Update와 Delete 그리고 간단한 JPQL 사용"

categories:
- GameSite-Project
tags:
- SpringBoot
- JPA
- GameSite Project

last_modified_at: 2021-06-16T16:55:10+09:00

---

이번에는 전에 만들었던 나머지 테스트 코드들을 작성하겠습니다.  
전 사실 테스트 코드를 그리 열심히 작성하지 않는데 이번 기회에 TC를 작성하는 습관을 좀 들여야겠네요ㅎㅎ.  
Given/When/Then을 최대한 이용해보겠습니다. TDD에 대한 자세한 내용은 마틴 파울러의 책에 잘 나와있다고 합니다.  
저도 기회되면 사서 읽어보겠습니다.  
바로 코드 작성을 해보겠습니다. 

# find()
```java
    @Test
    void find() {
        //given
        String userAccount = "userAccount";
        String userName = "Sadowbass";
        String password = "password";

        User user1 = new User(userAccount + 1, userName + 1, password);
        User user2 = new User(userAccount + 2, userName + 2, password);

        em.persist(user1);
        em.persist(user2);

        //flush를 수동으로 하면 transaction 마지막 commit시점에 전부 반영되는것이 아니라 flush 시점에 DB에 반영할수있다.
        //조회 쿼리를 확인하기 위해 명시적으로 flush를 사용한다
        System.out.println("=========");
        System.out.println("flush");
        em.flush();
        System.out.println("clear");
        em.clear();
        System.out.println("=========");

        //when
        User findUser = em.find(User.class, user1.getId());

        //then
        System.out.println("findUser = " + findUser);
        assertThat(findUser.getUserName()).isNotEqualTo(user2.getUserName());
        //영속성 컨텍스트를 clear로 비워버려서 새로 가져온 user는 동일성을 보장하지 않습니다.
        assertThat(findUser).isNotEqualTo(user1);
    }
```  
영속성 컨텍스트로 관리되는 엔티티는 DB에서 가져오지 않고 관리중인 객체를 리턴해주기때문에 강제로 flush & clear 처리하여 DB에서 가져오도록 하였습니다.  
em.find() 메소드의 첫번째 파라미터는 엔티티의 타입 클래스를, 2번째는 pk를 넣어주면 단건조회가 가능합니다.  
  
로그를 확인해 보겠습니다

```text
=========
flush
2021-06-16 17:29:17.778 DEBUG 82388 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        user
        (password, profile, user_account, user_name, id) 
    values
        (?, ?, ?, ?, ?)
2021-06-16 17:29:17.779 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [password]
2021-06-16 17:29:17.779 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [CLOB] - [null]
2021-06-16 17:29:17.779 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [userAccount1]
2021-06-16 17:29:17.779 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARCHAR] - [Sadowbass1]
2021-06-16 17:29:17.780 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [5] as [BIGINT] - [1]
2021-06-16 17:29:17.781 DEBUG 82388 --- [           main] org.hibernate.SQL                        : 
    insert 
    into
        user
        (password, profile, user_account, user_name, id) 
    values
        (?, ?, ?, ?, ?)
2021-06-16 17:29:17.781 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [password]
2021-06-16 17:29:17.781 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [CLOB] - [null]
2021-06-16 17:29:17.781 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [userAccount2]
2021-06-16 17:29:17.781 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARCHAR] - [Sadowbass2]
2021-06-16 17:29:17.781 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [5] as [BIGINT] - [2]
clear
=========
2021-06-16 17:29:17.785 DEBUG 82388 --- [           main] org.hibernate.SQL                        : 
    select
        user0_.id as id1_0_0_,
        user0_.password as password2_0_0_,
        user0_.profile as profile3_0_0_,
        user0_.user_account as user_acc4_0_0_,
        user0_.user_name as user_nam5_0_0_ 
    from
        user user0_ 
    where
        user0_.id=?
2021-06-16 17:29:17.786 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
2021-06-16 17:29:17.788 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([password2_0_0_] : [VARCHAR]) - [password]
2021-06-16 17:29:17.789 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([profile3_0_0_] : [CLOB]) - [null]
2021-06-16 17:29:17.789 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([user_acc4_0_0_] : [VARCHAR]) - [userAccount1]
2021-06-16 17:29:17.789 TRACE 82388 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([user_nam5_0_0_] : [VARCHAR]) - [Sadowbass1]
findUser = User{id=1, userAccount='userAccount1', userName='Sadowbass1', password='password', profile='null'}
```
*flush 시점에 영속화 시킨 2개의 User가 insert되었다. 그 이후 새롭게 select를 해온다*  

# findAll()
기본적으로 `EntityManager`는 기본 메서드로 다건 조회가 불가능합니다.  
그래서 이용하는데 JPQL(Java Persistence Query Language) 이라는 SQL과 유사한 문법을 이용해서 조회를 하게되는데 JPQL을 결국 JPA가 SQL로 바꿔주기때문에 SQL을 알고있어야 합니다.  
```java
    @Test
    void findAll() {
        //given
        String userAccount = "userAccount";
        String userName = "Sadowbass";
        String password = "password";

        User user1 = new User(userAccount + 1, userName + 1, password);
        User user2 = new User(userAccount + 2, userName + 2, password);

        em.persist(user1);
        em.persist(user2);

        System.out.println("=========");
        System.out.println("flush");
        em.flush();
        System.out.println("clear");
        em.clear();
        System.out.println("=========");

        //when
        String query = "select u from User u";
        List<User> resultList = em.createQuery(query, User.class).getResultList();
        //then

        for (User user : resultList) {
            System.out.println("user = " + user);
        }

        assertThat(resultList.get(0).getUserAccount()).isEqualTo(user1.getUserAccount());
        assertThat(resultList.get(1).getUserAccount()).isEqualTo(user2.getUserAccount());
    }
```  
when절의 query 문자열을 보겠습니다.  
흔히 모든것을 조회해올때 사용하는 SQL 쿼리와 비슷한데 뭔가 애매하게 다르죠?  
JPA는 도메인, 객체단위로 처리를 하기때문에 저렇게 생겼습니다.  
`select u` 부분은 `from User u` 에서 명시한 u를 뜻하며 jpql에서 alias는 필수적으로 기재하여야 합니다 (as는 명시를 하여도 하지 않아도 됩니다. 여기에서는 생략.)  
  
jpql에서 result를 받는방법은 크게 2가지가 있는데요.
1. getResultList()
2. getSingleResult()

getResultList()는 List 타입으로 리턴이 되고 값이 없으면 size 0짜리가 리턴이 되는반면  
getSingleResult는 값이 없던가 2개 이상이면 Exception이 납니다.\..  
무조건 하나만 나오니까 난 SingleResult 써야지~ 했다가 생각치도 못한 Exception이 발생할 수 있으니 주의 하셔야 합니다.  

# update()
개인적으로 JPA CRUD에서 가장 골치 아픈부분이라고 생각합니다.
update하는 방식이 2가지
1. Dirty Checking을 이용한 방법
2. merge를 이용하여 병합하는 방법
이렇게 두가지가 있기 떄문인데 가능하면 영속성 컨텍스트가 관리하는 Dirty Checking을 이용해서 수정하는것이 좋습니다. (merge는 추후 작성해보겠습니다)  
  
```java
    @Test
    void update() {
        //given
        String userAccount = "userAccount";
        String userName = "Sadowbass";
        String password = "password";

        User user1 = new User(userAccount + 1, userName + 1, password);

        em.persist(user1);

        System.out.println("=========");
        System.out.println("flush");
        em.flush();
        System.out.println("clear");
        em.clear();
        System.out.println("=========");

        //when
        User findUser = em.find(User.class, user1.getId());
        System.out.println("=========");
        System.out.println("setUserName");
        findUser.setUserName("newName");
        System.out.println("=========");
        em.flush();
        em.clear();

        User updatedUser = em.find(User.class, user1.getId());
        //then
        assertThat(updatedUser.getUserName()).isEqualTo("newName");
    }
```  
*영속 컨텍스트를 비우고 업데이트를 실행하고 가져오기 위해 의도적으로 flush를 많이 사용한것이고 실제로 저렇게 많은 플러시를 쓸일은 적습니다.*  
  
단순한 예제를 위해 일단 User 엔티티에 Setter를 넣어주고 진행하겠습니다.  
먼저 똑같이 User 엔티티에 값을 넣고 DB에 flush 후 find로 영속성 컨텍스트에 User 엔티티를 넣어줍니다.  
그리고 안의 값을 수정하면 끝입니다.  
이렇게 하면 JPA가 저장하고 있던 스냅샷과 현재 변경된 객체의 차이를 찾아서 update query를 자기가 날려줍니다.  
생성된 SQL을 확인해보겠습니다.
```text
=========
2021-06-16 19:42:41.387 DEBUG 88776 --- [           main] org.hibernate.SQL                        : 
    select
        user0_.id as id1_0_0_,
        user0_.password as password2_0_0_,
        user0_.profile as profile3_0_0_,
        user0_.user_account as user_acc4_0_0_,
        user0_.user_name as user_nam5_0_0_ 
    from
        user user0_ 
    where
        user0_.id=?
2021-06-16 19:42:41.388 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [BIGINT] - [1]
2021-06-16 19:42:41.390 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([password2_0_0_] : [VARCHAR]) - [password]
2021-06-16 19:42:41.391 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([profile3_0_0_] : [CLOB]) - [null]
2021-06-16 19:42:41.391 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([user_acc4_0_0_] : [VARCHAR]) - [userAccount1]
2021-06-16 19:42:41.391 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicExtractor   : extracted value ([user_nam5_0_0_] : [VARCHAR]) - [Sadowbass1]
=========
setUserName
=========
2021-06-16 19:42:41.393 DEBUG 88776 --- [           main] org.hibernate.SQL                        : 
    update
        user 
    set
        password=?,
        profile=?,
        user_account=?,
        user_name=? 
    where
        id=?
2021-06-16 19:42:41.393 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [1] as [VARCHAR] - [password]
2021-06-16 19:42:41.393 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [2] as [CLOB] - [null]
2021-06-16 19:42:41.393 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [3] as [VARCHAR] - [userAccount1]
2021-06-16 19:42:41.393 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [4] as [VARCHAR] - [newName]
2021-06-16 19:42:41.393 TRACE 88776 --- [           main] o.h.type.descriptor.sql.BasicBinder      : binding parameter [5] as [BIGINT] - [1]
```  
setUserName 이후 update query가 동작하는것을 확인할 수 있습니다.  
jpa가 영속화 시킬 당시의 값과 현재의 값이 달라졌을경우 자동으로 변경을 확인해서 update 시킨다고 보시면 되겠습니다.  

# delete()
save()랑 거의 똑같습니다.  
바로 코드를 보면
```java
    @Test
    void delete() {
        //given
        String userAccount = "userAccount";
        String userName = "Sadowbass";
        String password = "password";

        User user1 = new User(userAccount + 1, userName + 1, password);

        em.persist(user1);

        System.out.println("=========");
        System.out.println("flush");
        em.flush();
        System.out.println("clear");
        em.clear();
        System.out.println("=========");
        //when
        User findUser = em.find(User.class, user1.getId());
        //then
        em.remove(findUser);
        em.flush();
        em.clear();

        User deletedUser = em.find(User.class, user1.getId());
        assertThat(deletedUser).isNull();
    }
```  
EntityManager의 remove()를 이용해서 지워줍니다.  
지울것은 pk를 넣는것이 아닌 지울 엔티티 자체를 넣으셔야 합니다.  

이렇게 간단한 EntityManager를 이용한 CRUD를 마치고 다음부터 모든 엔티티를 작성해보겠습니다.

