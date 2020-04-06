# Throwable

类继承关系图
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/06/1586169529886-1586169529895.png)



说明：
retrun和throw都是使程序跳出当前的方法，自然就是冲突的。如果非要跳出两次那么后者会覆盖前者。 finally里的return语句会覆盖掉try 或者catch 语句里的返回语句