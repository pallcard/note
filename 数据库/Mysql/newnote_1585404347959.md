# mysql sql
### DBMS分类

1. 基于共享文件系统的DBMS，例如：Microsoft Access
2. 基于客户机—服务器的DBMS，例如：MySql，Oracle，SQL Server


### 连接mysql
格式：
```java
mysql -h 主机地址 -u 用户名 -p 用户密码
例如：/usr/local/mysql/bin/mysql -u root -p
```

### 选择数据库
最初连接mysql时，没有任何数据库可供使用，需要使用use来选择一个数据库
```java
use 数据库名称
例如：USE mysql
```

### 显示数据库一些信息

```java
SHOW DATABASES;   //显示所有数据库
SHOW TABLES;       //显示当前数据库中所有表
SHOW COLUMNS FROM xxx;   //显示xxx表中的列
DESCRIBE xxx；//同上
SHOW STATUS;   //用于显示广泛的服务器状态信息
SHOW GRANTS;  //显示授权用户权限
SHOW ERRORS和SHOW WARNINGS；//用来显示服务器错误或警告消息
```

### 检索数据

许多SQL开发人员喜欢对所有SQL关键字使用大写，而对所有 列和表名使用小写，这样做使代码更易于阅读和调试。

#### select

```java
SELECT DISTINCT yyy FROM xxx LIMIT 5; //检索xxx表中yyy列(DISTINCT去重，不多于5)
SELECT * FROM xxx LIMIT 5, 5; //从第5行开始5行，下标0
```
 **注**
DITINCT会作用于所有的列，不会部分使用

#### order by


```java
select * from xxx order by yyy;  
```
**注**
1.默认升序，降序使用DESC
2.可以指定多列，先按第一列排，再按照第二列，。。。
3.在多个列上降序排序 如果想在多个列上进行降序排序，必须 对每个列指定DESC关键字。
4. limit要在order by之后

#### where
```java
select * from xxx where yyy="";
例：
SELECT * FROM news WHERE title = "新闻4" ORDER BY title;
SELECT * FROM news WHERE title BETWEEN "新闻1" AND "新闻5" ORDER BY title;
SELECT * FROM news WHERE creator_id IS NULL LIMIT 10;
```
**注**

1.在SELECT语句中，数据根据WHERE子句中指定的搜索条件进行过滤

2.order by位于where之后

#### AND OR IN

```java
SELECT * FROM news WHERE title BETWEEN "新闻3" AND "新闻6" OR id < 3;
例如：IN的使用
SELECT * FROM news WHERE title NOT IN ("新闻1","新闻2","新闻3","新闻10");
```

**注**

1. SQL（像多数语言一样）在处理OR操作符前，优先处理AND操 作符。**使用圆括号明确地分组相应的操作符**，圆括号具有较AND或OR操作符高的计算次序

2.IN在WHERE子句中用来指定要匹配值的清单的关键字，功能与OR 相当。

### 通配符

```java
SELECT * FROM news WHERE title LIKE "新闻1%"
SELECT * FROM news WHERE title LIKE "新闻_"
```
1. %表示任何字符出现 ***任意次数***,不能匹配到NULL
2. _匹配单个字符

### 正则表达式

```java
SELECT * FROM news WHERE title REGEXP "新闻1|新闻2";
```

1.注意与like的区别

* 搜索两个串之一（或者为这个串，或者为另一个串），使用|,例如1|2
* 匹配任何单一字符[],例如[1,2,3]是匹配1或2或3
* 匹配特殊字符，必须用\\为前导，例如\\.
* 重复元素
* ^有两种用法。在集合中（用[和]定义），用它 来否定该集合，否则，用来指串的开始处。[0-9\\.]一个数（包括以小数点开始的数）

| 字符 | 说明 
| --- | --- 
| * | 0个或多个匹配  
| + | 1个或多个匹配 
| ？ | 0个或1个匹配 
| ｛n｝ | 指定数目的匹配 
| ｛n,｝ | 不少于指定数目的匹配 
| {n,m} | 匹配数目的范围（m不超过255） 
|^ | 文本的开始
|$ | 文本的结尾 

### 计算字段

#### 拼接

拼接（concatenate） 将值联结到一起构成单个值。
```java
SELECT CONCAT("title:",title,",id:",id) AS test2 FROM news LIMIT 0,10;
```
trim(),ltrim(),rtrim(),去掉串左右空格
as 别名
计算+ - * /
#### 函数

##### 聚集函数
| 函数 |说明 |
| ---|---|
|AVG() |返回某列的平均值|
|COUNT() |返回某列的行数 |
|MAX()|返回某列的最大值|
|MIN()|返回某列的最小值|
|SUM()|返回某列值之和 |


**注**
聚集函数中使用 DISTINCT
```java
SELECT COUNT(DISTINCT title) FROM news;
```
### 分组

**格式**
```java
SELECT type, COUNT(*) AS num FROM news GROUP BY type;
```
**筛选**
```java
SELECT *, COUNT(title) AS num FROM news GROUP BY TYPE HAVING num > 2;
```
| 子句 | 说明 | 是否必须使用|
|  ---   | ---  |---|
|SELECT|要返回的列或表达式 |是|
|FROM|从中检索数据的表 |仅在从表选择数据时使用|
|WHERE|行级过滤 |否|
|GROUP BY | 分组说明 |仅在按组计算聚集时使用|
|HAVING|组级过滤 |否|
|ORDER BY|输出排序顺序 |否|
|LIMIT|要检索的行数 |否|

### 子查询
```java
SELECT * FROM theme WHERE id IN (SELECT id FROM news WHERE create_time > "2019-01-12");
```

### 联结

1.内部联结

它基于两个表之间的 相等测试。这种联结也称为内部联结。
```java
SELECT * FROM movie_arrange,films WHERE movie_arrange.id = films.id;
//推荐
SELECT * FROM movie_arrange INNER JOIN films ON movie_arrange.id = films.id;
```
2.外部联结
许多联结将一个表中的行与另一个表中的行相关联。但有时候会需 要包含没有关联行的那些行。
```java
//左外连接，左表所有行保留
SELECT * FROM movie_arrange LEFT OUTER JOIN films ON movie_arrange.id = films.id;
```
### 组合查询

MySQL也允许执行多个查询（多条SELECT语句），并将结果作为单个 查询结果集返回。这些组合查询通常称为并（union）或复合查询
```java
SELECT cinema_id, movie_name FROM movie_arrange
UNION
SELECT id, name FROM films
```
**注**
UNION 会自动取消重复行，如果想返回所有匹配行，可使用UNION ALL

### 全文搜索

为了进行全文本搜索，必须索引被搜索的列，而且要随着数据的改 变不断地重新索引。

全文搜索会将行中内容使用分词器进行拆分，并建立索引，
```java
建表时加入
FULLTEXT (title,body)
查询时，Match()指定被搜索的列，Against()指定要使用的搜索表达式。
SELECT * FROM articles 
WHERE MATCH (title,body)
AGAINST ('database' IN NATURAL LANGUAGE MODE);
```
参考：
https://blog.csdn.net/bbirdsky/article/details/45368897

###  插入 更新 删除

**插入**

```java
INSERT INTO articles (title,body) VALUES
('How To Use MySQL Well','After you went...'),
 ('MySQL Tutorial','DBMS stands for DataBase ...');
```

**更新**

```java
UPDATE articles SET title = 'def' WHERE id = 1;
```

**删除**
```java
DELETE FROM articles WHERE id = 7;
```
**注**

如果省略了WHERE子句，则UPDATE或DELETE将被应用到表中 所有的行

### 表操作
**创建表**
```java
CREATE TABLE articles (
     id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
     title VARCHAR(200) DEFAULT 'no title',
     body TEXT,
     FULLTEXT (title,body)
) ENGINE=INNODB;
```
**更新表**
```java
ALTER TABLE articles ADD score CHAR(2);
ALTER TABLE articles DROP score;
```

**删除表**
```java
DROP TABLE articles
```

**重命名表**
```java
RENAME TABLE articles TO articles2;
```