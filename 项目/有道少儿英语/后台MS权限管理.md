# 后台MS权限管理
## 数据库设计
数据库使用5张表，分别是 用户表、角色表、权限表、用户角色表、角色权限表（属性省略）
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/08/Popo%E6%88%AA%E5%9B%BE20203811956-1583668518708.png?token=AHBYBJ5HICBD3EV27RU7ZUS6MTOWK)

## 整体思路
1. 项目启动时读取所有接口，更新权限表
2. 修改登录拦截器，判断用户权限 HandlerInterceptorAdapter
3. 角色、权限接口

## 优化&问题
### 问题1：MS后端启动很慢

线上部署涉及集群环境，项目启动时，多个机器会调用权限初始化（更新权限表）接口，故很慢；为了只让一个机器进行权限表的更新，故使用了分布式锁+rediskey（5min过期key）
* 只用分布式锁：第一个机器拿到锁初始化一遍释放锁，第二个应用又拿到锁，进行了初始化。
* 只用rediskey(5min过期key):可能第一个机器拿到key之后，还未设置key，第二个机器就拿到key，也会初始化多次。

### 问题2：连续多次调用登录的接口，自动注册时，会出现多个账号
1.数据库层面，使得账号名为unique，这样当第二次调用使用会出现报错，无法插入。
2.代码层面，在判断用户是否登录过时，使用分布式锁，只让用户就行一次用户插入。

### 优化1：访问速度优化
使用redis缓存优化速度，每次查询用户的拥有的权限时，现在redis里面查询，若没有再去数据库里面查询，并写入redis缓存中。（string）key:前缀_用户名

### 优化2：记录权限操作
添加操作日志，权限操作表

## 涉及知识

### redis分布式锁

* 加锁操作的正确姿势为：

1. 使用setnx命令保证互斥性
2. 需要设置锁的过期时间，避免死锁
3. setnx和设置过期时间需要保持原子性，避免在设置setnx成功之后在设置过期时间客户端崩溃导致死锁
4. 加锁的Value 值为一个唯一标示。可以采用UUID作为唯一标示。加锁成功后需要把唯一标示返回给客户端来用来客户端进行解锁操作
* 解锁的正确姿势为：

1. 需要拿加锁成功的唯一标示要进行解锁，从而保证加锁和解锁的是同一个客户端

2. 解锁操作需要比较唯一标示是否相等，相等再执行删除操作。这2个操作可以采用Lua脚本方式使2个命令的原子性。

### 文件操作
获取ms下，所有的class文件，读取@Controller.class, @RestController.class，然后将class中的方法上RequestMapping，GetMapping,PostMapping的url取到

