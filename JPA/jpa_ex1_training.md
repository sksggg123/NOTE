
> 참고) 데이터베이스 방언!!  
> hibernate.dialect  
> 1. H2: org.hibernate.dialect.H2Dialect  
> 2. Oracle 10g: org.hibernate.dialect.Oracle10GDialect  
> 3. MySQL: org.hibernate.dialect.MySQL5InnoDBDialect  
> 등...  

## JPA 기본 사용법
```java
// 애플리케이션 실행 시점 한번만 수행하여 전체 애플리케이션에서 공유한다.
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
// 쓰레드가 공유 하면 안된다. 한번의 요청에 사용 후 해제를 필요
EntityManager em = emf.createEntityManager();

// 수행쿼리 코드

em.close();  // 요청 종료될때 close를 하여 database connection을 해제처리.
emf.close(); // 애플리케이션 종료 시 close를 통해 자원 릴리즈처리.
```

EntityManagerFactory의 경우 애플리케이션 실행 시점에 한번만 생성하여 애플리케이션이 종료될 때 까지 공유하여 사용한다.  
EntityManager의 경우 한번의 요청에 수행이 종료되면 반환을 하여 Database의 Connection이 물려있는 상태를 종료해 주어야 한다. Transation에 사용되기에 Thread간 공유를 하면 안된다.  

## Entity 만들기
```java
@Entity
public class Member {
  @Id
  private Long id;
  private String name;

  // Getter & Setter
}
```

위의 클래스에 Entity 어노테이션을 붙임으로 JPA에서 감지가 이루어져 필드변수를 컬럼으로 매핑시키게 된다.  
참고로 알아둘 것은 @Table, @Column 어노테이션이다.  
@Table 어노테이션의 경우에는 DB 테이블과 Entity 객체의 이름을 다르게 지정할 수 있다. 예를 들면 DB Table의 이름은 "user"이고 Entity 이름은 "Member"로 정의할때 사용할 수 있다. 

```java
@Entity
@Table(name = "user")
public class Member {
}
```

@Column 어노테이션의 경우에는 @Table과 같이 컬럼에 이름이 다를 경우 매핑시킬때 사용 된다. 예를 들면 User Table의 username 컬림이 있을경우 아래와 같이 작성이 가능하다.

```java
@Entity
@Table(name = "user")
public class Member {
  @Id
  private Long id;

  @Column(name = "username")
  private String name;
}
```

## JPA CRUD...

- Create

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();

try {
  Member member = new Member();
  member.setId(1L);
  member.setName("kwonkun");

  em.persist(member);
  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close();
}
emf.close();
```

- Read

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();

try {
  Member findMember = em.find(Member.class, 1L); // Entity객체, PK

  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close();
}
emf.close();
```

- Update

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();

try {
  Member findMember = em.find(Member.class, 1L);
  findMember.setName("kwon");
  
  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close();
}
emf.close();
```

- Delete

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();
tx.begin();

try {
  Member findMember = em.find(Member.class, 1L);
  em.remove(findMember);

  tx.commit();
} catch (Exception e) {
  tx.rollback();
} finally {
  em.close();
}
emf.close();
```

여기서 특이점은 수정할때 단순 setter에 데이터를 넣어주면 DB의 데이터가 변경되는 부분이다. 이렇게 동작하게되는 이유는 Transation에 묶여있을때 commit이 되는 순간 JPA에서는 데이터의 변경 삭제에 대한 감지가 이루어지며 해당 변형에 대해 쿼리를 생성하여 일괄로 수행되기 때문이다.  
Console Log를 보면 select -> update 쿼리 2개가 수행되는 것을 확인할 수 있다.  
