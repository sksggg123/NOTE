
## JPA 영속성 컨텍스트

> 영속성 이란 "엔티티를 영구 저장하는 환경"이라는 뜻.  

```java
EntityManager.persist(entity);
```
위의 엔티티 매니저를 통해 엔티티를 저장할때 영속성 컨텍스트에 저장이 된다. (DB에 저장되는 단계가 아님)  
엔티티 매니저를 생성할때 영속성 컨텍스트가 같이 생성이 된다.  

## 엔티티의 생명주기

### 비영속 : 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
```java
// 단순 객체를 생성한 상태
Member member = new Member();
member.setId(100L);
member.setName("kwon");
```

### 영속 : 영속성 컨텍스트에 관리되는 상태
```java
// 객체를 생성한 상태
Member member = new Member();
member.setId(100L);
member.setName("kwon");

// 객체를 저장한 상태, 이때가 영속상태이다.
EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

em.persist(member);
```

### 준영속 : 영속성 컨텍스트에 저장되었다가 분리된 상태
```java
em.detach(member);
```

### 삭제 : 삭제된 상태
```java
em.remove(member);
```

## 영속성 컨텍스트가 있을때의 장점

### 첫 번째 장점
영속성 컨텍스트는 엔티티 매니저가 생성될때 같이 생성이 된다.  
영속성 컨텍스트에 저장이 될때 DB에 저장이 되는 것은 아니다. 단순 JPA의 1차 캐시에 올라가는 것이며, 동일한 엔티티를 조회할때 1차 캐시 (영속성 컨텍스트)에 올라간 엔티티를 반환해준다.  
1차 캐시의 엔티티를 반환하기 때문에 DB를 호출할 필요가 없어 쿼리 수행이 되지 않는다.  

영속성 컨텍스트에서 반환되는 엔티티는 동일하다 아래의 구문이 true라는 의미이다.  
```java
Member member1 = new Member();
member1.setId(1L);
member1.setName("kwon");

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

em.persist(member)     // 영속성 컨텍스트에 member1 엔티티가 저장이 된다.

Member member2 = em.find(Member.class, 1L);  // 영속성 컨텍스트에 있는 member1 값을 반환한다. (2번째 인자값인 1L을 통해 member1 반환) - 쿼리가 수행되지 않음.
member1 == member2 // "true"이다. 자바에서 볼때 참조값이 같은 상황이다.
```

### 두 번째 장점
영속성 컨텍스트에서는 쓰기 지연을 지원한다.  
```java
em.persist(member1);
em.persist(member2);

transaction.commit();
``` 
위와 같은 로직으로 수행이 되었다면  수행이 되었다면 member1,2가 persist되었을때 영속성 컨텍스트에 등록이 된다. DB에 저장이 된 상태는 아니다.  
이러한 상황이 쓰기 지연상태 이다.  

마지막에 트랙잭션을 commit할때 insert쿼리 2개가 수행이되어 DB에 적재가 된다.  
참고로 commit을 하기전 detach, remove와 같이 영속상태의 엔티티를 없애게 되면 commit 시점에 쿼리가 수행이 되지 않는다.

> 영속성 컨텍스트의 장점을 정리하자면..  
> 영속성 컨텍스트(1차 캐시)에 저장이 됨과 동시에 쓰기지연 SQL 저장소에 쿼리를 저장해 둔다.  
> 트랙잭션 commit시점에 쓰기지연 SQL 저장소에 등록된 쿼리가 일괄로 DB에 수행이 된다.

## 엔티티 수정 (변경감지 - 더티채킹)

엔티티를 찾아 수정을 하게 되면 바로 DB에 업데이트가 되지 않는다.  
트랜잭션 commit이 되는 시점에 1차 캐시(영속성 컨텍스트)에 저장된 엔티티와 엔티티의 최초시점의 스냅샷 상태를 비교하여 변경분이 발생한 엔티티에 대해 update 쿼리를 만들어 주며, DB에 쓰기 지연 SQL 저장소에 등록된 쿼리들이 일괄 수행된다.  

## 플러시

플러시는 엔티티의 변경, 추가, 삭제에 대한 상태를 DB에 반영하는 것이다. DB에 반영할 수 있는 조건은 아래 3가지가 있다.  
1. 트랜잭션 commit
2. ``em.flush()``를 통해 영속성 컨텍스트에 등록된 쿼리들을 트랜잭션 commit하기 전 직접 DB에 동기화
3. JPQL 쿼리가 실행될때 자동으로 플러시가 호출이 된다. (이유는 JPQL은 DB에 직접 쿼리를 날리기 때문에 DB에 직전까지 추가, 변경, 삭제한 상태가 반영이 되야한다.)
