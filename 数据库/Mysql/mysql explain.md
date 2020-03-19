# mysql explain
## 用法
```sql
# explain sql语句
explain select * from TB_ORDER
```
结果：	
|id|select_type|table|partitions|type|posssible_keys|key|key_len|ref|rows|filtered|Extra|
|-|-|-|-|-|-|-|-|-|-|-|-|
|1|SIMPLE|TB_ORDER|<null>|ALL|<null>|<null>|<null>|<null>|1794|100|<null>	

## 字段说明
### id
SQL执行的顺序的标识,SQL从大到小的执行
* id相同时，执行顺序由上至下
* 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
* id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

### select_type
查询中每个select子句的类型

1. SIMPLE(简单SELECT,不使用UNION或子查询等)
2. PRIMARY(查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY)
3. UNION(UNION中的第二个或后面的SELECT语句)
4. DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)
5. UNION RESULT(UNION的结果)
6. SUBQUERY(子查询中的第一个SELECT)
7. DEPENDENT SUBQUERY(子查询中的第一个SELECT，取决于外面的查询)
8. DERIVED(派生表的SELECT, FROM子句的子查询)
9. UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)

### table
显示这一行的数据是关于哪张表的，有时不是真实的表名字,看到的是derivedx(x是个数字,我的理解是第几步执行的结果)

### type
表示MySQL在表中找到所需行的方式，又称“访问类型”。

常用的类型有： ALL, index,  range, ref, eq_ref, const, system, NULL（从左到右，性能从差到好）

ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

index: Full Index Scan，index与ALL区别为index类型只遍历索引树

range:只检索给定范围的行，使用一个索引来选择行

ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量,system是const类型的特例，当查询的表只有一行的情况下，使用system

NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。



