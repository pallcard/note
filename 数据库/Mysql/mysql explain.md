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






