# redis笔记

<!-- TOC -->

- [redis笔记](#redis笔记)
    - [零、redis是什么](#零redis是什么)
    - [一、redis与memcached比较](#一redis与memcached比较)
    - [二、安装](#二安装)
    - [三、配置](#三配置)
    - [四、通用key操作](#四通用key操作)
    - [五、redis中的5中数据结构](#五redis中的5中数据结构)
        - [1. 字符串（string）](#1-字符串string)
        - [2. 列表（list）链表支持 有序 可重复](#2-列表list链表支持-有序-可重复)
        - [3. 集合（set）无序 不可重复](#3-集合set无序-不可重复)
        - [4. 哈希（hash）键值对  key => value](#4-哈希hash键值对--key--value)
        - [5. 有序集合（zset）键值对  成员 => 分值 成员必须唯一](#5-有序集合zset键值对--成员--分值-成员必须唯一)
    - [六、redis事务](#六redis事务)
        - [mysql事务与redis事务比较：](#mysql事务与redis事务比较)
        - [悲观锁与乐观锁](#悲观锁与乐观锁)
    - [七、发布订阅](#七发布订阅)
    - [八、持久化](#八持久化)
        - [redis 快照rdb](#redis-快照rdb)
        - [redis 日志aof](#redis-日志aof)
    - [九、redis主从复制](#九redis主从复制)
    - [十、redis表设计](#十redis表设计)
    - [十一、面试](#十一面试)
        - [1、缓存雪崩](#1缓存雪崩)
        - [2、缓存穿透](#2缓存穿透)
        - [3、缓存与数据库读写一致](#3缓存与数据库读写一致)
    - [docker实现redis主从](#docker实现redis主从)
        - [1、命令行模式](#1命令行模式)
        - [2、docker-compose模式 推荐](#2docker-compose模式-推荐)
    - [十二、参考资料](#十二参考资料)

<!-- /TOC -->

## 零、redis是什么

redis是什么，是一种非关系型数据库，统称nosql。

## 一、redis与memcached比较

- 1、redis受益于“持久化”可以做存储(storge)，memcached只能做缓存(cache)
- 2、redis有多种数据结构，memcached只有一种类型`字符串(string)`

## 二、安装

安装最新稳定版

```sh
# 源码安装redis-4.0
# 下载
wget http://download.redis.io/releases/redis-4.0.1.tar.gz
# 解压
tar zxvf redis-4.0.1.tar.gz
cd redis-4.0.1
# 编译
make && make test && make install
错误：You need tcl 8.5 or newer in order to run the Redis test
解决：yum -y install tcl
cd utils
# 赋予运行权限
chmod +x install_server.sh
# 运行脚本
./install_server.sh
# 配置
Port           : 6379
Config file    : /etc/redis/redis.conf
Log file       : /var/log/redis.log
Data dir       : /var/lib/redis/redis
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
# 自启动
chkconfig redis_6379 on
vim /etc/redis/redis.conf
requirepass Root666,./
service redis_6379 restart
```

## 三、配置

```sh
redis-benchmark     redis性能测试工具
redis-check-aof     检查aof日志的工具
redis-check-rdb     检查rdb日志的工具
redis-cli           连接用的客户端
redis-server        服务进程

# 配置文件启动 这样会霸占终端
/usr/local/bin/redis-server /etc/redis/redis.conf
# 修改为不霸占终端
vim /etc/redis/redis.conf
daemonize no  => yes

# redis 默认有16个数据库，默认使用0号数据库，可使用select、move等命令操作
databases 16
```

## 四、通用key操作

1. keys 查询

```sh
在redis里,允许模糊查询key
有3个通配符 - ? []
-: 通配任意多个字符
?: 通配单个字符
[]: 通配括号内的某1个字符

```

1. keys 查询
2. del 删除
3. rename 重命名
4. move 移到另外一个库
5. randomkey 随机
6. exists 存在
7. type 类型
8. ttl 剩余生命周期
9. expire 设置生命周期
10. persist 永久有效
11. flushdb 清空

## 五、redis中的5中数据结构

### 1. 字符串（string）

- set
  - set name shengj -> OK
- get
  - get name -> "shengj"
- del
  - del name -> (integer) 1
  - get name -> (nil)
- mset
  - mset name shengj age 23 sex male -> OK
- mget
  - mget age sex
    ```sh
    1) "23"
    2) "male"
    ```
- setrange
  - setrange sex 2 1 将sex的第3个字符改成1 -> (integer) 4
  - get sex -> "ma1e"
- append
  - append name GG -> (integer) 8
  - get name -> "shengjGG"
- getrange
  - getrange name 1 2 -> "he"
- incr 自增
- incrby 自增一个量级
- incrbyfloat 自增一个浮点数
- decr 递减
- decrby 递减一个量级
- decrbyfloat 递减一个浮点数
- setbit 设置二进制位数
- getbit 获取二进制表示
- bitop 位操作

---

### 2. 列表（list）链表支持 有序 可重复

- rpush 右边插入
  - rpush list item1 -> (integer) 1
  - rpush list item2 -> (integer) 2
  - rpush list item3 -> (integer) 3
- lrange 列出链表值
  - lrange list 0 -1
    ```sh
    1) "item1"
    2) "item2"
    3) "item3"
    ```
- lindex
  - lindex list 1 -> "item2"
- lpop
  - lpop list -> "item1"
  - lrange list 0 -1
    ```sh
    1) "item2"
    2) "item3"
    ```
- ltrim
  - ltrim list 3 0 -> OK
  - lrange list 0 -1 -> (empty list or set)
- lpush 左边插入
- rpop 右边删除
- lrem

---

### 3. 集合（set）无序 不可重复

- sadd 增加
  - sadd set item1 -> (integer) 1
  - sadd set item2 -> (integer) 1
  - sadd set item3 -> (integer) 1
  - sadd set item1 -> (integer) 0  已存在
- smembers 所有集合元素
  - smembers set
    ```sh
    1) "item3"
    2) "item2"
    3) "item1"
    ```
- sismember 存不存在
  - sismember set item1 -> (integer) 1
  - sismember set item -> (integer) 0 不存在
- srem 移除元素
  - srem set item1 -> (integer) 1
  - smembers set
    ```sh
    1) "item3"
    2) "item2"
    ```
- spop 随机删除一个元素
- srandmember 随机获取一个元素 -> 抽奖
- scard 多少个元素
- smove 移动
- sinter 交集
- sinterstore 交集并赋值
- suion 并集
- sdiff 差集

---

### 4. 哈希（hash）键值对  key => value

- hset 设置一个
  - hset hash key1 value1 -> (integer) 1
  - hset hash key2 value2 -> (integer) 1
  - hset hash key3 value3 -> (integer) 1
  - hset hash key1 value1 -> (integer) 0 已存在
- hgetall 获取全部
  - hgetall hash
    ```sh
    1) "key1"
    2) "value1"
    3) "key2"
    4) "value2"
    5) "key3"
    6) "value3"
    ```
- hget 获取一个
  - hget hash key1 -> "value1"
- hdel 删除
  - hdel hash key1 -> (integer) 1
  - hgetall hash
    ```sh
    1) "key2"
    2) "value2"
    3) "key3"
    4) "value3"
    ```
- hmset 设置多个
- hmget 获取多个
- hlen 个数
- hexists 是否存在增长
- hinrby 增长
- hkeys 所有的key
- hvals 所有的值

---

### 5. 有序集合（zset）键值对  成员 => 分值 成员必须唯一

- zadd 增加
  - zadd zset 100 item1 -> (integer) 1
  - zadd zset 200 item2 -> (integer) 1
  - zadd zset 300 item3 -> (integer) 1
  - zadd zset 100 item1 -> (integer) 0 已存在
- zrange 按分值排序
  - zrange zset 0 -1 withscores
    ```sh
    1) "item1"
    2) "100"
    3) "item2"
    4) "200"
    5) "item3"
    6) "300"
    ```
- zrangebyscore 按分值的一部分排序
  - zrangebyscore zset 0 200 withscores
    ```sh
    1) "item1"
    2) "100"
    3) "item2"
    4) "200"
    ```
- zrem 删除
  - zrem zset item1 -> (integer) 1
  - zrange zset 0 -1 withscores
    ```sh
    1) "item2"
    2) "200"
    3) "item3"
    4) "300"
    ```
- zrank 排名升序
- zremrangebyscore 按分值删除一部分
- zremrangebyrank 按排名删除一部分
- zcard 个数

## 六、redis事务

### mysql事务与redis事务比较：

|比较|mysql|redis|
|---|---|---|
|开启|start transaction|multi|
|语句|普通sql语句|普通redis命令|
|失败|rollback|discard|
|成功|commit|exec|

如果已经成功执行了2条语句, 第3条语句出错.

rollback后,前2条的语句影响消失.

discard只是结束本次事务,前2条语句造成的影响仍然还在

### 悲观锁与乐观锁

我正在买票`ticket -1 , money -100`而票只有1张, 如果在我multi之后,和exec之前, 票被别人买了,即ticket变成0了.我该如何观察这种情景,并不再提交

悲观的想法:

    世界充满危险,肯定有人和我抢, 给 ticket上锁, 只有我能操作. [悲观锁]

乐观的想法:

    没有那么人和我抢,因此,我只需要注意,
  --有没有人更改ticket的值就可以了 [乐观锁]

Redis的事务中,启用的是乐观锁,只负责监测key没有被改动

```sh

具体的命令----  watch命令

redis 127.0.0.1:6379> watch ticket
OK
redis 127.0.0.1:6379> multi
OK
redis 127.0.0.1:6379> decr ticket
QUEUED
redis 127.0.0.1:6379> decrby money 100
QUEUED
redis 127.0.0.1:6379> exec
(nil)   // 返回nil,说明监视的ticket已经改变了,事务就取消了.
redis 127.0.0.1:6379> get ticket
"0"
redis 127.0.0.1:6379> get money
"200"

watch key1 key2  ... keyN
作用:监听key1 key2..keyN有没有变化,如果有变, 则事务取消

unwatch
作用: 取消所有watch监听

```

## 七、发布订阅

订阅端: subscribe 频道名称

发布端: publish 频道名称 发布内容

## 八、持久化

### redis 快照rdb

有限制，还是容易数据丢失，恢复快

```sh

save 900 1      # 900内,有1条写入,则产生快照 
save 300 1000   # 如果300秒内有1000次写入,则产生快照
save 60 10000  # 如果60秒内有10000次写入,则产生快照
(这3个选项都屏蔽,则rdb禁用)

stop-writes-on-bgsave-error yes  # 后台备份进程出错时,主进程停不停止写入?
rdbcompression yes    # 导出的rdb文件是否压缩
Rdbchecksum   yes   # 导入rbd恢复时数据时,要不要检验rdb的完整性
dbfilename dump.rdb  # 导出来的rdb文件名
dir ./  //rdb的放置路径

```

### redis 日志aof

```sh

appendonly no # 是否打开 aof日志功能

appendfsync always   # 每1个命令,都立即同步到aof. 安全,速度慢
appendfsync everysec # 折衷方案,每秒写1次
appendfsync no      # 写入工作交给操作系统,由操作系统判断缓冲区大小,统一写入到aof. 同步频率低,速度快,


no-appendfsync-on-rewrite  yes: # 正在导出rdb快照的过程中,要不要停止同步aof
auto-aof-rewrite-percentage 100 #aof文件大小比起上次重写时的大小,增长率100%时,重写
auto-aof-rewrite-min-size 64mb #aof文件,至少超过64M时,重写
```

    注: 在dump rdb过程中,aof如果停止同步,会不会丢失?
    答: 不会,所有的操作缓存在内存的队列里, dump完成后,统一操作.

    注: aof重写是指什么?
    答: aof重写是指把内存中的数据,逆化成命令,写入到.aof日志里.以解决 aof日志过大的问题.

    问: 如果rdb文件,和aof文件都存在,优先用谁来恢复数据?
    答: aof

    问: 2种是否可以同时用?
    答: 可以,而且推荐这么做

    问: 恢复时rdb和aof哪个恢复的快
    答: rdb快,因为其是数据的内存映射,直 接载入到内存,而aof是命令,需要逐条执行

## 九、redis主从复制

```sh
Master配置:
1:关闭rdb快照(备份工作交给slave)
2:可以开启aof

slave配置:
1: 声明slave-of
2: 配置密码[如果master有密码]
3: [某1个]slave打开 rdb快照功能
4: 配置是否只读[slave-read-only]


```

## 十、redis表设计

主键表

|列名|操作|备注|
|--|--|--|
|global:user_id|incr|全局user_id|
|global:post_id|incr|全局post_id|

---

mysql用户表

|列名|操作|备注||
|--|--|--|--|
|user_id|user_name|password|authsecret|
|1|shengj|123456|,./!@#|

redis用户表

|列名|操作|备注||
|--|--|--|--|
|user:user_id|user:user_id:*:user_name|user:user_id:*:password|user:user_id:*:authsecret|
|1|shengj|123456|,./!@#|

---

mysql发送表

|列名|操作|备注|||
|--|--|--|--|--|
|post_id|user_id|user_name|time|content|
|1|1|shengj|1370987654|测试内容|

redis发送表

|列名|操作|备注|||
|--|--|--|--|--|
|post:post_id|post:post_id:*:user_id|post:post_id:*:user_name|post:post_id:*:time|post:post_id:*:content|
|1|1|shengj|1370987654|测试内容|

---

关注表：following  -> set user_id

粉丝表：follower -> set user_id

推送表：receivepost -> list user_ids

拉取表：pullpost -> zset user_ids

## 十一、面试

### 1、缓存雪崩

问题：当我们的缓存失效或者redis挂了，那么这个时候的请求都会直接走数据库，就会给数据库造成极大的压力，导致数据库也挂了

解决：

1. 对缓存设置不同的过期时间，这样就不会导致缓存同时失效
2. 建立redis集群，保证服务的可靠性

### 2、缓存穿透

问题：当有大量用户不走我们设置的键值，就会直接走数据库，就会给数据库造成极大的压力，导致数据库也挂了

解决：

1. 参数过滤和提醒，引导用户走我们的设置的键值
2. 对不合法的参数进行空对象缓存，并设置较短的过期时间

### 3、缓存与数据库读写一致

问题：如果一直是读的话，是没问题的，但是更新操作会导致数据库已经更新了，缓存还是旧的数据

解决：

并发下解决数据库与缓存不一致的思路：将删除缓存、修改数据库、读取缓存等的操作积压到队列里边，实现串行化。

- 先删除缓存，再更新数据库

在高并发下表现不如意，在原子性被破坏时表现优异

- 先更新数据库，再删除缓存(Cache Aside Pattern设计模式)

在高并发下表现优异，在原子性被破坏时表现不如意

## docker实现redis主从

[docker实现redis主从](https://github.com/OMGZui/redis_m_s)

### 1、命令行模式

```bash
# 拉取redis
docker pull redis

# 主
docker run -v $(pwd)/master/redis.conf:/usr/local/etc/redis/redis.conf --name redis-master redis redis-server /usr/local/etc/redis/redis.conf

# 从1 --link redis-master:master master是别名
docker run -v $(pwd)/slave1/redis.conf:/usr/local/etc/redis/redis.conf --name redis-slave1 --link redis-master:master redis redis-server /usr/local/etc/redis/redis.conf

# 从2
docker run -v $(pwd)/slave2/redis.conf:/usr/local/etc/redis/redis.conf --name redis-slave2 --link redis-master:master redis redis-server /usr/local/etc/redis/redis.conf

```

### 2、docker-compose模式 推荐

```bash
# 拉取redis
docker pull redis

# 目录
├── docker-compose.yml
├── master
│   ├── Dockerfile
│   └── redis.conf
├── redis.conf
├── slave1
│   ├── Dockerfile
│   └── redis.conf
└── slave2
    ├── Dockerfile
    └── redis.conf

# 启动
docker-compose up -d master slave1 slave2

# 查看主容器
docker-compose exec master bash
root@cab5db8d544b:/data# redis-cli
127.0.0.1:6379> info Replication
# Replication
role:master
connected_slaves:2
slave0:ip=172.23.0.3,port=6379,state=online,offset=1043,lag=0
slave1:ip=172.23.0.4,port=6379,state=online,offset=1043,lag=0
master_replid:995257c6b5ac62f7908cc2c7bb770f2f17b60401
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:1043
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:1043
```

## 十二、参考资料

- [redis](https://redis.io/)
- [Docker：创建Redis集群](https://lw900925.github.io/docker/docker-redis-cluster.html)