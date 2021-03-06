# mysql多表查询

## 举例分析
说明:表格仅为举例说明
tb_user_age
|id|age|
|-|-|
|1|10|
|2|11|
|3|12|
|4|13|
tb_user_name
|id|age|
|-|-|
|2|zhang|
|3|li|
|4|zhou|
|5|chen|

```sql
create table tb_user_age (
    id bigint auto_increment comment 'ID' primary key,
    age int null comment '年龄'
)
comment = '用户年龄' engine = innodb charset utf8mb4;

create table tb_user_name (
     id bigint auto_increment comment 'ID' primary key,
     name varchar(30) null comment '姓名'
)
comment = '用户姓名' engine = innodb charset utf8mb4;


insert into tb_user_age (age) values (10),(11),(12),(13);
insert into tb_user_name (id,name) values (2,'zhang'),(3,'li'),(4,'zhou'),(5,'chen');
```

### 内连接 （join 或 inner join）
```sql
select a.*, b.* from tb_user_age a 
inner join tb_user_name b 
on a.id = b.id

# a.id  age    b.id     name
# 2	11	2	zhang
# 3	12	3	li
# 4	13	4	zhou
```
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/Popo%E6%88%AA%E5%9B%BE202031411598-1584158370889.png?token=AHBYBJ6WNC2N4SVHY2JSQPS6NRLOC)
这种场景下得到的是满足某一条件的A，B内部的数据；正因为得到的是内部共有数据，所以连接方式称为内连接。

### 外连接
#### 左连接 （left join 或left outer join)
* 右连接 （right join 或right outer join)与左连接类似
```sql
select a.*, b.* from tb_user_age a 
left join tb_user_name b 
on a.id = b.id
# a.id  age    b.id     name
# 2	11	2	zhang
# 3	12	3	li
# 4	13	4	zhou
# 1	10	<null>	<null>
```
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/Popo%E6%88%AA%E5%9B%BE202031412329-1584158631279.png?token=AHBYBJZEBIELUYE5H4BOQLK6NRL6K)
这种场景下得到的是A的所有数据，和满足某一条件的B的数据;

#### 全连接 （mysql不支持，左连接 union 右连接）
```sql
select a.*, b.* from tb_user_age a 
left join tb_user_name b on a.id = b.id
union
select a.*, b.* from tb_user_age a 
right join tb_user_name b on a.id = b.id

# a.id  age    b.id     name
# 2	11	2	zhang
# 3	12	3	li
# 4	13	4	zhou
# 1	10	<null>	<null>
# <null><null>  5        chen
```

#### 交叉连接（cross join）
表格表做笛卡儿积
```sql
select a.*, b.* from tb_user_age a 
cross join tb_user_name b

# a.id  age    b.id     name
# 1	10	2	zhang
# 2	11	2	zhang
# 3	12	2	zhang
# 4	13	2	zhang
# 1	10	3	li
# 2	11	3	li
# 3	12	3	li
# 4	13	3	li
# 1	10	4	zhou
# 2	11	4	zhou
# 3	12	4	zhou
# 4	13	4	zhou
# 1	10	5	chen
# 2	11	5	chen
# 3	12	5	chen
# 4	13	5	chen
```
* cross 连接， 后面可以使用on或where，当使用on a.id = b.id 时，则和内连接是一样的结果。

## 总结
* 一般cross join后面加上where条件，但是用cross join+on也是被解释为cross join+where；
* 如果连接表格使用的是逗号，会被解释为交叉连接，但后面条件只能使用where；
* 其他连接只能使用on
