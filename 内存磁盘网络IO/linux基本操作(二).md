上一节我们由浅入深介绍了linux的基本操作, 这一节接着介绍进阶版的

上一节链接:  https://blog.csdn.net/qq_32565267/article/details/112727419

### 1.正则表达式的简单使用

在linux中正则表达式有两种操作符,一种是匹配操作符,一种是重复操作符.

![image-20210117102708687](image\image-20210117102708687.png)

```bash
#grep
man grep


```



### 1.正则表达式的简单使用(grep)

```bash
#先创建一个文件, 写入如下内容,保存! 随意哈
hello hx1098
hello angelababy
hello dilireba
ttttt what fuck
ll hello zhangxinyi
ll hel mdsfhsiodf lo
iiiii uiwerhen
hello 66666
wo ai ni zhong guo
mei zi zi

#查找包含hello的行
grep "hello" hello(我的文件名)
#查找包含0-9的数据行
grep "[0-9]" hello
#查找包含o和6 的数据行
grep "[o6]" hello
#查找至少包含三位数的行(需要加上转义符号)
grep "[0-9]\{3\}" hello
grep -E "[0-9]{3}" hello  (E:扩展表达式)
#包含hello的单词
grep "\<hello\>" hello
#查询包含三位数的
grep  "\([^0-9][0-9]\|^[0-9]\)[0-9]\([0-9][^0-9]\|[0-9]$\)" hello
#简化后
grep -E  "([^0-9][0-9]|^[0-9])[0-9]([0-9][^0-9]|[0-9]$)" hello

```



### 2.反向引用简化正则表达式

```bash
#看一下我的文档
cat hello
##start
123
hello hx1098
hello angelababy
hello dilireba
ttttt what fuck
ll hello zhangxinyi
ll hel mdsfhsiodf lo 123
iiiii uiwerhen
hello 66666
wo ai ni zhong guo
mei zi zi 
114 dfj 456
helloehiuwhreihrhhello
hello world  hello world
world hello world hello
##end 就这末多了嘿嘿

#问题: 
#查找以hello开头的数据
grep "\<hello" hello
#查找以hello单词开头的数据
grep "\<hello\>" hello
```

![image-20210117143529372](image\image-20210117143529372.png)



```bash
#问题2
#找出出现2-3次"l"的行
grep "l\{2,3\}" hello
```

![image-20210117143946516](image\image-20210117143946516.png)



```bash
#问题3
#找出hello world hello wrold的行
grep "hello.*world.*hello.*world.*" hello
#简化(\1 : hello.*   \2: world.*)  反向引用
grep "\(hello.*\)\(world.*\).*\1.*\2" hello
```



### 3.其他文本处理命令(cut,sort,wc)

```bash
cut
#按照空格进行切割(f1代表第一个位置)
cut -d ' ' -f1 hello
cut -d ' ' -f1,2 hello
cut -d ' ' -f1-3 hello
#不显示分隔符, 即: 不显示没有分割符号的行
cut -d ' ' -f1-3  -s hello

#复制一个文件
cp /etc/passwd  .
#打印出所的用户名称
cut -d ':' -s -f1 passwd
```

![image-20210117145609256](image\image-20210117145609256.png)





```bash
#sort
man sort

#编辑一个文件start,按照颜值打分,内容如下(仅仅是学习, 莫怪,莫怪!)
#start
zhangziyi 12
liudehua 10
angelababy 6
damimi 12
liqin 9
laowang 5
Laowang 5
#end

#问题: 
#按照人名字母进行排序(双引号貌似也是可以的哦)
sort -t ' ' -k1 start
sort -t " " -k1 start
#按照人名字母进行排序(去重复)
sort -t " " -k1 -u start
#按照人名字母进行排序(去重复,忽略大小写),注意: 忽略大小写时候,laowang 后面的颜值分必须一样的鸭!否则没有效果的
sort -t " " -k1 -u -f start
#按照颜值来进行排序(默认是按照字典序来排序的)
sort -t ' ' -k2  start
#升序按照数字排序
sort -t ' ' -k2  -n start
#降序按照数字排序
sort -t ' ' -k2  -n r start


```

![image-20210117152631837](image\image-20210117152631837.png)





```bash
#wc的使用(可不是人家想的那样哦!!) 
#还是使用start的颜值文件
wc start

[root@node01 ~]# cat start 
zhangziyi 12
liudehua 10
angelababy 6
damimi 12
liqin 9
laowang 5
Laowang 5
#7:代表有7行数据, 14:14个单词; 76个字节(加上空格);
[root@node01 ~]# wc start 
 7 14 76 start

#查看有多少个单词
wc -w start
#多少行
wc -l start
#多少个字节
wc -c start

#进阶组合使用,看一下我的linux有多少个用户
cat password | wc -l

```



### 4.大众情人级别的命令(sed): 行编辑器



![image-20210117153846649](D:\内存磁盘网络IO\image\image-20210117153846649.png)



```bash
#sed 和 vi 相比, 一个是行级别, 一个是文件级别的
vi /txt
#输入一下字符
hello1
hello2
hello3
#保存,退出
```

```bash
#开始干饭,少年!
#显示第二行数据(一般人思维)
[root@node01 ~]# sed "2p" txt 
hello1
hello2
hello2
hello3
#可以看到: 这个显示了四行数据, what funck? 这里是因为sed 会吧数据一行行的加载到内存,默认会把内存中的数据打印出来, 然后就会有了第一行, 第二行,和第四行数据, 那为啥会多出一行来嘞? 因为做了"2p"的操作, 嗖嘎,所以又打印了一遍这个数据. 那如果我不想显示哪?(即不显示模式空间的内容) 如下:
[root@node01 ~]# sed -n  "2p" txt 
hello2
[root@node01 ~]# sed -n  "3p" txt 
hello3

#删除第三行数据(删除时候就不要加上-n静默模式了,否则哈都不显示的)
[root@node01 ~]# sed "3d" txt 
hello1
hello2
[root@node01 ~]# sed -n "3d" txt 
[root@node01 ~]#
#分析:　为啥第一个显示内容，　第二个就不显示了
# 那是因为: 第一个没加模式空间, 一行二行数据加载到内存,显示了出来, 第三行也加载到了内存, 但是被删除后,吧空行写回到了第三行,所以显示了第一行第二行
#          第二个虽然也加了模式空间, 但是只显示被删除的行, 因为被删除了, 所以就不显示了.


#查看txt数据, 发现数据并没有真正的把数据给删除掉
#真删除
[root@node01 ~]# sed -i "3d" txt 
[root@node01 ~]# 
[root@node01 ~]# cat txt 
hello1
hello2
[root@node01 ~]# 
#分析: 为啥真删除的时候啥也不显示了? 
#sed命令将数据一行行的加载到内存, 
# 第一行的时候, 删除吗？不是第三行数据, 不删除, 不显示
# 第二行的时候, 删除吗？不是第三行数据, 不删除, 不显示
# 第三行的时候, 删除吗？要得删除; 删除后空写回到第三行, 返回出一个空行
# 所以你看到返回的数据是空的. 明白了


#a: 追加
#追加一个小姐姐
[root@node01 ~]# sed "axiaojiejie" txt 
hello1
xiaojiejie
hello2
xiaojiejie
#会在第一行下面和第二行下面追加一个小姐姐, 那我在上面追加呢
[root@node01 ~]# sed "ixiaojiejie" txt 
xiaojiejie
hello1
xiaojiejie
hello2
[root@node01 ~]# cat txt 
hello1
hello2
[root@node01 ~]# 

#此时发现, 文件并没有被修改, 还是要加上i的命令的
[root@node01 ~]# sed -i "ixiaojiejie" txt 
[root@node01 ~]# cat txt 
xiaojiejie
hello1
xiaojiejie
hello2

#c替换
[root@node01 ~]# sed "cxiaojiejie" txt 
xiaojiejie
xiaojiejie
xiaojiejie
xiaojiejie
sed  -i "cxiaojiejie" txt

```



sed进阶版:

```bash
#例子1:文件准备
#创建一个文件inittablep,写入如下内容
 0.jdsoipfhpiodsf
 1.idhsjifhsoidfsd
 2.dsohfishdfihsdipf
 3.dsiphfipdshapufihds[oifs
 4.fdhuishafuidshauifhasd
 5.iuoehuisdf
 6.duioshafios 
 7.dsuifhdsuhf
id:6:hello


#操作
cat inittab
#把该文件的最后的3改为6
sed "s/3/8/" inittab
#将所有的数据都改为了8
sed "s/[0-9]/.8" inittab
#将最后一行的数字6改为8
sed "s/id:[0-9]:hello/id:8:hello/" inittab
#使用反向引用,简化(貌似也没咋简化, 就是反斜杠多了)
sed "s/\(id:\)[0-9]\(:hello\)/\18\2/" inittab

#动态的sed
version=2
echo $version
sed "s/\(id:\)[0-9]\(:hello\)/\1$version\2/" inittab
version=7
sed "s/\(id:\)[0-9]\(:hello\)/\1$version\2/" inittab

```



```bash
#例子二:
#文件准备:
cp /etc/sysconfig/network-scripts/ifcfg-ens33  .
cat
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.190.66
GATEWAY=192.168.190.2
NETMASK=255.255.255.0
DNS1=114.114.114.114

#原来修改文件需要打开, 现在用sed来修改文件
[root@node01 ~]# sed "s/\(IPADDR=\([0-9]\?[0-9][0-9]\?.\)\{3\}\).*/\188/" ifcfg-ens33 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.190.88
GATEWAY=192.168.190.2
NETMASK=255.255.255.0
DNS1=114.114.114.114
#可以看到文件已经被正确修改, 要真正修改的话, 在加上-i 就可以了

```





