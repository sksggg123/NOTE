

# 연관관계 매핑

## 초기 도메인 상태

**Member Domain**
```java
@Entity
@Table(name = "ex1_member")
public class Ex1Member {

    @Id @GeneratedValue
    private Long id;

    private Long teamId;

    private String name;
}
```

**Team Domain**
```java
@Entity
@Table(name = "ex1_team")
public class Ex1Team {

    @Id @GeneratedValue
    private Long id;

    private String name;
}
```

위의 상태에서 매핑 관계를 사용할때에는 아래와 같은 소스가 된다.  

```java
Ex1Team team = new Ex1Team();
team.setName("team_kwon");
em.persist(team);

Ex1Member member = new Ex1Member();
member.setName("kwon");
member.setTeamId(team.getId());
em.persist(member);

Ex1Member findMember = em.find(Ex1Member.class, member.getId());
Ex1Team findTeam = em.find(Ex1Team.class, member.getTeamId());
```

DB Table끼리는 참조키(FK)로 바로 양방향 접근이 및 조회가 가능하지만 JPA에서는 위와같이 설계가 들어간다면 Team과 Member를 ID를 조회하기 위해서는 각각의 참조하고 있는 ID를 통해서 조회해야 한다.  
객체지향적이지 못한 구조와 접근방법이다. 이러한 현상을 해결하기 위해 단방향 연관관계를 사용할 경우 가능하다.

## 단방향 연관관계

**Member Domain**
```java
@Entity
@Table(name = "ex1_member")
public class Ex1Member {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "team_id")
    private Ex1Team team;

    private String name;
}
```

**Team Domain**
```java
@Entity
@Table(name = "ex1_team")
public class Ex1Team {

    @Id @GeneratedValue
    private Long id;

    private String name;
}
```

```java
Ex1Team team = new Ex1Team();
team.setName("team_kwon");
em.persist(team);

Ex1Member member = new Ex1Member();
member.setName("kwon");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

Ex1Member findMember = em.find(Ex1Member.class, member.getId());
Ex1Team findTeam = findMember.getTeam();
```

위와 같이 단방향 연관관계 설정을 **@ManyToOne**으로 설정을하여 객체지향적으로 설계할 수 있다.  
참고로 flush, clear를 하는 이유는 find할때 Member의 Id를 가지고 오기 위해서는 영속성 컨텍스트의 상태를 DB와 동기화 시키기 위해서이다.  

## 양방향 연관관계

> 참고) 양방향 연관관계는 사실 단방향 연관관계가 서로 이어져있는 부분이다.  

위의 단방향 연관관계 구조를 보면 Member에서 Team의 객체를 가지고 올 수 있다. 하지만 반대는??  
반대로 Team에서 Member로 가지고 올 수가 없다.  

DB의 상태로 보면 Member Table의 FK인 team_id를 통해 Team t join Member m on t.id = m.team_id 쿼리를 통해 참조가 가능하다. 하지만 JPA에서는 참조값을 가지고 있지 않기 때문에 역으로 접근을 할 수가 없다.  
이런 현상을 해결하기 위해 양방향(단방향 2개) 연관관계를 통해 접근이 가능하도록 처리할 수 있다.

**Member Domain**
```java
@Entity
@Table(name = "ex1_member")
public class Ex1Member {

    @Id
    @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "team_id")
    private Ex1Team team;

    private String name;
}
```

**Team Domain**
```java
@Entity
@Table(name = "ex1_team")
public class Ex1Team {

    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team")
    private List<Ex1Member> members = new ArrayList<>();
}
```

```java
Ex1Team team = new Ex1Team();
team.setName("team_kwon");
em.persist(team);

Ex1Member member = new Ex1Member();
member.setName("kwon");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

Ex1Team findTeam = em.find(Ex1Team.class, team.getId());
List<Ex1Member> members = findTeam.getMembers();

for (Ex1Member ex1M : members) {
    System.out.println("member.name = " + ex1M.getName());
}
```

Member 객체는 변한것이 없고 Team 객체에 **@OneToMany()**가 추가되었다.  
OneToMany에서 mappedby에 정의한 "team"은 Member 객체에 @ManyToOne으로 정의한 Team필드의 변수명을 정의한 것이다.  
이렇게 역으로 단방향 연관관계를 정의하여 양방향간 참조가 가능하도록 매핑한것이 양방향 연관관계이다.  

이러한 상태에서 중요한 개념이 있다. DB로 보았을때는 Member Table에 FK인 team_id만 있으면 상호간의 참조가 가능한 상태이다. 하지만 JPA는 불가능하여 위와 같이 양방향 연관관계를 설정하였다.  
여기서 중요한 것은 Member 객체에 team을 수정하면 DB의 데이터가 수정되어야 하는지.. 아니면 Team 객체에 members를 수정하면 되는지 모호한 상태이다.  
이러한 상태를 해소하기 위한 설정이 **mappedby**이다.  

> mappedby로 정의한 필드값은 오로지 read only이다. jpa에 있어 되게 중요한 개념이다.  
> 이러한 개념을 **"연관관계의 주인"** 이라고 한다.

그럼 꼭 Team객체에 members와 같이 정의한 관계에서 mappedby로 설정해야하는것인가??  
그건 아니다. 다만 유지보수 및 DB의 FK가 정의된 객체에 수정권한을 주는 것이 일관성 있고 추후 유지보수가 용이하기에 이처럼 구조를 잡는것을 권장한다.  
유지보수가 용이하다는 말은 어떤의미냐면.. 만약 Member 객체의 Team에 mappedby를 정의할 경우 read only가 되고 Team객체의 members에 수정이 이루어져야 한다.  
이러한 상태가 되면 team객체의 members가 수정이 발생할 경우 DB상 Member Table에 update쿼리가 수행된다. 소스 상으로는 Team을 수정했지만 DB상으로는 FK가 Member이기에 Member Table로 쿼리가 수행되는 것이다.
