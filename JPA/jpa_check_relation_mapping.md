
# 양방향 매핑시 알아 두어야 할 점

## 참고 엔티티

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

## 연관관계의 주인에 값을 입력해야 한다.

```java
Team team = new Team();
team.setName("team");
em.persist(team);

Member member = new Member();
member.setName("member");

team.getMembers().add(member); // check 1
member.setTeam(team); // check 2
em.persist(member);
```

check1에서 값을 설정하고 persist하게 될 경우 DB에는 값이 저장되지 않는다.  
check2에서 값을 설정하고 persist하게 될 경우 DB에 값이 저장된다.  

현재 엔티티의 연관관계 주인에 Member 엔티티에 Team필드가 주인으로 설정이 되어있다. 그렇다면 Member 엔티티에 Team필드에 값을 설정해야 DB에 저장이 된다.  
하지만 객체 지향적 성격을 고려해서 양쪽에 값을 모두 설정하자. (로직적으로..) check 1과 check2 모두 하자는 의미이다.  

그 이유는 check1의 값을 하지않고 find를 하게될 경우 아직 1차캐시(영속성 컨텍스트)에 FK값이 없는(DB에 저장하기 전) 상태의 값을 찾기 때문에 Team 엔티티를 가지고 오지를 못한다.  
그렇기 때문에 양방향 매핑을 할 경우 양쪽 모두에게 저장하는 것이 좋다. 아래는 양쪽모두에게 저장하는 효율적인 방법이다.  

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

    /**
      * Team 엔티티 관리
      */
    public void setTeam(Ex1Team team) {
      this.team = team;
      this.team.getMembers().add(this); // team 엔티티에 read only인 members 필드에 현재 Member 셋팅
    }
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

Ex1Team findTeam = em.find(Ex1Team.class, team.getId());
List<Ex1Member> members = findTeam.getMembers();

for (Ex1Member ex1M : members) {
    System.out.println("member.name = " + ex1M.getName());
}
```

Member 엔티티에 team.getMembers().add(this)를 통해 값을 셋팅해주기 때문에 아래 flush, clear가 없는 (1차 캐시존재 - DB쿼리 insert 쿼리 전) 상태에서도 team.getMembers()를 통해 member를 조회할 수가 있게 된다.  

## 무한 루프를 조심하자

toString()과 같이 양방향에 엔티티를 참조하고 있는 Member(Team) Team(Member List)가 있을 경우 StackOverFlow가 발생하게 된다. 주의하자.  

## 연관관계 주인을 정하는 기준

DB상 FK가 있는 테이블(엔티티)에 필드를 기준으로 정하는게 좋다.
