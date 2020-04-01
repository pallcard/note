# mysql的update in select
You can’t specify target table for update in FROM clause 

【日期】：2020-04-01

【问题】: 想要将一个表中的所有已授权的改成为授权的， `update tb_user set auth = 0 where user_id in (select user_id from tb_user where auth = 1)`

【原因】：不能先select出同一表中的某些值，再update这个表(在同一语句中)

【如何发现】：直接执行sql

【如何修复】：在select 中加一层中间表， `update tb_user set auth = 0 where user_id in (select  select user_id from tb_user where auth = 1)`

【总结】：***
