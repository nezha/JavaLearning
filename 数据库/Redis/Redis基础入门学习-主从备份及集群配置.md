
# Redis基础入门学习-主从备份及集群配置

## Redis主从备份

### 1.创建Redis节点

> 我们在`redis-3.2/redis_cluster/`下分别创建三个文件夹`/6000/`,`/6001/`和`/6002/`，这种方式用于放置配置文件，模拟创建3个节点。

```bash
cp ./etc/redis.conf ./redis_cluster/6000/
cp ./etc/redis.conf ./redis_cluster/6001/
cp ./etc/redis.conf ./redis_cluster/6002/
```

> 假设master节点是6000端口

### 2.修改配置文件

- `master`节点 > `6000`

```bash
port 6000
pidfile /var/run/redis_6000.pid
requirepass 123456
```

- `Slave1`节点 > `6001`

```bash
port 6001
pidfile /var/run/redis_6001.pid
slaveof 47.101.55.66 6000
requirepass 123456
masterauth 123456
```

- `Slave2`节点 > `6002`

```bash
port 6002
pidfile /var/run/redis_6002.pid
slaveof 47.101.55.66 6000
requirepass 123456
masterauth 123456
```

### 3.启动主节点和从节点

> 启动节点并且验证服务

```bash
./bin/redis-server ./redis_cluster/6000/redis.conf
./bin/redis-server ./redis_cluster/6001/redis.conf
./bin/redis-server ./redis_cluster/6002/redis.conf
ps -ef |grep redis

# 关闭的话
./bin/redis-cli -p 6000 -a 123456 shutdown
./bin/redis-cli -p 6001 -a 123456 shutdown
./bin/redis-cli -p 6002 -a 123456 shutdown

# 批量关闭
pkill -9 redis
```

### 4.验证主从复制

```bash
# 登录Redis服务
./bin/redis-cli -h 127.0.0.1 -p 6000 -a 123456
# 可以使用info查看主从关系
```

> 在主节点中插入，在从节点看一下



## Redis集群

> 参考文献：
> Redis 集群部署及踩过的坑：<http://blog.jobbole.com/113760/>
> 玩转Redis集群(上)：<https://www.jianshu.com/p/dbc62ed27f03>
> 要是您还有解决不了的，来看我这里的问题排坑。

### 启动Redis多个实例

我们在`Redis`安装目录下创建目录`redis_cluster`，并编写`6000.conf~6005.conf` 6个配置文件，这6个配置文件用来启动6个实例，后面将使用这6个实例组成集群。

以6000.conf为例，配置文件需要填写如下几项：

```bash
port  6000                                        //端口6000-6005        
bind 0.0.0.0                                     //默认ip为127.0.0.1 需要改为其他节点机器可访问的ip 否则创建集群时无法访问对应的端口，无法创建集群, 我用的阿里云ECS，就是公网IP
daemonize    yes                               //redis后台运行
pidfile  ./redis_6000.pid          //pidfile文件对应6000-6005 
cluster-enabled  yes                           //开启集群 
cluster-config-file  nodes_6000.conf   //集群的配置  配置文件首次启动自动生成 6000-6005 
cluster-node-timeout  5000                //请求超时
appendonly  yes                           //aof日志开启  有需要就开启，它会每次写操作都记录一条日志　
```

### 开启端口

由于我们集群需要用到6000-6005的端口，所以如果是ECS的话需要将这些端口打开，而且打开Redis的集群总线端口，不然会有很多问题的！

网上很多文章居然让关防火墙，也是够了。。。

> `redis集群总线端口为redis客户端端口加上10000，比如说你的redis 6379端口为客户端通讯端口，那么16379端口为集群总线端口`

### 安装Ruby

> 这一步问题也是比较多的，一定要注意

这一步可以参考本文中的`问题一`

### 创建集群

- gem 这个命令来安装 redis接口    gem是ruby的一个工具包

```bash
gem install redis
```

- 使用`redis-trib.rb`来建立集群

```bash
./src/redis-trib.rb  create  --replicas  1 47.1xx.xx.66:6000 47.1xx.xx.66:6001 47.1xx.xx.66:6002 47.1xx.xx.66:6003 47.1xx.xx.66:6004 47.1xx.xx.66:6005
```

> `--replicas  1`  表示 自动为每一个master节点分配一个slave节点    上面有6个节点，程序会按照一定规则生成 3个master（主）3个slave(从)


### 登录验证

```shell
redis-cli -c -h 47.1xx.xx.66 -p 6000 -a 123456

47.1xx.xx.66:6000> cluster nodes
04b7ed3ea70b9f3b038a4ffcaff691acaa9ceb59 47.1xx.xx.66:6005 slave 306004e971e0a66b9bbb5c9524609680eeaa81f9 0 1542379253546 6 connected
f39cc13f88ec3ce74be94817e688294320327556 47.1xx.xx.66:6003 slave d5d84a1e573c89be2ada38f30be0cecaeeddbe04 0 1542379254046 4 connected
afaa57622d52240ca7be573bc574b6096e0aa0c0 47.1xx.xx.66:6001 master - 0 1542379254547 2 connected 5461-10922
306004e971e0a66b9bbb5c9524609680eeaa81f9 47.1xx.xx.66:6002 master - 0 1542379255048 3 connected 10923-16383
7e941c80b075aee07305a58193d0e47bbc3cc5af 47.1xx.xx.66:6004 slave afaa57622d52240ca7be573bc574b6096e0aa0c0 0 1542379253546 5 connected
d5d84a1e573c89be2ada38f30be0cecaeeddbe04 172.19.91.173:6000 myself,master - 0 0 1 connected 0-5460
```

### 遇到的问题

#### 问题一:`redis requires ruby version 2.2.2`

> 参考文献：redis requires ruby version 2.2.2的解决方案：<https://www.jianshu.com/p/72443fef9554>

做`Redis`的`Cluster`集群的时候，在执行`gem install redis`时，提示如下错误：

```bash
gem install redis
    ERROR:  Error installing redis:
     redis requires Ruby version >= 2.2.2.
```

##### 1.安装RVM（具体命令可以查看官网，Ruby官网地址 和 Ruby官网安装教程）：

```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 
curl -sSL https://get.rvm.io | bash -s stable
find / -name rvm -print
source /usr/local/rvm/scripts/rvm
```

##### 2.查看rvm库中已知的ruby版本

```
[root@linux ~]# rvm list known
　　　　MRI Rubies
　　　　[ruby-]1.8.6[-p420]
　　　　[ruby-]1.8.7[-head] # security released on head
　　　　[ruby-]1.9.1[-p431]
　　　　[ruby-]1.9.2[-p330]
　　　　[ruby-]1.9.3[-p551]
　　　　[ruby-]2.0.0[-p648]
      ....
```

##### 3.安装使用一个ruby版本

```bash
rvm install 2.4.1
rvm use 2.4.1
```

##### 4.常用操作命令

```bash
# 设置默认版本
rvm use 2.4.1 --default
# 卸载一个已知版本：
rvm remove 2.3.4
# 查看ruby版本：
ruby --version
# 安装redis：
gem install redis
```

#### 问题二：create ERR Slot 9838 is already busy错误

<https://www.oschina.net/question/1031396_246716>

> `用redis-cli登录到每个节点执行flushall和cluster reset就可以了`

#### 问题三：redis集群报错Node is not empty

<https://www.jianshu.com/p/338bc2a74300>

> `1)将每个节点下aof、rdb、nodes.conf本地备份文件删除； `
> `2)172.168.63.201:7001> flushdb #清空当前数据库(可省略) `
> `3)之后再执行脚本，成功执行；`

#### 问题四：Sorry, can't connect to node

<https://blog.csdn.net/xiaobo060/article/details/80616718>

> `因为redis设置了密码`
> 需要修改`/usr/local/rvm/gems/ruby-2.4.1/gems/redis-4.0.1/lib/redis/client.rb`

```ruby
    DEFAULTS = {
      :url => lambda { ENV["REDIS_URL"] },
      :scheme => "redis",
      :host => "127.0.0.1",
      :port => 6379,
      :path => nil,
      :timeout => 5.0,
      :password => "xxxxxx",
      :db => 0,
      :driver => nil,
      :id => nil,
      :tcp_keepalive => 0,
      :reconnect_attempts => 1,
      :inherit_socket => false
    }
```

#### 问题五：redis集群部署一直卡在Waiting for the cluster to join ......

在集群的时候`redis-trib.rb  create  --replicas  1`一直在等待，网上的那些好多都不靠谱！**使用cluster meet ip port命令根本是无效的**

同时，很少有博客提到redis集群总线的内容，都是叫你关闭防火墙，实际生产中谁会这么做？最后，感慨一句，还是官方文档最有用！

> `redis集群总线端口为redis客户端端口加上10000，比如说你的redis 6379端口为客户端通讯端口，那么16379端口为集群总线端口`




## 参考文献

1. 阿里云Redis开发规范：<https://yq.aliyun.com/articles/531067?spm=a2c4e.11163080.searchblog.9.38e22ec1b4kcwD>
2. 为什么我们做分布式要用 Redis ？：<https://mp.weixin.qq.com/s/1TXICDZ_BoJ7FeAFqiZa5A>
3. 为什么要用Redis：<https://mp.weixin.qq.com/s/XU6k7T7H6TyM9kurDxoGRw>