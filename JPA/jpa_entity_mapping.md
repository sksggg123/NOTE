
# 엔티티 매핑


## 객체와 테이블 매핑

###***** @Entity**

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


