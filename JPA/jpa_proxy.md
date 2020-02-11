
# 프록시

> 엔티티와 연관관계가 설정된 모든 엔티티를 사용유무와는 관계없이 모든 엔티티를 한번에 Join되어 가지고 오게 된다.  
> 비즈니스 로직마다 다르겠지만 빈번하게 사용하지 않는 연관관계일 경우에도 Join되어 가지고 오게 된다.  
> 1~2개의 작은 모델링 관계에서는 성능적으로 이슈가 없겠지만 현업에서의 수많은 테이블과 관계를 맺고 있는 상황에서는 성능적 이슈가 크다.  
> 프록시는 위의 이슈를 해결하기 위한 개념이다. 실제 엔티티를 조회할때 바로 데이터베이스를 호출하는게 아닌 가짜 엔티티를 반환해 준다.  
> 가짜 엔티티의 경우에는 실제 get을 할때 DB에 쿼리가 수행되어 진다.  
> 여기에 사용하는 것이 **지연로딩(LAZY)**이다.  

### 간단한 예제를 통해 쿼리의 상태를 확인해보자.

**Member Domain**

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

**Team Doemain**

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

```java
Member findMember = em.find(Member.class, m1.getId());

/*
 Hibernate: 
    select
        member0_.id as id1_3_0_,
        member0_.name as name2_3_0_,
        member0_.team_id as team_id3_3_0_,
        team1_.id as id1_5_1_,
        team1_.name as name2_5_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.team_id=team1_.id 
    where
        member0_.id=?
*/
```

Member와 Team은 다:1 관계이다. 여기서 Member를 조회할 경우 Team의 엔티티도 Join을 통해 같이 조회가 된다.  
비즈니스 로직에 따라 Member를 조회 후 Team도 무조건 적으로 사용되지 않는다면 성능상 한방쿼리로 데이터를 가지고 오는 것은 효율적이지 못하다.  

```java
System.out.println("======== Proxy ========");
Member findMember = em.getReference(Member.class, m1.getId());
System.out.println("======== Proxy ========");


System.out.println("=================");
System.out.println(findMember);
System.out.println("=================");

/*
======== Proxy ========
======== Proxy ========

=================
Hibernate: 
    select
        member0_.id as id1_3_0_,
        member0_.name as name2_3_0_,
        member0_.team_id as team_id3_3_0_,
        team1_.id as id1_5_1_,
        team1_.name as name2_5_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.team_id=team1_.id 
    where
        member0_.id=?
Member{id=2, name='m1', team=hellojpa.domain.customer.Team@4d6f623d}
=================
*/
```

위의 예제는 Proxy이다. 아래는 Proxy를 활용하여 변경 했을때의 쿼리가 발생하는 시점을 확인 할 수 있다.  
**getReference**를 통해 proxy객체를 반환 받을 수 있다.  

주석의 내용을 보면 proxy로 가지고오는 구간에는 DB를 찌르지 않는다. 반환된 가짜 엔티티인 Proxy의 데이터를 get할때 proxy가 초기화가 되며 DB에 쿼리가 날라간다.  
조금더 자세하게 말을 하자면, Proxy는 실제 엔티티를 상속받은 객체이다. 
실제 엔티티에 접근을 할 수 있는 가짜 엔티티가 proxy이다. 또한 proxy는 최초 영속성 컨텍스트에 값이 있는지를 요청하며, 영속성 컨텍스트에 값이 없을 때 DB에 쿼리를 수행하여 실제 엔티티를 참조하게 된다.  

## Proxy의 특징

1. proxy객체는  처음 사용할 때 한 번만 초기화 한다.
2. proxy가 초기화가 되었다고 해서 실제 엔티티로 바뀌는 것은 아니다. 단지 접근을 할 수 있는 것이다. 
3. proxy는 실제 엔티티를 상속 받은 것 이기 때문에 타입 체크 시 mismatch가 되기에 조심해야 한다.  
4. 영속성 컨텍스트에 찾고자 하는 엔티티가 이미 캐시가 되어있다면, getReference를 하더라도 proxy객체를 반환하지 않고 실제 엔티티를 반환한다.  

## 즉시로딩과 지연로딩

> 위의 proxy의 개념을 숙지하고 **즉시로딩** 과 **지연로딩**을 진행해보자.  

### 즉시로딩

엔티티를 조회할 때 연관된 엔티티도 함께 조회한다.  
설정 방법은 @ManyToOne(fetch = FetchType.EAGER)를 통해 사용 가능하다.  

**Member Domain**

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "team_id")
    private Team team;
}

System.out.println("======== EAGER ========");
Member findMember = em.find(Member.class, m1.getId());
System.out.println("======== EAGER ========");

System.out.println("=================");
System.out.println(findMember);
System.out.println("=================");

/*
======== EAGER ========
Hibernate: 
    select
        member0_.id as id1_3_0_,
        member0_.name as name2_3_0_,
        member0_.team_id as team_id3_3_0_,
        team1_.id as id1_5_1_,
        team1_.name as name2_5_1_ 
    from
        Member member0_ 
    left outer join
        Team team1_ 
            on member0_.team_id=team1_.id 
    where
        member0_.id=?
======== EAGER ========


=================
Member{id=2, name='m1', team=hellojpa.domain.customer.Team@62515a47}
=================
*/
```

### 지연로딩

엔티티를 조회할 때 쿼리가 수행되지 않고 연관된 엔티티를 실제 사용할 때 조회한다.  
설정 방법은 @ManyToOne(fetch = FetchType.LAZY)를 통해 사용 가능하다.

**Member Domain**

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
}

System.out.println("======== LAZY ========");
System.out.println("======== LAZY ========");
Member findMember = em.find(Member.class, m1.getId());

System.out.println("=================");
System.out.println(findMember);
System.out.println("=================");

/*
======== LAZY ========
Hibernate: 
    select
        member0_.id as id1_3_0_,
        member0_.name as name2_3_0_,
        member0_.team_id as team_id3_3_0_ 
    from
        Member member0_ 
    where
        member0_.id=?
======== LAZY ========

=================
Hibernate: 
    select
        team0_.id as id1_5_0_,
        team0_.name as name2_5_0_ 
    from
        Team team0_ 
    where
        team0_.id=?
Member{id=2, name='m1', team=hellojpa.domain.customer.Team@e93f3d5}
=================
*/
```

## CASCAME

특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶으면 영속성 전이 기능을 사용하면 된다.  
위의 Team과 Member의 경우로 볼때 Team과 Member 각각 모드 영속성을 시켜야 DB에 저장이 되게 된다.   

쉽게 말하면 다음과 같다. 부모 엔티티를 저장할 때 자식 엔티티도 같이 저장을 하는 것이다.  


**Team Domain**

```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", cascade = CascadeType.ALL)
    private List<Member> members = new ArrayList<>();
}
```

```java
Member m1 = new Member();
m1.setName("m1");
Member m2 = new Member();
m2.setName("m2");

Team t1 = new Team();
t1.setName("t1");

m1.setTeam(t1);
m2.setTeam(t1);

t1.getMembers().add(m1);
t1.getMembers().add(m2);

em.persist(t1);

/**
Hibernate: 
    insert 
        into
            Team
            (name, id) 
        values
            (?, ?)
Hibernate: 
    insert 
        into
            Member
            (name, team_id, id) 
        values
            (?, ?, ?)
Hibernate: 
    insert 
        into
            Member
            (name, team_id, id) 
        values
            (?, ?, ?)
**/
```

이 외에 고아객체 또한 ```orphanRemoval = true``` 를 통해 설정할 수 있다.  
고아객체는 부모 엔티티에서 자식 엔티티를 삭제할 수 있는 기능이다.  

> 참고로..  
> 영속성전이 + 고아객체 는 부모 엔티티에서 자식 엔티티의 생명주기를 관리한다.  
> 이 것은 실제 참조되고 있는게 부모 엔티티만 있을때 사용을 해야하며, 실제로는 이런 기능이 있다 정도만 참고만 하자.  