## H2 database 다운
- http://www.h2database.com/html/main.html
1. 하단 이미지 다운 참고 (mac은 all platform으로 다운)  
![h2database](https://github.com/sksggg123/TIL/blob/master/images/H2_Database_Engine.gif)

2. zip file 해제
3. mac의 경우 bin 디렉토리 하위에 h2.sh 실행
4. http://127.0.0.1:8082/으로 접속
5. jdbc url을 하단 이미지처럼 변경
![h2url](https://github.com/sksggg123/TIL/blob/master/images/H2_Database_url.gif)
6. pom.xml 설정
```xml
    <dependencies>
        <!--   JAP 하이버네이트   -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.3.10.Final</version>
        </dependency>
        <!--   H2 데이터베이스   -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.199</version>
        </dependency>
    </dependencies>
```
7. persistence.xml 설정
```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```
