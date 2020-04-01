# mysql的update in select
You can’t specify target table for update in FROM clause 

【日期】：2020-04-01

【问题】: 想要将一个表中的所有已授权的改成为授权的， `update tb_user set auth = 0 where user_id in (select user_id from tb_user where auth = 1)`

【原因】：***

【如何发现】：***

【如何修复】：***

【总结】：***
