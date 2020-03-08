# 后台MS权限管理
## 数据库设计
数据库使用5张表，分别是 用户表、角色表、权限表、用户角色表、角色权限表（属性省略）
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/08/Screen%20Shot%202019-06-25%20at%2011.42.16%20AM-1583637023859.png?token=AHBYBJ3GYFIXXCLZQADJBHK6MRRF6)

## 整体思路
1. 项目启动时读取所有接口，更新权限表
2. 
