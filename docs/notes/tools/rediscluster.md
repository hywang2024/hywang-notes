# Redis Cluster

安装Redis Cluster6.0.9

在同一台机器安装伪集群，3主3从

解压文件后

```shell
cd /usr/local/soft/redis-6.0.9
mkdir redis-cluster
cd redis-cluster
mkdir 7291 7292 7293 7294 7295 7296
```

复制redis配置文件到7291目录

```shell
cp /usr/local/soft/redis-6.0.9/redis.conf /usr/local/soft/redis-6.0.9/redis-cluster/7291
```

修改7291的redis.conf配置文件，内容：

```shell
cd /usr/local/soft/redis-6.0.9/redis-cluster/7291
>redis.conf
vim redis.conf
```

```properties
port 7291
daemonize yes
protected-mode no
dir /usr/local/soft/redis-6.0.9/redis-cluster/7291/
cluster-enabled yes
cluster-config-file nodes-7291.conf
cluster-node-timeout 5000
appendonly yes
pidfile /var/run/redis_7291.pid
```

如果是外网集群需要增加以下配置

```properties
# 实际给各节点网卡分配的IP（公网IP）
cluster-announce-ip 47.xx.xx.xx
# 节点映射端口
cluster-announce-port ${PORT}
# 节点总线端口
cluster-announce-bus-port 1${PORT}
```

把7291下的redis.conf复制到其他5个目录

```shell
cd /usr/local/soft/redis-6.0.9/redis-cluster/7291
cp redis.conf ../7292
cp redis.conf ../7293
cp redis.conf ../7294
cp redis.conf ../7295
cp redis.conf ../7296
```

批量替换内容

```shell
cd /usr/local/soft/redis-6.0.9/redis-cluster
sed -i 's/7291/7292/g' 7292/redis.conf
sed -i 's/7291/7293/g' 7293/redis.conf
sed -i 's/7291/7294/g' 7294/redis.conf
sed -i 's/7291/7295/g' 7295/redis.conf
sed -i 's/7291/7296/g' 7296/redis.conf
```

启动6个Redis节点

```shell
cd /usr/local/soft/redis-6.0.9/
./src/redis-server redis-cluster/7291/redis.conf
./src/redis-server redis-cluster/7292/redis.conf
./src/redis-server redis-cluster/7293/redis.conf
./src/redis-server redis-cluster/7294/redis.conf
./src/redis-server redis-cluster/7295/redis.conf
./src/redis-server redis-cluster/7296/redis.conf
```

检查是否启动成功

```shell
ps -ef|grep redis
```

启动成功后，创建集群(注意用绝对IP，不要用127.0.0.1)

```shell
cd /usr/local/soft/redis-6.0.9/src/
redis-cli --cluster create 192.168.44.181:7291 192.168.44.181:7292 192.168.44.181:7293 192.168.44.181:7294 192.168.44.181:7295 192.168.44.181:7296 --cluster-replicas 1
```

Redis会给出一个预计的方案，对6个节点分配3主3从，如果认为没有问题，输入yes确认

```
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.44.181:7295 to 192.168.44.181:7291
Adding replica 192.168.44.181:7296 to 192.168.44.181:7292
Adding replica 192.168.44.181:7294 to 192.168.44.181:7293
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 2058bd8fc0def0abe746816221c5b87f616e78ae 192.168.44.181:7291
   slots:[0-5460] (5461 slots) master
M: 4e41266f2fb7944420d66235475318f5f9526cd8 192.168.44.181:7292
   slots:[5461-10922] (5462 slots) master
M: 9ff8ac86b5faf3c0eca149f090800efea3b142e0 192.168.44.181:7293
   slots:[10923-16383] (5461 slots) master
S: 8383088d3ce75732fc9acb31ca4bce68833028f7 192.168.44.181:7294
   replicates 9ff8ac86b5faf3c0eca149f090800efea3b142e0
S: d185adbfa62133e30cee291b028eff451502ecca 192.168.44.181:7295
   replicates 2058bd8fc0def0abe746816221c5b87f616e78ae
S: 634709cf14809976ea40b65615f816e31d424748 192.168.44.181:7296
   replicates 4e41266f2fb7944420d66235475318f5f9526cd8
Can I set the above configuration? (type 'yes' to accept): 
```

注意看slot的分布：

```
7291  [0-5460] (5461个槽) 
7292  [5461-10922] (5462个槽) 
7293  [10923-16383] (5461个槽)
```

输入yes确认，集群创建完成

```
>>> Performing Cluster Check (using node 192.168.44.181:7291)
M: 2058bd8fc0def0abe746816221c5b87f616e78ae 192.168.44.181:7291
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 8383088d3ce75732fc9acb31ca4bce68833028f7 192.168.44.181:7294
   slots: (0 slots) slave
   replicates 9ff8ac86b5faf3c0eca149f090800efea3b142e0
S: d185adbfa62133e30cee291b028eff451502ecca 192.168.44.181:7295
   slots: (0 slots) slave
   replicates 2058bd8fc0def0abe746816221c5b87f616e78ae
M: 9ff8ac86b5faf3c0eca149f090800efea3b142e0 192.168.44.181:7293
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 4e41266f2fb7944420d66235475318f5f9526cd8 192.168.44.181:7292
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 634709cf14809976ea40b65615f816e31d424748 192.168.44.181:7296
   slots: (0 slots) slave
   replicates 4e41266f2fb7944420d66235475318f5f9526cd8
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

重置集群的方式是在每个节点上个执行`cluster reset`，然后重新创建集群

```
cd /usr/local/soft/redis-6.0.9/redis-cluster/
vim setkey.sh
```

setkey.sh脚本内容

```properties
#!/bin/bash
for ((i=0;i<20000;i++))
do
echo -en "helloworld" | redis-cli -h 192.168.44.181 -p 7291 -c -x set name$i >>redis.log
done
```

给脚本增加权限

```shell
chmod +x setkey.sh
./setkey.sh
```

链接到客户端

```
redis-cli -p 7291
redis-cli -p 7292
redis-cli -p 7293
```

每个节点分布的数据

```
127.0.0.1:7291> dbsize
(integer) 6652
127.0.0.1:7292> dbsize
(integer) 6683
127.0.0.1:7293> dbsize
(integer) 6665
```

新增节点如何重新分片：
一个新节点add-node加入集群后，是没有slots的

```
redis-cli --cluster reshard 目标节点（IP端口）
```

这时会要求你输入分配的槽位，生成reshard计划，确定就会迁移数据

### cluster管理命令

查看所有命令

```
redis-cli --cluster help
```

```
Cluster Manager Commands:
  create         host1:port1 ... hostN:portN
                 --cluster-replicas <arg>
  check          host:port
                 --cluster-search-multiple-owners
  info           host:port
  fix            host:port
                 --cluster-search-multiple-owners
  reshard        host:port
                 --cluster-from <arg>
                 --cluster-to <arg>
                 --cluster-slots <arg>
                 --cluster-yes
                 --cluster-timeout <arg>
                 --cluster-pipeline <arg>
                 --cluster-replace
  rebalance      host:port
                 --cluster-weight <node1=w1...nodeN=wN>
                 --cluster-use-empty-masters
                 --cluster-timeout <arg>
                 --cluster-simulate
                 --cluster-pipeline <arg>
                 --cluster-threshold <arg>
                 --cluster-replace
  add-node       new_host:new_port existing_host:existing_port
                 --cluster-slave
                 --cluster-master-id <arg>
  del-node       host:port node_id
  call           host:port command arg arg .. arg
  set-timeout    host:port milliseconds
  import         host:port
                 --cluster-from <arg>
                 --cluster-copy
                 --cluster-replace
  help           

For check, fix, reshard, del-node, set-timeout you can specify the host and port of any working node in the cluster.
```

### 集群命令

cluster info ：打印集群的信息
cluster nodes ：列出集群当前已知的所有节点（node），以及这些节点的相关信息。
cluster meet ：将 ip 和 port 所指定的节点添加到集群当中，让它成为集群的一份子。
cluster forget <node_id> ：从集群中移除 node_id 指定的节点(保证空槽道)。
cluster replicate <node_id> ：将当前节点设置为 node_id 指定的节点的从节点。
cluster saveconfig ：将节点的配置文件保存到硬盘里面。

### 槽slot命令

cluster addslots [slot …] ：将一个或多个槽（slot）指派（assign）给当前节点。
cluster delslots [slot …] ：移除一个或多个槽对当前节点的指派。
cluster flushslots ：移除指派给当前节点的所有槽，让当前节点变成一个没有指派任何槽的节点。
cluster setslot node <node_id> ：将槽 slot 指派给 node_id 指定的节点，如果槽已经指派给另一个节点，那么先让另一个节点删除该槽>，然后再进行指派。
cluster setslot migrating <node_id> ：将本节点的槽 slot 迁移到 node_id 指定的节点中。
cluster setslot importing <node_id> ：从 node_id 指定的节点中导入槽 slot 到本节点。
cluster setslot stable ：取消对槽 slot 的导入（import）或者迁移（migrate）。

### 键命令

cluster keyslot ：计算键 key 应该被放置在哪个槽上。
cluster countkeysinslot ：返回槽 slot 目前包含的键值对数量。
cluster getkeysinslot ：返回 count 个 slot 槽中的键