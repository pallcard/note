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
### mysql配置
此次实验安装的是mysql 5.6版本
```
# 
docker pull mysql:5.6
# -d 后台运行
# --name 设置名称 mymysql是用户自定义的容器名称
# -p 设置端口映射，第一个3306是当前主机的端口，第二个3306指容器中的端口；
# -e 设置root用户密码为xxxxx；
# mysql:5.6 可以用imageID代替，代表用于创建容器的镜像。
docker run -d --name mymysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=xxxxx mysql:5.6
# 运行容器，一般创建好容器后容器会自动运行
docker start 容器名或容器ID

docker exec -it 容器名或容器ID /bin/bash
#进入mysql
mysql -u root -p
#授权
GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "rw";
flush privileges;
#退出
#第一个exit退出mysql
#第二个exit退出容器的bash
exit
exit
```
### 远程访问mysql遇到的问题
连接不上，修改一下root的密码
```
首先登录MySQL
docker exec -it 容器名或容器ID /bin/bash
mysql> mysql -u root -p
mysql> use mysql;  
mysql> update user set password=password('123') where user='root' and host='localhost';  
mysql> flush privileges;  
```
### redis配置
```
docker pull docker
 
mkdir docker
cd docker
mkdir redis
cd redis
mkdir data
 
#创建启动容器，配置持久化启动
#--privileged=true：容器内的root拥有真正root权限，否则容器内root只是外部普通用户权限
#-v /docker/redis/redis.conf:/etc/redis/redis.conf：映射配置文件
#-v /docker/redis/data:/data：映射数据目录
#redis-server /etc/redis/redis.conf：指定配置文件启动redis-server进程
#--appendonly yes：开启数据持久化
#redistest2  :容器名称
docker run -d --privileged=true -p 6379:6379 -v /docker/redis/redis.conf:/etc/redis/redis.conf -v /docker/redis/data:/data --name redistest2 docker.io/redis:latest redis-server /etc/redis/redis.conf --appendonly yes
```

## 第三步：部署Spring boot项目
1. 使用mvn生成jar包 -----> springboot.jar
2. 新建Dockerfile文件(无后缀名)
    ```
    FROM java:8
    VOLUME /tmp
    COPY schoolAssistant.jar app.jar
    RUN bash -c "touch /app.jar"
    EXPOSE 7000
    ENTRYPOINT ["java", "-jar", "app.jar", "--spring.profiles.active=qa", "--server.port=7000", "> /log/app.log"]
    ```
3. 上传文件Dockerfile和springBootDocker.jar到服务器的同一目录下
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/05/19/1589818682320-1589818682321.png)
4. 将jar打包为镜像（docker build -t boot-docker .）
5. 运行镜像
    ```
    docker run --name spring-boot-docker -d -v /log:/log -p 7000:7000 spring-boot-docker
    ```
6. 查看日志
    ```
    sudo docker logs -f -t --tail -1  myspringboot
    ```

参考：
1. https://blog.csdn.net/No_Game_No_Life_/article/details/95179613?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase
2. https://blog.csdn.net/zhs145612zhs/article/details/82225591
3. https://www.cnblogs.com/whoyoung/p/10988136.html
4. https://www.cnblogs.com/mrhonest/p/10881646.html
5. https://blog.csdn.net/funtaster/article/details/83274727

