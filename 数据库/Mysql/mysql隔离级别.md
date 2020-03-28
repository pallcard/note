# mysql 隔离级别

## 多版本并发控制

多版本并发控制（Multi-Version Concurrency Control, MVCC）是 MySQL 的 InnoDB 存储引擎实现隔离级别的一种具体方式，用于实现提交读和可重复读这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，要求很低，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。

### 基本思想
MVCC 利用了多版本的思想，写操作更新最新的版本快照，而读操作去读旧版本快照，没有互斥关系。在 MVCC 中事务的修改操作（DELETE、INSERT、UPDATE）会为数据行新增一个版本快照。**脏读和不可重复读最根本的原因是事务读取到其它事务未提交的修改。**为了解决脏读和不可重复读问题，MVCC 规定只能读取已经提交的快照。

### 几个概念

#### 版本号
1. 系统版本号 SYS_ID：是一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。
2. 事务版本号 TRX_ID：事务开始时的系统版本号。

#### Undo日志

MVCC 的多版本指的是多个版本的快照，快照存储在 Undo 日志中，该日志通过回滚指针 ROLL_PTR 把一个数据行的所有快照连接起来。
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/28/1585408333610-1585408333698.png)

#### ReadView
当前系统未提交的事务列表TRX_IDs，还有该列表的最小值 TRX_ID_MIN 和 TRX_ID_MAX







