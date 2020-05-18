# Docker部署springboot过程

## 环境
1. 阿里云服务器，系统centos

## 第一步：Docker的安装
### 检测内核
通过uname -r来检查自己的内核是否是3.x及以上，否则需要升级
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/05/18/1589810615789-1589810615814.png)

### Docker安装
1. 安装一些必要的系统工具：
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
2. 添加软件源信息：
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
3. 更新 yum 缓存：
```
sudo yum makecache fast
```
4. 安装 Docker-ce：
```
sudo yum -y install docker-ce
```
5. 启动 Docker 后台服务:
```
sudo systemctl start docker
```
6. 测试运行 hello-world
docker run hello-world
————————————————
版权声明：本文为CSDN博主「No_Game_No_Life_」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/No_Game_No_Life_/java/article/details/95179613



