# Docker使用

## Docker 官方文档

https://docs.docker.com/install/linux/docker-ce/centos/

## 安装Docker

如果电脑安装过Docker，先执行卸载

```shell
	sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

安装依赖

```shell
sudo yum install -y yum-utils  device-mapper-persistent-data lvm2
```

设置Docker仓库，可以设置阿里云(自行百度)

```shell
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

 安装Docker

```shell
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

启动Docker

```shell
sudo systemctl start docker
```

测试Docker是否安装成功

```shell
sudo docker run hello-world
```

常用命令

```shell
## 启动docker
service docker start
## 重启docker服务
systemctl restart docker.service
## 进入容器内部
docker exec -it 4efb544b4a6c /bin/bash
## 删除镜像
docker rmi name
## 查看所有的容器
docker ps -a
## 查看容器中的进程
ps -aux
## 查看镜像
docker images |grep mysql
## 启动镜像
docker start NAMES
```

## Docker安装Redis

```shell
docker pull redis
```

```shell
docker run -d --privileged=true -p 6379:6379 --restart always -v /root/docker/redis/conf/redis.conf:/etc/redis/redis.conf -v /root/docker/redis/data:/data --name myredis redis redis-server /etc/redis/redis.conf --appendonly yes

 -d                                                  -> 以守护进程的方式启动容器
-p 6379:6379                                        -> 绑定宿主机端口
--name myredis                                      -> 指定容器名称
--restart always                                    -> 开机启动
--privileged=true                                   -> 提升容器内权限
-v /root/docker/redis/conf:/etc/redis/redis.conf    -> 映射配置文件
-v /root/docker/redis/data:/data                    -> 映射数据目录
--appendonly yes                                    -> 开启数据持久化
```

redis设置密码

```shell
redis-cli
# (error) NOAUTH Authentication required.
auth password
config set requirepass yourPassword ## 设置密码
```

## Docker安装Mysql

```shell
在本地创建mysql的映射目录
mkdir -p /root/mysql/data /root/mysql/logs /root/mysql/conf
在/root/mysql/conf中创建 *.cnf 文件(叫什么都行)
touch my.cnf
docker run -p 3306:3306 --name mysql -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/logs -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

```shell
docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v $PWD/conf:/etc/mysql/conf.d -v $PWD/data:/var/lib/mysql -v $PWD/logs:/logs --name test_mysql mysql:5.6  

 参数说明 
  -d 让容器在后台运行 
-p 3306:3306 将容器的 3306 端口映射到主机的 3306 端口
-e 设置环境变量，这里是设置mysql的root用户的初始密码，这个必须设置 
-v $PWD/conf:/etc/mysql/conf.d 将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf
-v $PWD/data:/var/lib/mysql 将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 
-v $PWD/logs:/logs 将主机当前目录下的 logs 目录挂载到容器的 /logs
–name 容器的名字，随便取，但是必须唯一
```

```shell
## 登录
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

CREATE USER 'hywang'@'%' IDENTIFIED BY '123456';
CREATE USER 'hywang'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; 
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
```

## Docker 安装RocketMQ

创建namesrv目录

```shell
mkdir /root/rocketmq/namesrv/logs
mkdir /root/rocketmq/namesrv/store
```

启动namesrv

```shell
docker run -d -p 9876:9876 -v /root/rocketmq/namesrv/logs:/root/logs -v /root/rocketmq/namesrv/store:/root/store --name rmqnamesrv -e "MAX_POSSIBLE_HEAP=100000000" rocketmqinc/rocketmq sh mqnamesrv
```

创建broker挂在目录，并创建broker.conf

```properties
　brokerClusterName = DefaultCluster
　brokerName = broker-a
　brokerId = 0
　deleteWhen = 04
　fileReservedTime = 48
　brokerRole = ASYNC_MASTER
　flushDiskType = ASYNC_FLUSH
　brokerIP1 = 192.168.0.107
```

启动broker

```shell
docker run -d -p 10911:10911 -p 10909:10909 -v  /root/rocketmq/broker/logs:/root/logs -v /root/rocketmq/broker/store:/root/store --name rmqbroker --link rmqnamesrv:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "MAX_POSSIBLE_HEAP=200000000" rocketmqinc/rocketmq sh mqbroker
```

安装rocketmq console

```shell
docker run -d -e "JAVA_OPTS=-Drocketmq.config.namesrvAddr=192.168.0.107:9876 -Drocketmq.config.isVIPChannel=false" -p 8082:8080 -t styletang/rocketmq-console-ng  
```

