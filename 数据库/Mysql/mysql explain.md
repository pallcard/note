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




