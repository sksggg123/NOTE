
## DB형태의 클래스 구조

**Member Domain**
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String name;
    private String city;
    private String street;
    private String zipcode;
}
```

**Order Domain**
```java
@Entity
@Table(name = "orders") // 대부분의 DB에 Order는 예약어로 사용중이기 때문에..
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;
    private Long memberId;
    private LocalDateTime orderDate;
    private OrderStatus state;
}
```

**Item Domain**
```java
@Entity
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;
}
```

**OrderItem Domain**
```java
@Entity
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;
    private Long orderId;
    private Long itemId;
    private int orderPrice;
    private int count;
}<Paste>
```

위의 상태에서 데이터의 조회 및 수정이 될때 소스가 객체지향적으로 나오지 않는다.  
아래 CRUD의 소스를 봐보자..

```java
Member member = new Member();
member.setName("kwon");
em.persist(member);

Order order = new Order();
order.setMemberId(member.getId());
order.setState(OrderStatus.ORDER);
em.persist(order);
```

위에 처럼 Member를 하나 생성 하고 Order에 memberid를 셋팅해주어 참조키를 매핑시키면 아래와 같이 데이터가 만들어진다.  

![joinjpa](https://github.com/sksggg123/TIL/blob/master/images/H2_Database_join_ex1.gif)


이제 위의 매핑된 엔티티를 연관관계 매핑을하여 바꿔보면 아래와 같이 된다.

**Member Domain**
```java
@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "member_id")
    private Long id;
    private String name;
    private String city;
    private String street;
    private String zipcode;
}
```

**Order Domain**
```java
@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne @JoinColumn(name = "member_id")
    private Member Member;

    private LocalDateTime orderDate;
    private OrderStatus state;
}
```

```java
Member member = new Member();
member.setName("kwon");
em.persist(member);

Order order = new Order();
order.setMember(member);
order.setState(OrderStatus.ORDER);
em.persist(order);

Order findOrder = em.find(Order.class, order.getId());
Member findMember = findOrder.getMember();
```
