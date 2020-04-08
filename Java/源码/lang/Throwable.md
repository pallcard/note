# Throwable

类继承关系图
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/06/1586169529886-1586169529895.png)



说明：
retrun和throw都是使程序跳出当前的方法，自然就是冲突的。如果非要跳出两次那么后者会覆盖前者。 finally里的return语句会覆盖掉try 或者catch 语句里的返回语句


https://blog.csdn.net/abc123lzf/article/details/82115048?depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-3&utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-3


https://blog.csdn.net/anmingyu11/article/details/51741468