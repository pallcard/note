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
6. 验证是否安装成功
sudo docker info

## 第二步：mysql、redis配置
### mysql
此次实验安装的是mysql 5.6版本
```

docker pull mysql:5.6
docker run -d --name mymysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxxx mysql:5.6
```


参考：
1. https://blog.csdn.net/No_Game_No_Life_/article/details/95179613?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase

2. https://blog.csdn.net/zhs145612zhs/article/details/82225591

