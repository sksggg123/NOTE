
## 객체와 관계형 데이터베이스의 차이
1. 상속
2. 연관관계
3. 데이터 타입
4. 데이터 식별 방법

## JPA (Java Persistence API)
- 객체는 객체대로 설계를 하며 관계형 데이터베이스는 관계형 데이터베이스대로 설계를 하고 중간에 ORM 프레임워크가 매핑작업을 해준다.
- 애플리케이션과 JDBC 사이에서 동작

## JPA 동작 - 저장
- 애플리케이션을 통해 객체를 전달받아 Entity를 분석하여 JDBC API를 통해 DB에 Insert 한다.

## JPA는 표준 명세
- JPA는 인터페이스의 모음
- JPA 2.1 표준 명세를 구현한 3가지 구현체
- 하이버네이트, EclipseLink, DataNucleus

## JPA를 왜 사용해야 하는가?
- SQL 중심적인 개발에서 객체 중심으로 개발
  - C : jpa.persist(object)
  - R : jpa.find(objectId)
  - U : object.setFiled("")
  - D : jpa.remove(object)
- 생산성
- 유지보수
  - 유지보수 측면에서 기존 필드 변깅 시 모든 SQL 수정할 필요없이 Domain에 필드만 추가해 주면 된다.
- 패러다임의 불일치 해결
  1. JPA와 상속
  2. JPA와 연관관계
  3. JPA와 객체 그래프 탐색
  4. JPA와 비교하기
- 성능
- 데이터 접근 추상화와 벤더 독립성
- 표준

## JPA의 장점
- JPA를 통해 가지고온 Domain객체의 경우 .get()을 할 경우 연관된 그래프 탐색을 하여 조회할 수 있다.
```java
String memberId = "100";
Member member = jpa.find(Member.class, memberId);
member.getTeam();
member.getOrder().getDelivery();
```
- JPA는 동일한 트랜잭션에서 조회한 엔티티 같음을 보장해준다.
``java
String memberid = "100";
Member member1 = jpa.find(Member.class, memberId);
Member member2 = jpa.find(Member.class, memberId);

member1 == member2; // 같다
```

## JPA의 성능 최적화 기능
1. 1차 캐시와 동일성 보장
  > 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 약간의 조회 성능 향상
2. 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
  > 트랜잭션을 커밋할 때까지 Insert SQL을 모음  
  > JDBC Batch SQL 기능을 사용해서 한번에 SQL 전송  
3. 지연로딩(Lazy Loading)
  > 지연로딩은 객체가 실제 사용될 때 로딩  
  ```java
  Member member = memberDAO.find(memberId); // -> select * from member
  Team team = member.getTeam();              
  String teamName = team.getName();         // -> select * from team
  ```
  > 즉시로딩의 경우 JOIN SQL로 한번에 연관된 객체까지 미리 조회  
  ```java
  Member member = memberDAO.find(memberId); // -> select m.*, t.* from member join team
  Team team = member.getTeam();
  String teamName = team.getName();
  ```
  > 위의 지연로딩과 즉시로딩은 옵션으로 활성화 / 비활성화가 가능하다. (튜닝이 간편)

