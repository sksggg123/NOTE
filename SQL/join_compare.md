# SQL Join 별 결과 정리

> Database는 Oracle..

## Test Table 데이터

|Table1|Table2|Table3|
|-|-|-|
|![table1](https://github.com/sksggg123/TIL/blob/master/images/SQL_TABLE_1.PNG)|![table2](https://github.com/sksggg123/TIL/blob/master/images/SQL_TABLE_2.PNG)|![table3](https://github.com/sksggg123/TIL/blob/master/images/SQL_TABLE_3.PNG)|

### Test Table 생성 script
```sql
create table join_table_1 (
    pk NUMBER(20), 
    id NUMBER(20), 
    fk NUMBER(20),
    no NUMBER(20)
);

create table join_table_2 (
    pk NUMBER(20), 
    id NUMBER(20), 
    fk NUMBER(20),
    no NUMBER(20)
);

create table join_table_3 (
    pk NUMBER(20), 
    id NUMBER(20), 
    fk NUMBER(20),
    no NUMBER(20)
);
```

## Outer Join

### table1 full outer join table2 full outer join table3

![full_outer_join_case1](https://github.com/sksggg123/TIL/blob/master/images/full_outer_join_case1.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            full outer join join_table_2 j2 on j2.id = j1.fk
            full outer join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 left outer join table2 left outer join table3

![left_outer_join_case1](https://github.com/sksggg123/TIL/blob/master/images/left_outer_join_case1.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            left outer join join_table_2 j2 on j2.id = j1.fk
            left outer join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 right outer join table2 right outer join table3

![right_outer_join_case1](https://github.com/sksggg123/TIL/blob/master/images/right_outer_join_case1.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            right outer join join_table_2 j2 on j2.id = j1.fk
            right outer join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 left outer join table2 right outer join table3

![outer_join_case2](https://github.com/sksggg123/TIL/blob/master/images/outer_join_case2.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            left outer join join_table_2 j2 on j2.id = j1.fk
            right outer join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 right outer join table2 left outer join table3

![outer_join_case3](https://github.com/sksggg123/TIL/blob/master/images/

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            right outer join join_table_2 j2 on j2.id = j1.fk
            left outer join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

## Inner Join

### table1 join table2 join table3

![inner_join_case1](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case1.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            join join_table_2 j2 on j2.id = j1.fk
            join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 join table2 left join table3

![inner_join_case2](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case2.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            join join_table_2 j2 on j2.id = j1.fk
            left join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 left join table2 join table3

![inner_join_case3](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case3.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            left join join_table_2 j2 on j2.id = j1.fk
            join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 right join table2 join table3

![inner_join_case4](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case4.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            right join join_table_2 j2 on j2.id = j1.fk
            join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 join table2 right join table3

![inner_join_case5](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case5.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            join join_table_2 j2 on j2.id = j1.fk
            right join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 left join table2 left join table3

![inner_join_case6](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case6.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            left join join_table_2 j2 on j2.id = j1.fk
            left join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 right join table2 right join table3

![inner_join_case7](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case7.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            right join join_table_2 j2 on j2.id = j1.fk
            right join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 left join table2 right join table3

![inner_join_case8](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case8.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            left join join_table_2 j2 on j2.id = j1.fk
            right join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```

### table1 right join table2 left join table3

![inner_join_case9](https://github.com/sksggg123/TIL/blob/master/images/inner_join_case9.PNG)

```sql
select  
j1.pk as "1_pk", j1.id as "1_id", j1.fk as "1_fk", j1.no as "1_no",
j2.pk as "2_pk", j2.id as "2_id", j2.fk as "2_fk", j2.no as "2_no",
j3.pk as "3_pk", j3.id as "3_id", j3.fk as "3_fk", j3.no as "3_no"
from    join_table_1 j1
            right join join_table_2 j2 on j2.id = j1.fk
            left join join_table_3 j3 on j3.id = j2.fk
order by j1.pk, j2.pk, j3.pk
```