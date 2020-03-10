
# 엔티티 매핑


## 객체와 테이블 매핑

### @Entity

JPA에서 해당 어노테이션이 선언된 객체를 DB테이블과 매핑 시킨다. 기본적으로 클래스 이름을 DB 테이블 이름으로 그대로 사용한다.  
참고로 JPA에서 @Entity를 사용할때 주의할 점이 있다. 
- 기본생성자가 필수로 필요
- final클래스, enum, interface, inner클래스는 사용이 불가
- 필드변수에 final을 사용불가

```java
@Entity
public class Member {
}
```

### **@Table
엔티티와 매핑할 테이블을 지정할때 사용한다. name 옵션을 통해 DB 테이블과 매핑시킬 수 있다.

## DB 스키마 생성
JPA 설정 상 "<property name="hibernate.hbm2ddl.auto" value="" />" 있다. valute 값에 아래의 5가지 중에 하나로 설정을 하면 된다.
- create : 기존 테이블 drop 후 create
- create-drop : 애플리케이션 종료 시점 테이블 drop
- update : 변경된 사항만 반영 단, 컬림이 추가되면 alter문이 수행되지만 필드변수가 삭제되어 컬럼이 삭제가 필요한 경우 따로 변경분이 없다.
- validate : 엔티티와 DB 테이블이 정상 매핑이 되었는지 확인
- none : 설정 비활성화

로컬에선 create, create-drop, update 필요한 설정으로 마음대로 사용해도 되지만 개발서버 이상 여러개발자들이 사용할떄는 validate 이상 단계로 사용하는게 좋다.  
이유는 테스트 데이터의 소실이 발생할 수 있기 때문이다. 운영에서는 none으로 설정하는게 맞다.

## 필드와 컬럼 매핑

|어노테이션|설명|
|-|-|
|@Column|DB 컬럼과 매핑|
|@Temporal|날째 타입 매핑|
|@Enumerated|enum 타입 매핑|
|@Lob|BLOB, CLOB 매핑|
|@Transient|특정 필드를 컬럼에 매핑하지 않음|

```java
@Entity
public class Member {

    @Id
    private String id;

    @Column(nullable = false, length = 10, name = "mbr_name")
    private String name;

    @Enumerated(value = EnumType.ORDINAL) // Enum의 문자의 번호를 DB에 적재
    @Enumerated(value = EnumType.STRING)  // Enum의 문자를 DB에 적재
    private MeberEnum meberEnum;
    
    @Lob
    private String description;

    @Temporal(value = TemporalType.TIMESTAMP)
    private Date localDateTime;

    @Temporal(value = TemporalType.DATE)
    private Date localDate;

    @Temporal(value = TemporalType.TIME)
    private Date localDateTime2;
}

// DB 테이블 생성 스키마 (JPA에서 만들어 줌)
/**
  create table Member (
       id varchar(255) not null,
        description clob,
        localDate date,
        localDateTime timestamp,
        localDateTime2 time,
        meberEnum varchar(255),
        mbr_name varchar(10) not null,
        primary key (id)
    )
  */
```

### @Column
위의 엔티티 중 Column어노테이션을 보면 3가지의 옵션이 있다. "nullable, length, name"  
null을 허용하지 않고 컬럼의 사이즈가 10이며, 매핑할 DB 컬럼의 명칭이 mbr_name인 상태.

### @Temporal
날짜를 매핑할때 사용 한다. 날짜 필드의 타입이 Date인데 현재 최신버전의 하이버네이트에서 LocalDate, LocalDateTime을 지원해주며 해당 타입으로 선언 시 어노테이션 생략이 가능하다.

### @Enumerated
enum 타입을 매핑할때 사용하며, value옵션에서 ORDINAL, STRING 2가지를 사용할 수 있다. 보편적으로 STRING을 사용한다.

### @Transient
DB의 컬럼과 매핑처리를 하지 않을때 사용한다.

### @Lob
Lob어노테이션에는 지정할 수 있는 옵션이 없으며, String일 경우 CLOB, byte일 경우 BLOB으로 설정이 된다.


## 기본 키 매핑

### @Id

@Id 어노테이션만 사용할 경우 직접 해당 필드값을 기본키(PK)로 사용하는 것.

```java
@Entity
public class Member {

    @Id
    private String id;

    @Column(nullable = false, length = 10)
    private String name;
}

// setId(""); 이 없기 때문에 수행이 되지 않는다.
Member member = new Member();
member.setName("KWON");
em.persist(member);

// 기본키은 id를 필수로 셋팅해주어야 한다.
// 정상동작!
Member member = new Member();
member.setId("1");
member.setName("KWON");
em.persist(member)
```

### @GeneratedValue

@GeneratedValue 어노테이션은 기본키를 자동할당 해준다. 추가적인 옵션으로는 아래 4가지가 있다.
- IDENTITY : 데이터베이스에 위임.
- SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용
- TABLE : 키 생성용 테이블 사용
- AUTO : 방언에 따라 자동 지정, 기본값

IDENTITY의 경우 기본키 생성을 데이터베이스에 위임한다.주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.  
JPA는 보통 트랜잭션 커밋 시점에 Insert SQL이 실행 된다. 그렇기 때문에 AUTO INCREMENT는 데이터베이스에 Insert할때 생성이 되기때문에 영속성 컨텍스트에서는 값을 알 수가 없다.  
그렇기때문에 IDENTITY는 예외적으로 persist할때 insert쿼리가 수행이 된다. 그 후에 내부적으로 select해서 값을 영속성 컨텍스트에 올린다. 아래 소스와 수행된 쿼리 절차를 확인해보자.  

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private String id;

    @Column(nullable = false, length = 10)
    private String name;
}

Member member = new Member();
member.setName("KWON");
System.out.println("===========영속성 컨텍스트 수행 시작==========");
em.persist(member);
System.out.println("===========영속성 컨텍스트 수행 종료==========");

tx.commit(); // 원래라면 이부분에 insert 쿼리가 수행되야 한다.

/**
  ===========영속성 컨텍스트 수행 시작==========
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (id, name) 
        values
            (null, ?)
===========영속성 컨텍스트 수행 종료==========
  */
```

SEQUENCE의 경우에는 유일한 값을 순서대로 생성하는 데이터베이스 오브젝트 이다.  
Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용을 한다.  

추가로 알아둘 옵션은 **initialValue**과 **allocationSize**이다. 시퀀스의 시작과 한번의 호출로 가지고 오는 시퀀스의 범위이다.  
이 설정은 동시성 문제를 해소해주는데 도움이 된다.  

이유는 다음과 같다.  

initialValue는 "1" allocationSize는 "10"으로 설정을 했다고 가정을 하면 영속성 컨텍스트에 저장할때 시퀀스의 번호를 불러올때 1~10 10개의 시퀀스 번호를 미리 발급받아 사용을 한다.  

```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        sequenceName = "MEMBER_SEQ",
        initialValue = 1, allocationSize = 1
)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
            generator = "MEMBER_SEQ_GENERATOR"
    )
    private Long id;

    @Column(nullable = false, length = 10)
    private String name;
}

Member member = new Member();
member.setName("KWON");
System.out.println("===========영속성 컨텍스트 수행 시작==========");
em.persist(member);
System.out.println("===========영속성 컨텍스트 수행 종료==========");

tx.commit();

/**
Hibernate: create sequence MEMBER_SEQ start with 1 increment by 1 // 자동으로 시퀀스 생성하는 쿼리. 애플리케이션 실행시점

===========영속성 컨텍스트 수행 시작==========
Hibernate: 
 call next value for MEMBER_SEQ
===========영속성 컨텍스트 수행 종료==========

Hibernate: 
    /* insert hellojpa.Member
            */ insert 
        into
                    Member
                                (name, id) 
          values
                      (?, ?)
*/
```


추가로 알아둘 옵션은 initialValue과 allocationSize이다. 시퀀스의 시작과 한번의 호출로 가지고 오는 시퀀스의 범위이다.
해당 설정은 동시성 문제를 해소해주는데 도움이 된다. 이유는 다음과 같다.  

initialValue는 "1" allocationSize는 "10"으로 설정을 했다고 가정을 하면 영속성 컨텍스트에 저장할때 시퀀스의 번호를 불러올때 1~10 10개의 시퀀스 번호를 미리 발급받아 사용을 한다.

```java
Member member1 = new Member();
member1.setName("KWON");

Member member2 = new Member();
member2.setName("BYEONG");

Member member3 = new Member();
member3.setName("YUN");

System.out.println("===========영속성 컨텍스트 수행 시작==========");
em.persist(member1); // seq = 1 | memory = 10
em.persist(member2); // seq = 2 | memory = 10
em.persist(member3); // seq = 3 | memory = 10
System.out.println("===========영속성 컨텍스트 수행 종료==========");

tx.commit();
```

위의 로직이 수행되면 시퀀스의 상태는 아래 이미지처럼 된다.  
![시퀀스](https://github.com/sksggg123/TIL/blob/master/images/H2_Sequence.gif)

TABLE은 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략이다. 장점으로는 모든 데이터베이스에서 사용 가능한 점이다.


## 권장하는 기본키 전략
- 기본 키 제약 조건 not null
- 대리키(대체키)를 사용하자(주민번호와 같은 키는 사용하지말자.)
- Long형으로 설정을 하자, Integer로 설정 시 10억 이상으로 넘어가게 될 경우 다시 초기화가 되기 때문이다.
