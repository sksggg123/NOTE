
# 다양한 연관관계 매핑

## 다대일 [N : 1]
다대일은 가장 많이 사용하는 연관관계이다. 양방향 적용을 할때는 외래키가 있는 쪽이 연관관계 주인으로 설정하는게 깔끔하다. 아래는 Member와 Team을 예제로 된 다대일 관계이다.  


### Member Entity (단방향)
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    private String name;
}
```

### Team Entity (단방향)
```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;
}
```

### 사용 (단방향)
```java
Team team = new Team();
team.setName("team1");
em.persist(team);

Member member = new Member();
member.setName("kwon");
member.setTeam(team);
em.persist(member);
```

### Member Entity (양방향)
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    private String name;
}
```

### Team Entity (양방향)
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

### 사용 (양방향)
```java
Team team = new Team();
team.setName("team1");
em.persist(team);

Member member = new Member();
member.setName("kwon");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

Team findTeam = em.find(Team.class, team.getId());
System.out.println(findTeam.getMembers());
```

## 일대다 [1 : N]
일대다 단방향의 경우는 "1"에 해당하는 필드가 연관관계의 주인이다. 테이블의 일대다 관계는 항상 "N" 쪽에 외래 키가 있다.  
또한 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 구조. 위의 Member와 Team의 관계를 보면 Team은 '1' Member는 'N'이다. Team의 입장에서 보면 일대다의 관계가 된다.  
위의 Entity에서 연관관계 주인은 Member에서 관리를 하고 있다. Team은 ReadOnly만 하고있다.  

이것을 일대다의 관계로 바꾼다면, Team에 ```List<Member> members```부분이 연관관계의 주인이 된다는 의미다.  
DB 테이블 입장에서 보면 '다' 쪽에 FK가 정의되기 때문에 위와 같이 Team에서 Member를 추가, 삭제, 변경이 이루어지게 되면 Member Table에 해당 쿼리가 수행되어 유지보수하기 어려워 질 수 있다. (아래 쿼리수행 상태참고)  

위와 같은 현상이 있기에 일대다 단방향 매핑을 설계하기 보다는 다대일 양방향 매핑을 사용하는 것 이 유지보수 차원에서 더 좋은 설계가 될 수 있다.  

### Membe Entity (단방향)
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;
}   
```

### Team Entity (단방향)
```java
@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany
    @JoinColumn(name = "MEMBER_ID")
    private List<Member> members = new ArrayList<>();
}
```

### 사용 (단방향)
```java
Member member = new Member();
member.setName("kwon");
em.persist(member);

List<Member> memberList = new ArrayList<>();
memberList.add(member);

Team team = new Team();
team.setName("team1");
team.setMembers(memberList);
em.persist(team);
```

### 수행된 쿼리 
```sql
Hibernate: 
    /* insert hellojpa.domain.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* insert hellojpa.domain.Team
        */ insert 
        into
            Team
            (name, id) 
        values
            (?, ?)
Hibernate: 
    /* create one-to-many row hellojpa.domain.Team.members */ update
        Member 
    set
        MEMBER_ID=? 
    where
        id=?
```

### 일대다 (양방향)

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(insertable = false, updatable = false)
    private Team team;
}
```

일대다의 양방향 매핑을 단방향의 상태에서 Member객체 즉 '다' 쪽에 ReanOnly 설정으로 매핑시키면 된다.  
굳이 위와같이 할 필요 없이.. 다대일 양방향을 사용하도록하자. 강의를 들으며 몇가지 단점들이 있었지만 제일 크게 와닿은 것은 쿼리가 다른 테이블로 수행이되는 구조가 되는것이 너무 안좋게 느껴졌다.  

추후 DBA와의 이슈추적에 방해가 심하게 있을 것으로 보임.

## 일대일 [1 : 1]
일대일은 반대도 일대일이기에 주 테이블이나 대상 테이블 중에 외래 키를 선택할 수 있다. 다대일 단방향, 양방향 매핑과 동일하며 어노테이션만 **OneToOne**으로 설정이 이루어지면 된다.  
또한 외래키에 유니크제약 조건이 들어간다.  

일대일관계는 DB에서 외래키가 있는 Entity에서 연관관계 주인으로 설정하여 관리해야 한다.

#### **주 테이블에 외래 키**
- 주 객체가 대상 객체의 참조를 가지는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 찾음.  
- 객체지향적.  
- JPA 매핑 관리 가능  
- 주 테이블 조회가 이루어질 경우 대상 테이블의 데이터도 같이 가지고 온다.  
- 대상 테이블에 값이 없을 경우 주 테이블의 외래키 값이 null로 된다.  

### Table

#### Member
|Column|
|-|
|id(PK)|
|locker_id(FK)|
|name|

#### Locker
|Column|
|-|
|id(PK)|
|name|

### Member Entity
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
   
    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}
```

### Locker Entity
```java
@Entity
public class Locker {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

#### **대상 테이블 외래키**
- 대상 테이블에 외래 키가 존재  
- 전통적인 테이터베이스 개발자 선호  
- 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지 된다.  
- 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩이 된다. 그 이유는 대상 테이블에 외래키가 있는지, 없는지 확인 후 값을 셋팅해야 하기에 즉시 쿼리가 수행 되어야 한다.

#### Member
|Column|
|-|
|id(PK)|
|name|

#### Locker
|Column|
|-|
|id(PK)|
|member_id(FK)|
|name|

위와 같은 구조에서 Member에서 단방향 외래키 관리(연관관계 주인) 설정으로 관리하지 못한다. JPA에서 지원을 하지 않는다. 단, 양방향 관리는 지원해준다.

### Member Entity
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;
   
    @OneToOne(mappedBy = "locker")
    private Locker locker;
}
```

### Locker Entity
```java
@Entity
public class Locker {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
}
```

## 다대다 [N : N]

DB Table의 관점에서는 정규화된 테이블 2개로 다대다 관계를 표현할 수 가 없다. 관리를 하려면 중간 레이어에 릴레이션을 맺는 Table이 생성관리해야 한다.  
하지만, 객체의 관점에서는 List컬랙션을 사용하면 관계가 가능하다. 

다대다의 관계는 DB에서도 없는 구조이기때문에 JPA를 활용하여 구현할때는 1:N N:1로 중간 릴레이션 테이블을 Entity를 승격시켜 관리하도록 하자.

### Member Entity
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();
}
```

### Product Entity
```java
@Entity
public class Product {

    @Id @GeneratedValue
    private String id;

    private String name;

    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();
}
```

위의 다대다 관계를 일대다, 다대일 관계로 릴레이션 테이블을 Entity로 변경했을떄는 아래와 같이 된다.

### MemberProduct Entity
```java
@Entity
public class MemberProduct {
    
    @Id @GeneratedValue
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
}
```

### Member Entity
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}
```

### Product Entity
```java
@Entity
public class Product {

    @Id @GeneratedValue
    private String id;

    private String name;

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}
```