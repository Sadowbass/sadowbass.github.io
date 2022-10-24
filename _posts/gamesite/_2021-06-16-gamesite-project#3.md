---
title: "첫번째 프로젝트 : EntityManager 사용해보기"

categories:
- GameSite-Project 
tags:
- SpringBoot
- JPA
- GameSite Project

last_modified_at: 2021-06-16T13:53:45+09:00

---

본격적으로 개발을 시작해보겠습니다.  
먼저 저희가 구현해야 할 기능과 어떻게 구현하는것이 좋을지 생각을 먼저 해보겠습니다.  

![.](/assets/images/gamesite/3/1.png)
*~~다시 얘기하자면 QueryDSL이 없어서 게시판 조회 쿼리를 동적으로 만드는 과정이 끔찍합니다~~*  
*게시판부터는 화면을 이용하지 않고 restful api로 작성하도록 수정합니다 jpa와 spring boot가 위주가 되어야 하는데 화면에 너무 많은 시간을 쓰게됩니다*  
*기본적인 jpa 이후로는 spring data jpa와 querydsl까지 바로 도입합니다*  


각각의 엔티티를 먼저 만들어보도록 하겠습니다.  
모든 엔티티는 domain package안에 넣도록 하겠습니다.

* domain
    * User.java
    * UserCharacter.java
    * Post.java
    * Comment.java

먼저 유저엔티티를 작성하겠습니다.  
요구 사항에 나와있는 엔티티의 구성 요소는 총 4가지 입니다.  
만들어 보겠습니다.

```java
@Getter
public class User {

    private Long id;
    private String userAccount;
    private String userName;
    private String password;
    private String profile;
}
```

롬복의 Getter를 이용하여 모든 필드의 Getter를 자동으로 만들어줍니다.  
Setter는 엔티티에서 쓰지 않습니다.  
이곳저곳에서 Setter를 불러서 값들을 바꾸다보면 후에 문제가 생길 가능성이 많음으로 객체를 생성할때는 생성자, Builder, static factory 패턴등을 이용합니다.  
좋은 예시들은 이펙티브자바 책에 잘 나와 있으니 꼭 읽어보세요.  

보시면 Long 타입의 id가 뭔가 싶으실텐데 이건 JPA와 DB에서 사용되는 PK값이며 여기서는 단순히 1씩 올라가는 Long타입 값을 이용하겠습니다.  
위의 구조는 흔히 볼수있는 Java Bean처럼 생긴 객체입니다(Setter가 없음으로 정확히는 Java Bean도 아닙니다). JPA의 Entity는 아니니 저것을 JPA에서 이용할수있는 Entity로 만들겠습니다.

```java
@Entity
@Getter
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "user_account")
    private String userAccount;

    @Column(name = "user_name")
    private String userName;

    @Column(name = "password")
    private String password;

    @Lob
    private String profile;

    protected User() {

    }

    public User(String userAccount, String userName, String password) {
        this.userAccount = userAccount;
        this.userName = userName;
        this.password = password;
    }
}
```
*@Entity Annotation과 필드마다 각종 신기한 어노테이션들이 추가되었다*

하나씩 설명해보겠습니다.
1. @Entity: JPA가 관리하는 엔티티임을 알려줍니다. 해당 어노테이션이 없으면 JPA에서 persist(영속화)가 불가능합니다
2. @Id: PK값입니다. 복합키등의 PK도 이용할 수 있지만 가장 간단하게 사용하겠습니다.
3. @GeneratedValue: 자동으로 증가되는 Oracle, PostgreSQL의 시퀀스, MySQL 계열의 Auto Increment라고 보시면 됩니다. 옵션을 통해 생성 방식을 정해줄수도 있습니다.
4. @Column: 컬럼명, 길이, 제약조건등을 설정할 수 있습니다. 여기서는 언더스코어 방식으로 컬럼명이 맵핑되도록 name만 설정하겠습니다. (hibernate는 필드명을 그대로 컬럼명이랑 맵핑시키는데 Spring Data JPA는 카멜케이스를 자동으로 언더스코어로 변환해줍니다. 저희는 Spring Data JPA를 의존성에 추가해서 안 해줘도 되지만 최대한 Data JPA를 안 쓰도록 하기 위해 명시합니다)
5. @Lob: varchar를 넘는 큰것들을 넣을때 쓰는 BLob, CLob을 설정하기 위해 사용합니다. 문자열 계열의 타입은 CLob으로 그외엔 BLob으로 맵핑됩니다.

자 이제 저희의 첫 엔티티가 생성되었으니 이것을 어떻게 영속화하고 JPA가 관리를 하게 해주는지 테스트 코드를 통해 알아보겠습니다.
엔티티 자체에 Test Class를 만들기는 좀 뭐하고 기본적인 JPA EntityManager 테스트라는 의미로 루트 패키지에 EntityManagerTest 클래스를 작성합니다.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
@Transactional
public class EntityManagerTest {

    @Autowired
    private EntityManager em;

    @Test
    void save() {

    }

    @Test
    void find() {

    }

    @Test
    void findAll() {

    }

    @Test
    void update() {

    }

    @Test
    void remove() {

    }
}
```

스프링 의존성 없이 자바 자체의 단위테스트가 가장 좋은 테스트라고 하지만.\..  
엔티티매니저와 자동으로 주입시켜주는 스프링의 기능이 지금은 더 중요할듯하니 @ExtendWith와 @SpringBootTest 어노테이션을 붙여줍니다.  
저는 JUnit5를 사용하기때문에 어노테이션이 저런데 JUnit4는 @RunWith를 사용해야되는것으로 알고 있습니다.  
  
# @Transactional
1. JPA는 트랜잭션 단위가 매우 중요하고 select를 제외한 insert, update, delete는 전부 트랜잭션 단위 안에 들어있어야 합니다.
2. 원래는 em.getTransaction()을 이용해서 트랜잭션을 가져오고 begin()시키고 마지막에 commit 혹은 rollback 해야하지만 이 어노테이션으로 끝내겠습니다.
3. 테스트 Class, Method는 무조건 롤백됩니다. 테스트시에도 롤백이 안 되었으면 한다면 @Rollback(false) 어노테이션을 추가하면 됩니다.

기본준비는 되었으니 `save()` 메소드부터 작성하겠습니다
```java
@Test
    void save() {
        //원래는 아래처럼 해야한다
        //EntityTransaction tx = em.getTransaction();
        //tx.begin();
        //do something...
        //tx.commit();

        //given
        String userAccount = "userAccount";
        String userName = "Sadowbass";
        String password = "password";

        User user = new User(userAccount, userName, password);
        em.persist(user);

        //when
        User findUser = em.find(User.class, user.getId());

        //then
        assertThat(findUser).isEqualTo(user);
        assertThat(findUser.getUserName()).isEqualTo(userName);
    }
```
*assertJ를 이용한 Test입니다*

보시면 궁금증이 몇가지 생길것입니다. 하나씩 보겠습니다.  
1. em.persist만 했는데 저게 DB에 저장이 되나요?
    * 예 DB에 저장하는 시점은 flush 시점이지만 jpa가 자동으로 저장을 해줍니다.
2. Entity에 PK(id)를 설정하지 않았는데 어떻게 저장이 되는건가요?
    * @GeneratedValue에 의해서 영속화 시점에 자동으로 생성하게 됩니다.
3. equals와 hashcode를 오버라이딩 하지 않았는데 저희가 만든 객체와 em.find()로 가져온 각각의 객체가 어떻게 equalTo를 통과 할수가 있나요?
    * jpa는(정확히는 영속성 컨텍스트) 영속화한 대상의 동일성을 보장해 줍니다.

잠시 Rollback을 막고 실제로 DB에 정상적으로 insert 되었는지 확인해보겠습니다.
![before save](/assets/images/gamesite/3/2.png)  
*아무런것도 없던 상태에서 save() 테스트 메소드를 실행하면!*

![after save](/assets/images/gamesite/3/3.png)  
*이렇게 DB에 값이 생성되는것을 확인할 수 있습니다*  

자세한 내용은 JPA의 영속성 컨텍스트 (persistence context)에 대하여 학습하시면 됩니다.  





