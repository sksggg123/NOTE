
# 실전예제 문제

## 조건
- Member, Order, OrderItem, Item 엔티티 만들기.
- 데이터 저장 및 결과값 추출 조건은 아래와 같다.
    1. kwon 고객이
    2. 상품1 (name = a, price = 10, stock = 2)
    3. 상품2 (name = b, price = 20, stock = 1)
    4. 주문을 한다.
        1. 첫 번째 주문은 상품1 1개, 상품2 1개
        2. 두 번째 주문은 상품1 1개
    5. kwon 이 주문한 정보 가지고 오기 (가격)

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

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    // getter setter 생략
}
```    

```java
@Entity
@Table(name = "ORDERS")
public class Order {

    @Id @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private Status status;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    public Order() {
    }

    public Order(Member member) {
        this.status = Status.ORDER;
        this.orderDate = LocalDateTime.now();
        this.member = member;
    }

    public List<OrderItem> getOrderItems() {
        return orderItems;
    }
}
```

```java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "item_id")
    private Item item;

    private int orderPrice;

    private int count;

    public void buy(Order order, Item item, int count) {
        this.order = order;
        this.item = item;
        this.count = count;
        this.orderPrice = item.getPrice() * count;
    }

    public int getOrderPrice() {
        return orderPrice;
    }
}
```

```java
@Entity
public class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;

    private int price;

    private int stockQuantity;

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public void setStockQuantity(int stockQuantity) {
        this.stockQuantity = stockQuantity;
    }
}
```

```java
public enum Status {
    ORDER, CANCEL;
}
```

```java
// kwon 고객이
Member kwon = new Member();
kwon.setName("kwon");
kwon.setCity("city");
kwon.setStreet("street");
kwon.setZipcode("10");
em.persist(kwon);

// 1차 캐시 flush, clear
em.flush();
em.clear();

// 상품1 (name = a, price = 10, stock = 2)
Item item1 = new Item();
item1.setName("a");
item1.setPrice(10);
item1.setStockQuantity(2);
em.persist(item1);

// 상품2 (name = b, price = 20, stock = 1)
Item item2 = new Item();
item2.setName("b");
item2.setPrice(20);
item2.setStockQuantity(1);
em.persist(item2);

// 주문을 한다.
// 첫 번째 주문은 상품1 1개, 상품2 1개
Order order1 = new Order(kwon);
em.persist(order1);

int item1Count = 1;
int item2Count = 1;

OrderItem orderItem1 = new OrderItem();
orderItem1.buy(order1, item1, item1Count);
em.persist(orderItem1);

OrderItem orderItem2 = new OrderItem();
orderItem2.buy(order1, item2, item2Count);
em.persist(orderItem2);

// 두 번째 주문은 상품1 1개
int item3Count = 1;

Order order2 = new Order(kwon);
em.persist(order2);

OrderItem orderItem3 = new OrderItem();
orderItem3.buy(order2, item1, item3Count);
em.persist(orderItem3);

// 1차 캐시 flush, clear
em.flush();
em.clear();

// kwon 이 주문한 정보 가지고 오기 (가격)
Member findMember = em.find(Member.class, kwon.getId());
List<Order> orders = findMember.getOrders();

int orderCount = 0;
int totalPrice = 0;
for (Order order : orders) {

    List<OrderItem> orderItems = order.getOrderItems();
    int innerTotal = 0;
    for (OrderItem orderItem : orderItems) {
        innerTotal += orderItem.getOrderPrice();
    }
    System.out.println((++orderCount) + " 주문가격 : " + innerTotal);
    totalPrice += innerTotal;
}

System.out.println("총 주문 가격 : " + totalPrice);
```

## DB 저장 결과

![DB 저장 결과](https://github.com/sksggg123/TIL/blob/master/images/H2_practice.gif)