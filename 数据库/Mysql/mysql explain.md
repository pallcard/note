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








