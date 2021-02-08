[TOC]

##               目录结构

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

 

```
server {
        listen       80;
        server_name  localhost;
        #location / {
        #    proxy_pass http://localhost:8080/404/404.html;
        #}
        location /v {
            proxy_pass http://127.0.0.1:8080/valentines;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```



