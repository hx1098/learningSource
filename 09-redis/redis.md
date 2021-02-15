[TOC]

##               目录结构

##       一、单机版

##### 1.安装

```bash
1，yum install wget
2,cd ~
3,mkdir soft
4,cd soft
5,wget    http://download.redis.io/releases/redis-5.0.5.tar.gz
6,tar xf    redis...tar.gz
7,cd redis-src
8,看README.md
9, make
#如果是首次的centos需要安装gcc
 ....yum install  gcc 
#如果没安装gcc先执行了make, 先要进行一次清理缓存 
 ....  make distclean
10,make
11,cd src   ....生成了可执行程序
12, cd ..
13,make install PREFIX=/opt/mashibing/redis5
14,vi /etc/profile
...   export  REDIS_HOME=/opt/mashibing/redis5
...   export PATH=$PATH:$REDIS_HOME/bin
..source /etc/profile
15,cd utils
16,./install_server.sh  （可以执行一次或多次）
    a)  一个物理机中可以有多个redis实例（进程），通过port区分
    b)  可执行程序就一份在目录，但是内存中未来的多个实例需要各自的配置文件，持久化目录等资源
    c)  service   redis_6379  start/stop/stauts     实际上是执行的这个脚本   /etc/init.d/redis_6379
    d)  脚本还会帮你启动！
17,ps -fe |  grep redis  
```

##### 2.连接

```bash
redis-cli
redis-cli -h
redis-cli -p 6380
set k380:1 hello
#使用8号库
select 8 
#指定8号库,一共16个数据库 
redis-cli -p 6380 -n 8


help @generic

```

##### 3.字符串操作

![image-20210205161128102](image\image-20210205161128102.png)

```bash
#nx: 如果k1不存在就设置为ooxx,反之xx: k1存在时候才能更新...
set k1 ooxx  nx

```

![image-20210205161716847](image\image-20210205161716847.png)



```bash
mset k3 a k4 b
mget k3 k4
```

![image-20210205162057578](image\image-20210205162057578.png)





```bash
help @String
#追加
APPEND k1 wrold
#追加空格
APPEND k1 " wrold" 
#取指定长度,
GETRANGE k1 10 16
#正反向索引
GETRANGE k1 10 -1
GETRANGE k1 0 -1
SETRANGE k1 10 hello
#获取长度
STRLEN k1
```

![image-20210205163047941](image\image-20210205162057579.png)



##### 4.数值类型操作

```bash
#value类型
type key
OBJECT help
OBJECT encoding k1
OBJECT encoding k2
#增加
INCR k1
INCRBY k1 11
#减法
DECR k1
DECRBY k1 21
#float数值
INCRBYFLOAT k1 0.3
#由此可见, string类型数据在这些特定情况下是int, 而且可以做计算


set  k3 jjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjjj
OBJECT encoding k3
APPEND k3 jjjjjjjjjjjjjjjj
OBJECT encoding k3
#可看到数据类型可改变,由embstr 到 raw 类型
```

![image-20210205170917714](image\image-20210205170917714.png)





```bash
 STRLEN k1
 set k2 9
 OBJECT encoding k2
 APPEND k2 999
 get k2
 OBJECT encoding k2
 INCR k2
 OBJECT encoding k2
 STRLEN k2
 #由此可以看到有些命令可以使其变成int, 有些变成raw
 
 help  @string
 #此命令是获取值并修改值, 可减少网络io
 GETSET k1 niubiren
```

![image-20210205171622007](image\image-20210205171622007.png)



```bash
#这个谨慎操作哦,清空数据库, 我这里仅仅是演示操作
FLUSHDB
MSETNX k1 a k2 b
MGET k1 k2
MSETNX k2 d k3 e
mget k1 k2 k3
#最后一个的值是空的, 并且k2的值也没有改变,没有放进去, 属于原子操作了
```



##### 5.二进制安全

```bash
get k3
set k3 a
STRLEN k3
APPEND k3 中
get k3
STRLEN k3
#可看到长度变成了4
```

```
key 
```

![image-20210205175849641](image\image-20210205175849641.png)





##### 6. bitmap

###### 1.bitmap基本操作

```bash
#一个字节有八个二进制位
help setbit
setbit k1 1 1
#占一个字节
STRLEN k1
#
get k1
SETBIT k1 7 1
get k1
man ascii

setbit k1 9 1
STRLEN k1 #此时长度为2
get k1    #此时显示A@
```

![image-20210206212946254](image\image-20210206212946254.png)

![image-20210206231748041](image\image-20210206231748041.png)

```
bitpos k1 1 0 0
bitpos k1 1 1 1
bitpos k1 1 0 1

BITCOUNT k1 0 1
BITCOUNT k1 0 0 
BITCOUNT k1 1 1

bitop 

```

###### 2.bitmap的使用场景

1. 如果公司有用户， 统计用户的登录天数， 且窗口随机.

   ```
   setbit sean 1 1
   setbit sean 7 1
   setbit sean 364 1
   strlen 	sean 
   bitcount sean -2 -1
   ```

2. 加入某东就是自己的,某18做活动, 凡是登录的就送礼物,大库备货多少礼物,假设有2E用户.

   ```bash
   分析:
   僵尸用户
   忠诚用户／冷热用户
   其实就是：活跃用户的统计,
   setbit 20190101 1 1  #第一天1号用户登录
   setbit 20190102 1 1  #第二天1号用户登录
   setbit 20190102 7 1  #第二天7号用户登录
   #统计用多少用户
   bitop or destkey 20190101 20190102
   BITCOUNT destkey 0 -1
   
   ```

   

   

##### 7.list的使用

key还是为一个string类型的数据, value为list的数据

1.LPUSH的使用

```bash
#从左边推进去几个数据
LPUSH k1 a b c d e f g
#从右边推进去几个数据
RPUSH k1 a b c d e f g
127.0.0.1:6379> LPOP k1
"g"
127.0.0.1:6379> LPOP k1
"f"
127.0.0.1:6379> RPOP k2
"g"
127.0.0.1:6379> RPOP k1
"a"
#可以看到, 一个有趣的事情, 向左压入的数据, 可以先出也可以后出来, 类似一个压栈出栈的操作.

127.0.0.1:6379> LPUSH k1 a b c d e f g
(integer) 7
127.0.0.1:6379> RPUSH k2 a b c d e f g
(integer) 7
127.0.0.1:6379> LRANGE k1 0 -1
1) "g"
2) "f"
3) "e"
4) "d"
5) "c"
6) "b"
7) "a"
127.0.0.1:6379> LRANGE k2 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
7) "g"
127.0.0.1:6379> 
#可以看出, LPUSH是向左边累积数据的, RPUSH是向右累积数据的,数据刚好相反
```

2.LINDEX

```bash
LRANGE k1 0 -1
#取出来数据"e"
LINDEX k1 2
#取出来最后一个
LINDEX k1 -1
#将"c"修改为"666"
LSET k1 4 666
LRANGE k1 0 -1
#类似数组的结构

help @list
#list的数据的移出
127.0.0.1:6379> LRANGE k3 0 -1
 1) "d"
 2) "6"
 3) "a"
 4) "5"
 5) "c"
 6) "4"
 7) "a"
 8) "3"
 9) "b"
10) "2"
11) "a"
12) "1"
127.0.0.1:6379> LREM k3 2 a
(integer) 2
127.0.0.1:6379> LRANGE k3 0 -1
 1) "d"
 2) "6"
 3) "5"
 4) "c"
 5) "4"
 6) "3"
 7) "b"
 8) "2"
 9) "a"
10) "1"
127.0.0.1:6379> 

#在"6"的后面插入一个"a"
LINSERT k3 after 6 a
lrange k3 0 -1
```

3.阻塞的单波订阅

```bash
blpop 
```

4.ltrim

```bash
lpush k4 a b c d e g 
ltrim k4 0 -1 #删除两端的数据(0 -1 是不会删除数据的)
```



##### 8.数据类型hash的使用

  hash 类似 JAVA中的HASHMAP

```bash
help @hashs
#hash设置name
hset sean name zzl
hmset sean age 18 address bijing
hget sean age
hmget sean naem address
#获取全部的
HGETALL sean

#年龄运算,可以进行数值计算, 可以做点赞, 收藏, 详情页
HINCRBYFLOAT sean age 0.5
HGET sean age
HINCRBYFLOAT sean age -1
HGET sean age
```



##### 9.数据类型set的使用

SET不维护排序, 可以去重,和list相比, list存入时候是有序的, set是无需的, 去重复

1.简单的增删改查

```bash
#查看帮助
help @set
#刷新数据库, 演示使用, 生产绝对不能使用
FLUSHDB
#添加数据
SADD k1 tom sean peter ooxx xxoo tom
#查看当前的数据, 生产绝对不能使用(影响redis的吞吐量)
SMEMBERS k1
#移除掉两个数据
SREM k1 ooxx xxoo
SMEMBERS k1



```



2.求交差集

```bash
#求交集
SADD k2 1 2 3 4 5 6 7 8 9
sadd k3 4 5 6 7 8 9
#直接求出k2 k3的交集, 不转存到新的容器
SINTER k2 k3
#求出k2 k3的交集, 转存到新的容器temp
SINTERSTORE temp k2 k3
SMEMBERS temp

#求并集
SUNION k2 k3
#求差集, 求谁的差集一定要搞明白, 位置不对是不一样的.
SDIFF k2 k3
SDIFF k3 k2

```



3.集合操作(用的相当多)

1.用途:　用户抽奖（中奖是否重复），家庭争斗

```bash
#刷新数据库, 演示使用, 生产绝对不能使用
FLUSHALL 
SADD k1 tom ooxx xxoo xoxo oxox xoox oxxo
#抽出三个用户,去重复,但是不会删除该数据,下次抽奖还会出现
SRANDMEMBER k1 3
#抽出三个用户,不去重复,但是不会删除该数据,下次抽奖还会出现
SRANDMEMBER k1 -3 

#该抽奖方式抽出后会将改数据弹出
spop k1

```

##### 10.有序集sorted_set

 特点:　有排序，set相比, sorted_set插入的数据是有顺序的.

 1.简单的增删改查

```bash
FLUSHALL
#增加
zadd k1 8 apple 2 banana 3 orange
#查看数据, 默认不带分值
ZRANGE k1 0 -1
#查看数据, 带上分值
ZRANGE k1 0 -1 withscores

#取出分值3到8的数据
ZRANGEBYSCORE k1 3 8
#由低到高取出前两名
ZRANGE k1 0 1
#取出后两名的
ZREVRANGE k1 0 1
ZRANGE k1 -2 -1

#取出分值
ZSCORE k1 apple
#取出分值
ZRANK k1 apple

#分值计算
ZRANGE k1 0 -1 withscores
#banana的分值增加2.5
ZINCRBY k1 2.5 banana
ZRANGE k1 0 -1 withscores
```



2.权重聚合计算

```bash
FLUSHALL
ZADD k1 80 tom 60 sean 70 baby
zadd k2 60 tom 100 sean 40 yiming
#求并集,有相同的会加起来
ZUNIONSTORE untemp 2 k1 k2
ZRANGE untemp 0 -1 withscores
#求并集的同时增加权沪权重
ZUNIONSTORE untemp1 2 k1 k2 weights 1 0.5 
ZRANGE untemp1 0 -1 withscores
#取两个相同人的的最大值, 不同取自己本身的值 
ZUNIONSTORE untemp2 2 k1 k2 aggregate max
ZRANGE untemp2 0 -1 withscores

#更多的可以看帮助文档了.
help @zset
```



3.思考: zset是如何是实现高效排序的? 增删改查的速度快不快？

1.底层使用了skip list的结构（跳表）

![image-20210208213735761](image\image-20210208213735761.png)





##### 11.管道pipline的使用

有关redis的使用都在中文网站上:  [redis.cn]()

1.说白了就是redis为了提高效率, 将多个redis命令的请求压缩成一个请求, 来批量执行命令

```bash
#开一个窗口
nc localhost 6379
keys *
set k1 hello

#开另外一个窗口,并且连上端口为6379的redis
redis-cli -p 6379 
keys *
#综上: 可以看到只要监听一个socket, 往里面塞数据, 就可以进行数据的读写操作了.
```



```bash
#演示管道   \n 表示换行, 但是windows中好像是\r\n
echo -e "sfafs\nsjiodfj"
#设置k2=hello, 并且将k2的值加一,  获取k2的值, 返回的值有个"$3" 代表的是宽度(跟aof有联系)
echo -e "set k2 99\nincr k2\n get k2"|  nc localhost 6379
#此外还支持大量数据的初始化导入功能
http://redis.cn/topics/mass-insert.html
```



##### 12.redis的发布订阅功能

就不从官网粘贴描述, 字面意思, 如果你想看小姐姐的直播那啥的, 我下次还让给我推送新的消息的话,咳咳就是这个功能.

```bash
#查看一下帮助文档
help @pubsub
#开辟一个ooxx的频道, 发送hello
PUBLISH ooxx "hello world"
#另外开一个窗口, 进行订阅改频道
SUBSCRIBE ooxx

#在第一个窗口进行发送数据: 
PUBLISH ooxx "hello meinv"
#在第二个窗口可以看到发送的数据了
#综上 :  订阅这必须提前监听该频道, 才能正确接受到信息, 否则是不会接受到消息的
```

思考:　在微信中或者某课堂中, 用户自己可以收到群里的消息, 或者课堂的推送记录; 还可以看到历史的消息, 从最新的到过往的消息是存储到哪里呢? 关系型数据库　ＯＲ　redis?  如下图

一个client发布者pub, 两个订阅者sub, 一个订阅者是用户另一端, 一个订阅者通过redis扔到kafka中, 最终持久化到db中

![image-20210208231846657](image\image-20210208231846657.png)





##### 13.redis的事物(transactions)

1.开启事物

```bash
#@transactions, 类似oracle一样, 更新数据后需要手动执行下
help  @transactions
#开启事物
MULTI
#执行事物
EXEC

#窗口1
MULTI
set k1 hello
EXEC
get k1

#窗口2
MULTI
get k1
exec
DEL k1

#如果先执行了窗口2的del, 则窗口1的最后一个 get k1 就会获取到一个空值了!
#无非就是谁先执行的问题
```



2.watch

```bash
set k1 "29384"

#窗口1
127.0.0.1:6379> WATCH k1  #开启监控
OK
127.0.0.1:6379> MULTI  #开启事物
OK
127.0.0.1:6379> get k1 
QUEUED
127.0.0.1:6379> keys *
QUEUED
127.0.0.1:6379> exec  #比窗口2执行的晚
(nil)
127.0.0.1:6379> 

#窗口2
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> del k1
QUEUED
127.0.0.1:6379> set k1 "8888"
QUEUED
127.0.0.1:6379> exec  #比窗口1执行的早
1) "29384"
2) (integer) 1
3) OK
127.0.0.1:6379> get k1
"8888"
127.0.0.1:6379> 

```

#可以看到:　窗口１的事物没有执行，应为k1的值被更改了.要不要watch, 看自己的场景.



##### 14.布隆过滤器

```bash
#安装布隆过滤器插件
wget https://github.com/RedisBloom/RedisBloom/archive/master.zip
unzip master.zip 
cd RedisBloom-master/
make
cp redisbloom.so  /opt/mashibing/redis5/
service redis_6379 stop
redis-server --loadmodule /opt/mashibing/redis5/redisbloom.so

redis-cli 
#测试命令,
BF.ADD k1 abc
#返回1
BF.EXISTS k1 abc
#返回0
BF.EXISTS k1 ad

```

1.作为缓存和数据库的区别

1.缓存:　数据不重要，不是全量的数据，应该随着访问量变化（热数据），如何根据业务进行变化,只保留热数据,因为内存的瓶颈.

```bash
set k1 aaa ex 20
get k1
#获取当前的键的过期时间
ttl k1

EXPIRE k1 50
ttl k1
set k1 bbb
ttl k1
#发生写的操作后会清除时间限制

#定时
EXPIREAT
time
EXPIREAT k1 1613094731
ttl k1
```

过期策略:

1. 主动 周期轮询判定
2. 被动 主动访问时候判定



##### 14.持久化RDB 和 fork

http://redis.cn/topics/persistence.html

###### **RDB的优点**

- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.

- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.

- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.

- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

  ###### **RDB的缺点**

- 如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.

- RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.

  



1.管道

```bash
num=0
#num可以增加一
((num++))
echo $num
#nun 不会增加1
((numm++)) | echo ok
echo $num

#可看到子进程是不同的.
echo $BASHPID | more
echo $BASHPID | more
## $$高于管道
```



```bash
num=0
echo $num
yum install psmisc
echo $$
/bin/bash
echo $$
pstree
echo $num
#可以看到是两个bash, 最后一个echo $num 获取不到值, 进程之间是隔离的.

#定义为全局的
export num
/bin/bash
echo $num
#此时是可以获取到值的.

父进程可以让子进程看到值的.
```

![image-20210211152202169](image\image-20210211152202169.png)



```bash
#创建一个文件,
vi test.sh

#!/bin/bash
echo $$
echo $num
num=999
echo num:$num
sleep 30
echo $num

#更改为可执行文件
chomd +x test.sh
#执行test.sh, &是后台执行的意思
./test.sh & 

#在linux当中,可以看到子进程不会影响到主进程的值
```

![image-20210211154045507](image\image-20210211154045507.png)



```bash
#再次执行这个test.sh
./test.sh &
echo $$
num=88
echo $num
#等待30秒, 可以看到
#num打印出来的999, 没有影响到子进程的值

#父进程的修改不会影响到也不会破坏子进程
```

思考:　创建子进程的速度应该是什么程度？

如果父进程是redis,内存数据是10G,考虑的问题就是:速度问题和内存空间够不够的问题,这几引入了linux中的fork,计算机玩的就是fork, 速度快,占用空间小



##### 15.持久化AOF (append追加)

 写操作记录到文件当中, 丢失数据较少,rdb和aof可以同时开启, <font color=red>如果开启了aof,只会使用aof来恢复数据</font>, 4.0以后包含RDB的全量,增加记录新的写操作.

```bash
#我的文件可以不台一样， 你们的可能是redis.conf
vim /etc/redis/6379.conf

#阻塞执行
137 daemonize yes
    #日志文件注释掉
172 #  logfile /var/log/redis_6379.log
700 appendonly yes
#关闭混合模式
807 aof-use-rdb-preamble no

:wq! #保存
cd  /var/lib/redis/6379
rm -rf *
kill -9 redis
redis-server /etc/redis/6379.conf

#新开第二个窗口,额可以看到这个文件还没有, appendonly.aof
cd  /var/lib/redis/6379
vi appendonly.aof

#再次新开一个窗口, 链接商redis
redis-cli
set k1 hello

#打开第二个窗口, 
vi appendonly.aof
#可以看到这个文件里面已经有数据了

#*:代表有几个元素，　
#＄:代表有几个字符

#在第三个窗口敲击如下命令
BGSAVE

cd  /var/lib/redis/6379
ll
#可以看到多一个rdb文件

vim dump.rdb
#检查文件是否有报错
redis-check-rdb dump.rdb
```

![image-20210213141214683](image\image-20210213141214683.png)



![image-20210213141821971](image\image-20210213141821971.png)



```bash
set k1 234
set k1 489574398
set ninini
#写了折磨多以后, aof文件会有重复的日志,

#老版本
BGREWRITEAOF
#执行完以后, 再次去看第二个窗口的文件, 
vi appendonly.aof
#可以看到文件已经少了很多.
方便执行有意义的命令, 加快恢复速度
```

![image-20210213142448873](image\image-20210213142448873.png)





2.新版本的aof规则:

```bash
#接着上面的, 删除第二窗口, 退出第三个窗口的客户端
rm -rf *

#更改redis的配置文件
vim /etc/redis/6379_conf
#将807行的这个更改为混合模式
807 aof-use-rdb-preamble yes
:wq!


#第一个窗口,重新加载redis的配置,并重启redis
kill redis
redis-server /etc/redis/6379.conf
#第二个窗口,  appendonly.aof,没有东西显示
vi appendonly.aof
#第三个窗口
redis-cli
set k1 11
set k1 22
set k3 33

#第二个窗口,  appendonly.aof,有显示命令了, 开头不适宜redis开头的
vi appendonly.aof

#第三个窗口
BGREWRITEAOF

#第二个窗口,  appendonly.aof,有显示命令了, 开头是以redis开头的
vi appendonly.aof

#第三个窗口
set k1 ww
set k2 mm

#第二个窗口,  appendonly.aof文件中会以明文的方式显示出来
vi appendonly.aof

#两个命令执行后, 会在, /etc/redis/6379下生成两个文件rdb和aof的文件
BGSAVE
BGREWRITEAOF
```

![image-20210213151559987](image\image-20210213151559987.png)





## 二、集群版知识点 

##### 1.AKF原理

参考: https://www.cnblogs.com/-wenli/p/13584796.html

传统的单机redis存在问题:

- 单点故障

- 容量有限(一个redis顶多10个G,就到头了)

- 压力(cpu压力, IO压力)

  而AKF就是从三个维度来控制redis, 进行拆分,具体为:

  1. X 轴：直接水平复制应用进程来扩展系统。
  2. Y 轴：将功能拆分出来扩展系统。
  3. Z 轴：基于用户信息扩展系统。

  

  ![image-20210213164849743](image\image-20210213164849743.png)

而拆分之后面临数据一致性的问题, 有下面三种解决方案, 每一种有其优缺点.

![image-20210213165000341](image\image-20210213165000341.png)



##### 2.主从复制原理

 接下来我们在单机的centos中实现一个伪分布式, 神魔意思呢? 就是在在一个centos系统中来进行多个不同端口的redis, 来模拟业务场景, 话不多说, 干饭人!

```bash
#先玩个三台的吧! 安装一下redis, 不知道如何安装的,开篇第一大节的就是安装的, 
#端口分别是: 6379 6380 6381
cd /root/soft/redis-5.0.8/utils
./install_server.sh
#选好端口, 一路next就行,不多说了
```

![image-20210214101736137](image\image-20210214101736137.png)



```bash
#在家目录下新建立一个文件夹
mkdir test
cd test
#拷贝配置文件到当前目录.
cp /etc/redis/*  ./

vi 6379.conf
#更改一下配置
daemonize no  #后台运行关掉, 默认是yes
pidfile /var/run/redis_6379.pid #pid文件默认
logfile /var/log/redis_6379.log #开启的, 现在注释掉
appendonly no #关闭aof日志 172行,默认是no,日志文件注释掉,将日志不在后台打印, 直接输出到屏幕当中

vim 6380.conf
daemonize no
# logfile /var/log/redis_6380.log 关闭此行(因为我上面做过操作, 所以这里不太确定,重新查看一下)
appendonly no #aof默认就是, 不用管了.

vim 6381.conf
daemonize no
# logfile /var/log/redis_6381.log
#OK, 万事俱备!
#配置的目的:  不让redis产生aof日志, 直接让redis在前台阻塞运行

#删除掉所用的文件, 不要怕, 里面是redis的dump文件
cd /var/lib/redis/
rm -rf ./*
mkdir 6379 6380 6381

#回到回到家目录下的test
ps -ef|grep redis
#杀掉所有的redis进程, auto的就不用了
kill -9 110
#开三个窗口, 指定新的配置文件重新启动
redis-server  ~/test/6379.conf
redis-server  ~/test/6380.conf
redis-server  ~/test/6381.conf
```

![image-20210214111244572](image\image-20210214111244572.png)



```bash
#再开三个窗口
redis-cli -p 6379
redis-cli -p 6380
redis-cli -p 6381
#分别查看数据, 都为空的.
keys *

cd /var/lib/redis
ls -h /6379 /6380 /6381
#分别查看里面子文件夹的数据, 是否为空, 不为空的话一定是你没清理.
```

配置:  6379为主库, 其他两个为从库(就这末愉快的决定吧!)

如何配置呢?

使用命令: help slaveof

redis4.0以前是: 	SLAVEOF

redis5.0以后是:    REPLICAOF

```bash
#在6380的端口redis执行
REPLICAOF 127.0.0.1 6379
#可以看到6379的日志, 和6380的日志, 
#不同的是6380会flushdb, 自身会清空后再同步6379
#此时新开两个窗口, 分别链接6379,6380, 
set k1 hello
#在6380窗口获取数据,
get k1
#可以获取数据了, 但是从库只能读, 不能写, 后面在用配置文件调节.

#在6381的端口执行同步操作
REPLICAOF 127.0.0.1 6379
 keys *
```

<img src="image\image-20210214142301885.png" alt="image-20210214142301885" style="zoom:67%;" />



![image-20210214144510335](image\image-20210214144510335.png)

<strong>思考1:  </strong>如果程序正在运行当中, 突然我的6381挂了, 这其中我的6379和6380两个数据库来了一波又一波的数据,  然后6381通过紧急修复好了, 这中间是先给他全部删除还是追加呢?

操作: 

先把6381给挂掉, 往到6379中增加几条数据, 查看情况

```bash
#随意增加数据
set k2 2342
set k3 2342
set k4 2342

#再次启动6381, 并追随6379
redis-server ~/test/6381.conf  --replicaof 127.0.0.1 6379
#可以看到并没有flushdb的操作, 可见是追加的, 而且自身也可看到新增加的数据
```

![image-20210214150557925](image\image-20210214150557925.png)



<strong>思考2:  </strong>如果我主数据库不想当大哥了, 我想让6381或者其他人当大哥, 如何操作?

```bash
#在6380中操作,不追随6379
REPLICAOF no one
REPLICAOF 127.0.0.1 6380
```

不过这种是人工的方式进程主从切换的,在生产环境中就不合适了.

##### 3.Redis高可用(Sentinel)哨兵

中文地址发出来, 就不去里面复制粘贴了: http://redis.cn/topics/sentinel.html

- **监控（Monitoring**）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

话不多说了, 开始配置吧

```bash
#回到redis的配置文件目录
cd ~/test
vi 26379.conf
#写入一下配置,并保存, 按照最简单的开始写, 一套哨兵可以监控多套集群
port 26379
#监控, mymaster哨兵名称, 网络地址, 端口号, 2是投票的权重.
sentinel monitor mymaster 127.0.0.1 6379 2
```

复制两个出来.

```bash
cp 26379.conf 26380.conf
cp 26379.conf 26381.conf
#修改复制出来的文件中的端口号,改为和名字一样的端口号
vi 26381.conf
vi 26380.conf

```

![image-20210214162727148](image\image-20210214162727148.png)

开始启动各个redis服务

```bash
#第一个启动
redis-server ./6379.conf
#第二个启动, 并追随6379
redis-server ./6380.conf  --replicaof 127.0.0.1  6379
#第三个启动, 并追随6379
redis-server ./6381.conf  --replicaof 127.0.0.1  6379
```

启动哨兵

```bash
#进入到第一节编译的目录, 查看该目录下是否有一个redis-sentinel文件
cd /root/soft/redis-5.0.8/src
#这个安装目录中是没有redis-sentinel这个文件的, 可以看到我的这个文件已经软连接到了redis-server
cd /opt/mashibing/redis5/bin/

#启动
#新开一个窗口, 启动主服务的哨兵
redis-server ./26379.conf  --sentinel
#新开一个窗口, 启动
redis-server ./26380.conf --sentinel
#新开一个窗口, 启动
redis-server ./26381.conf --sentinel

#每一个里面都有别人的哨兵.
```

![image-20210214181150495](image\image-20210214181150495.png)

![image-20210214181624480](image\image-20210214181624480.png)

![image-20210214181719224](image\image-20210214181719224.png)



现在可以看到, 6379是主, 6380和6381是从,  现在搞一个破坏, 破坏掉6379的redis, 直接停掉它, 看看有什么反应.

```bash
12085:X 14 Feb 2021 05:23:21.521 # +sdown master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:21.579 # +odown master mymaster 127.0.0.1 6379 #quorum 3/2
12085:X 14 Feb 2021 05:23:21.579 # +new-epoch 1
12085:X 14 Feb 2021 05:23:21.579 # +try-failover master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:21.580 # +vote-for-leader 97e0996c78bf68c28f36af4d4c52a17c7ae1ff78 1
12085:X 14 Feb 2021 05:23:21.581 # 5f6d4309dfe41c5640bb66439e9e0e14ebb98826 voted for 5f6d4309dfe41c5640bb66439e
9e0e14ebb98826 112085:X 14 Feb 2021 05:23:21.583 # 3bfc6d6484c3c875994fee3edafa31aea9ec1e3e voted for 97e0996c78bf68c28f36af4d4c
52a17c7ae1ff78 112085:X 14 Feb 2021 05:23:21.653 # +elected-leader master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:21.653 # +failover-state-select-slave master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:21.737 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:21.737 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ myma
ster 127.0.0.1 637912085:X 14 Feb 2021 05:23:21.793 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster
 127.0.0.1 637912085:X 14 Feb 2021 05:23:22.541 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:22.541 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:22.599 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6
37912085:X 14 Feb 2021 05:23:23.631 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1
 637912085:X 14 Feb 2021 05:23:23.631 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6
37912085:X 14 Feb 2021 05:23:23.715 # -odown master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:23.715 # +failover-end master mymaster 127.0.0.1 6379
12085:X 14 Feb 2021 05:23:23.715 # +switch-master mymaster 127.0.0.1 6379 127.0.0.1 6380
12085:X 14 Feb 2021 05:23:23.715 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
12085:X 14 Feb 2021 05:23:23.715 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6380
12085:X 14 Feb 2021 05:23:53.742 # +sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6380
12085:X 14 Feb 2021 05:25:52.473 # -sdown slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 6380
12085:X 14 Feb 2021 05:26:02.453 * +convert-to-slave slave 127.0.0.1:6379 127.0.0.1 6379 @ mymaster 127.0.0.1 63
80
```

可以看到 mymaster 127.0.0.1 6380 切换到了6380, 测试下数据, 完全ojbk, 那抹, redis这些哨兵是如何通讯的呢?

```bash
#查看订阅发现: 他们是一直在订阅一直在通讯

127.0.0.1:6380> PSUBSCRIBE *
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "*"
3) (integer) 1
1) "pmessage"
2) "*"
3) "__sentinel__:hello"
4) "127.0.0.1,26380,97e0996c78bf68c28f36af4d4c52a17c7ae1ff78,1,mymaster,127.0.0.1,6380,1"
1) "pmessage"
2) "*"
3) "__sentinel__:hello"
4) "127.0.0.1,26381,3bfc6d6484c3c875994fee3edafa31aea9ec1e3e,1,mymaster,127.0.0.1,6380,1"
1) "pmessage"
2) "*"
3) "__sentinel__:hello"
4) "127.0.0.1,26379,5f6d4309dfe41c5640bb66439e9e0e14ebb98826,1,mymaster,127.0.0.1,6380,1"
1) "pmessage"
2) "*"
3) "__sentinel__:hello"
4) "127.0.0.1,26380,97e0996c78bf68c28f36af4d4c52a17c7ae1ff78,1,mymaster,127.0.0.1,6380,1"
```



##### 4.redis集群代理

摘自： https://blog.csdn.net/rebaic/article/details/76384028

无论是为了解决redis的高可用问题、还是为了可扩展性、或者是为了维护方便，用一款redis代理都是上佳的选择。在github上有众多开源的redis代理，如下：

- [predixy](https://github.com/joyieldInc/predixy)
- [twemproxy](https://github.com/twitter/twemproxy)
- [codis](https://github.com/CodisLabs/codis/)
- [redis-cerberus](https://github.com/projecteru/redis-cerberus)

###### 1.twemproxy代理

这里我就直接演示下使用twitter公司的twemproxy吧。

```bash
#连上你的虚拟机
cd soft
mkdir twemprox

#没安装git的话先安装。
yum install -y git
git version
yum update nss
git clone https://github.com/twitter/twemproxy.git

#开始编译
cd twemproxy
#但是这个时候看一下README, 你会发现， 正常的都是直接make的， twitter他丫的下面有一个快速清单, 需要提前安装点东西,
```

- To build twemproxy from source with *debug logs enabled* and *assertions enabled*:

  ```
  $ git clone git@github.com:twitter/twemproxy.git
  $ cd twemproxy
  $ autoreconf -fvi
  $ ./configure --enable-debug=full
  $ make
  $ src/nutcracker -h
  ```

  A quick checklist:

  - Use newer version of gcc (older version of gcc has problems)
  - Use CFLAGS="-O1" ./configure && make
  - Use CFLAGS="-O3 -fno-strict-aliasing" ./configure && make
  - `autoreconf -fvi && ./configure` needs `automake` and `libtool` to be installed

大概意思就是执行`autoreconf  和configure`之前要安装  `automake` `libtool` 的插件, 那我们就直接安装一下吧!

```bash
yum install automake libtool -y
#如果还报错, 就执行,看好自己的系统
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

#执行完多一个configure文件
autoreconf -fvi
./configure --enable-debug=full
make
#在src下有一个可执行程序,nutcracker
cd src 
cd ../scripts
cp nutcracker.init /etc/init.d/twemproxy
cd /etc/init.d
#有时候, 变绿也是一件好事哦
chmod +x twemproxy

#新开一个窗口，拷贝配置文件
mkdir /etc/nutcracker/
cd ~/soft/twemproxy/conf
cp ./*  /etc/nutcracker/
cd /etc/nutcracker/

#回到第一个窗口, 将nutcracker作为一个命令来使用
cd /root/soft/twemproxy/src
cp nutcracker  /usr/bin/

#修改配置文件准备启动
cd /etc/nutcracker/
#备份一下
cp nutcracker.yml  nutcracker.yml.bak 
vim nutcracker.yml
```

```bash
#文件中就这个不用删除就行, 其他的暂时不配置
#fnv1a_64 hssh算法
#22121 是你要代理的redis端口, 后期可以直接链接这个端口就能往里面塞数据
alpha:
  listen: 127.0.0.1:22121
  hash: fnv1a_64
  distribution: ketama
  auto_eject_hosts: true
  redis: true
  server_retry_timeout: 2000
  server_failure_limit: 1
  servers:
   - 127.0.0.1:6379:1
   - 127.0.0.1:6380:1

```

开始启动redis

```bash
#新开一个窗口
cd ~
mkdir data
cd data
mkdir 6379
mkdir 6380
#查看自己的redis是否已经启动了, 如果启动了,我这里是使用命令stop不管用, 索性直接kill -9 来关掉的
ps -ef|grep redis
kill -9 930
#或者这样关闭
redis-cli 6379 shutdown 

#终于启动了
cd /6379
redis-server --port 6379

cd /6380
redis-server --port 6380

#启动代理
service twemproxy start
#新开一个窗口,连接上redis
redis-cli -p 22121

set k2 hello
set k1 1231
set k4 34985
set 1 hello

#开一个新窗口
redis-cli -p 6379
keys *

#开一个新窗口
redis-cli -p 6380
keys *

#可以看到, 数据是不一致的. 这是因为默认的算法造成的,这时候代理模式已经生效了.
#那些命令不支持呢? keys *   watch  mulit

#关闭: 没办法, 电脑配置差, 关闭吧还是
service twemproxy stop
```



###### 2.pridixy代理

pridixy编译不做介绍了, 这个需要c++11的规范, (偷偷告诉你,centos6.5是不行滴!), 我们直接从github拿到直接编译好的就行了

地址：

https://github.com/joyieldInc/predixy

https://github.com/joyieldInc/predixy/releases/download/1.0.5/predixy-1.0.5-bin-amd64-linux.tar.gz　

```bash
cd ~/soft
mkdir predixy
tar xf predixy-1.0.5-bin-amd64-linux.tar.gz
cd predixy-1.0.5/bin
ll -h
#此时可以看到一个14M的程序。


```

需要配置的下项目：

```bash
#更改配置文件
vim ../conf/predixy.conf
#放开
Bind 127.0.0.1:7617
#放开sentinel模式， 注释掉try的85行数据， 导入要给哨兵的配置。
84  Include sentinel.conf
85  #Include try.conf
```

更改哨兵的配置文件

```bash
vim sentinel.conf
#鼠标定位到## Examples:下的 SentinelServerPool 的位置
#在vi命令行模式下输入
:.,$y
#回车后复制出来了
#按下： Gnp 就粘贴出来了。

#将“#”替换为“”
:.,$s/#//
#回车就全部解锁了
#最终效果

SentinelServerPool {
    Databases 16
    Hash crc16
    HashTag "{}"
    Distribution modula
    MasterReadPriority 60
    StaticSlaveReadPriority 50
    DynamicSlaveReadPriority 50
    RefreshInterval 1
    ServerTimeout 1
    ServerFailureLimit 10
    ServerRetryTimeout 1
    KeepAlive 120
    Sentinels {
        + 127.0.0.1:26379
        + 127.0.0.1:26380
        + 127.0.0.1:26381
    }
    Group ooxx {

    }
    Group xxoo {
    }
```



```bash
#这个窗口先这样了。 等这启动
cd ../bin/

#新开三个窗口， 把哨兵跑起来哦
cd  ~/test/
vi 26379.cong
#输入以下配置， ooxx和xxoo两套集群， 36379 46379两个主
port 26379
sentinel monitor ooxx 127.0.0.1 36379 2
sentinel monitor xxoo 127.0.0.1 46379 2

vim 26380.conf
#输入以下配置， ooxx和xxoo两套集群， 36379 46379两个主
port 26380
sentinel monitor ooxx 127.0.0.1 36379 2
sentinel monitor xxoo 127.0.0.1 46379 2

vim 26381.conf
#输入以下配置， ooxx和xxoo两套集群， 36379 46379两个主
port 26381
sentinel monitor ooxx 127.0.0.1 36379 2
sentinel monitor xxoo 127.0.0.1 46379 2
```

![image-20210215173507230](image\image-20210215173507230.png)



```bash
#开始启动(这次是真的要启动)， 三个窗口
redis-server 26379.conf  --sentinel
redis-server 26380.conf  --sentinel
redis-server 26381.conf  --sentinel

#在启动两对主从, 新开两个新的窗口
cd ~/test
mkdir 36379
mkdir 36380
mkdir 46379
mkdir 46370

cd /36379
redis-server --port 36379 #起来一个主
cd /36380
redis-server --port 36380 --replicaof 127.0.0.1 36379 #起来一个36380哨兵， 追随36379

cd /46379
redis-server --port 46379 #起来一个主
cd /46380
redis-server --port 46380 --replicaof 127.0.0.1 46379 #起来一个36380哨兵， 追随36379


#启动
cd /root/soft/predixy/predixy-1.0.5/bin
./predixy   ../conf/predixy.conf

redis-cli -p 7617
#往上塞数据就行了

```



