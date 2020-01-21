
# 상속관계 매핑

> 관계형 데이터 베이스는 상송관계가 없다.  
> 슈퍼타입, 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.  
> 상송관계 매핑은 객체의 상속과 구조와 DB의 슈퍼타입 서브타입 관계를 매핑한다.  

공통적으로 상속매핑에서 부모 Entity에는 Inheritance 어노테이션을 붙여줘야하며 옵션으로 JOINED, SINGLE_TABLE, TABLE_PER_CLASS 3가지 옵션을 사용하여 DB구조를 정의할 수 있다.  
그 3가지 구조는 아래와 같다.  

## 조인전략
상속(부모) Table과 부모를 상속받은 자식 Table 모두 만들어지면 부모의 PK를 기반으로 자식에게 FK가 생긴다.  

부모 Entity에는 **abstract**를 붙여 만들어주며 @Inheritance 어노테이션을 선언하여 정의한다.  
@DiscriminatorColumn의 경우에는 Item Table에 자식 테이블의 구분값을 넣을 수 있다.  

### Item Entity
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}
```

### Album Entity
```java
@Entity
@DiscriminatorValue("A") // default entity name ex) Album
public class Album extends Item {

    private String artist;
}
```

### Book Entity
```java
@Entity
@DiscriminatorValue("B") // default entity name ex) Book
public class Book extends Item {

    private String author;
    private String isbn;
}
```

### Movie Entity
```java
@Entity
@DiscriminatorValue("M") // default entity name ex) Movie
public class Movie extends Item {

    private String director;
    private String actor;
}
```

![joined](https://github.com/sksggg123/TIL/blob/master/images/H2_join_mapping.gif)


## 단일 테이블 전략

하나의 Table에 자식 Table의 속성(컬럼)이 모두 뭉쳐있는 구조이다.  
하나의 Table에 뭉쳐있다보니 각 자식 Table의 상태값을 필수적으로 구분해야하기에 JPA에서 @DiscriminatorColumn을 선언하지 않더라도 DTYPE을 정의해 준다.  
참고로 단일 테이블의 경우 자식 Entity가 매핑한 속성(컬럼)은 모두 null 허용으로 되어야 한다. 

JOINED 옵션을 SINGLE_TABLE로 변경만 하면 아래와 같이 하나의 단일 Table로 자식 Table의 속성(컬럼)들을 내포하는 구조로 생성이 된다.

```sql
create table Item (
    DTYPE varchar(31) not null,
    id bigint not null,
    name varchar(255),
    price integer not null,
    actor varchar(255),
    director varchar(255),
    artist varchar(255),
    author varchar(255),
    isbn varchar(255),
    primary key (id)
)
```

### Item Entity
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}
```

### Album Entity
```java
@Entity
@DiscriminatorValue("A") // default entity name ex) Album
public class Album extends Item {

    private String artist;
}
```

### Book Entity
```java
@Entity
@DiscriminatorValue("B") // default entity name ex) Book
public class Book extends Item {

    private String author;
    private String isbn;
}
```

### Movie Entity
```java
@Entity
@DiscriminatorValue("M") // default entity name ex) Movie
public class Movie extends Item {

    private String director;
    private String actor;
}
```
![singleTable](https://github.com/sksggg123/TIL/blob/master/images/H2_single_table_mapping.gif)


## 구현 클래스마다 테이블 전략
상속(부모) Entity는 생성이 되지 않고 자식 Entity에만 부모 Entity의 ID와 공통적으로 사용하는 컬럼을 생성하여 준다.
위의 Album, Book, Movie 테이블에 Item에 정의된 id, price, name이 각 자식 Table에 생성이 된다는 의미이다. 

구현 클래스마다 테이블 전략의 경우 공통인 부모 Table이 생성되지 않고 단일 테이블로 뭉쳐지지도 않는데 오로지 자식 Entity 기준으로 Table이 생성되기에 @DiscriminatorColumn이 필요가 없다. 선언을 하더라도 해당 컬럼이 생기지는 않는다.

### Item Entity
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id @GeneratedValue
    private Long id;
    private String name;
    private int price;
}
```

### Album Entity
```java
@Entity
public class Album extends Item {

    private String artist;
}
```

### Book Entity
```java
@Entity
public class Book extends Item {

    private String author;
    private String isbn;
}
```

### Movie Entity
```java
@Entity
public class Movie extends Item {

    private String director;
    private String actor;
}
```

![tablePerClass](https://github.com/sksggg123/TIL/blob/master/images/H2_table_per_class_mapping.gif)

## 장단점 비교

조인전략의 경우 테이블 정규화가 되며, 외래 키 참조 무결성 제약조건 활용이 가능하다. 하지만 조회시 조인 쿼리가 수행이 되며 성능이 다소 저하될 수 있고 쿼리 자체도 복잡해 진다. 또한 데이터 저장 시 insert 쿼리가 부모,자식 Table에 각각 1번 씩 총 2번 수행이 된다.  

단일테이블전략의 경우 하나의 테이블로 모든 자식 값을 갖기 때문에 쿼리가 단순하고 성능이 좋다. 하지만 모든 자식 컬럼은 null을 허용해야하며 데이터가 증대됨에 따라 불필요한 데이터공간이 커지게 된다.  

구현 클래스마다 테이블 전략의 경우 실무에서는 잘 사용하지 않는다. 예를 들어 객체지향적으로 조회를 진행할때에 Union 쿼리가 수행되어 성능이 너무 안좋아 진다.

## 각 전략 별 조회 쿼리 수행 비교

```java
Album album = new Album();
album.setArtist("artist");
album.setName("album name");
album.setPrice(100);
em.persist(album);

Book book = new Book();
book.setAuthor("book");
book.setIsbn("isbn");
book.setName("book name");
book.setPrice(200);
em.persist(book);

Movie movie = new Movie();
movie.setActor("actor");
movie.setDirector("director");
movie.setName("movie name");
movie.setPrice(300);
em.persist(movie);

// 영속성 컨텍스트 (1차 캐시 릴리즈)
em.flush();
em.clear();

// 조회
Item item = em.find(Album.class, album.getId());
System.out.println(item);

tx.commit();
```

### JOINED
Album Entity를 통해 조회가 이루어질 경우 Item Table과 Album Table에 Join 쿼리가 수행되어 결과값이 나온다.  

```sql
select
    album0_.id as id2_2_0_,
    album0_1_.name as name3_2_0_,
    album0_1_.price as price4_2_0_,
    album0_.artist as artist1_0_0_ 
from
    Album album0_ 
inner join
    Item album0_1_ 
        on album0_.id=album0_1_.id 
where
    album0_.id=?
```

### SINGLE_TABLE
단일 테이블은 하나의 테이블을 참조하기 때문에 아래와 같이 수행 됨.  

```sql
select
    album0_.id as id2_0_0_,
    album0_.name as name3_0_0_,
    album0_.price as price4_0_0_,
    album0_.artist as artist7_0_0_ 
from
    Item album0_ 
where
    album0_.id=? 
    and album0_.DTYPE='A'
```

### TABLE_PER_CLASS
단순 album의 데이터를 조회 시 모든 자식 테이블을 union all하여 전체 탐색이 이루어 짐.  

```sql
select
        item0_.id as id1_2_0_,
        item0_.name as name2_2_0_,
        item0_.price as price3_2_0_,
        item0_.actor as actor1_3_0_,
        item0_.director as director2_3_0_,
        item0_.artist as artist1_0_0_,
        item0_.author as author1_1_0_,
        item0_.isbn as isbn2_1_0_,
        item0_.clazz_ as clazz_0_ 
    from
        ( select
            id,
            name,
            price,
            actor,
            director,
            null as artist,
            null as author,
            null as isbn,
            1 as clazz_ 
        from
            Movie 
        union
        all select
            id,
            name,
            price,
            null as actor,
            null as director,
            artist,
            null as author,
            null as isbn,
            2 as clazz_ 
        from
            Album 
        union
        all select
            id,
            name,
            price,
            null as actor,
            null as director,
            null as artist,
            author,
            isbn,
            3 as clazz_ 
        from
            Book 
    ) item0_ 
where
    item0_.id=?
```