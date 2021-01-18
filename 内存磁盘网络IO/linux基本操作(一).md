### 1.内部命令外部命令

```bash
#1.shell自带的命令
#2.不是shell自带的命令
#bash shell程序.
#命令类型
type cd   (buildin内部命令)
type ifconfig

cat ifconfig  (ELF 64-bit LSB)
#
whereis ifconfig

echo "123213"
echo $PATH
```



### 2.linux命令查看帮助文档

```bash
#man
man ifconfig
man yum
#安装man
yum install man

#helo
help cd

```



### 3.bash shell定义变量

```bash
#定义变量
a=2
echo $a
a=angelababy
echo $a

#定义集合,中间不能有空格
arr=(1 2 3)
echo $arr[1]
echo ${arr[1]}
echo ${arr[2]}

#追加~线
echo ~$a~

#当前bash进程号
echo $$

#任务管理器,可以看到vi的进程
ps -ef
#编辑
vim a
#杀掉进程号
kill -9 1287
```



### 4.hash优化命令查询时间的原理

```bash
#yum:先去path下查找,还有hash优化
yum install man
#可看到输入的
hash
hash -r  #清空hash
hash
```



#### 5.linux文件系统

```bash
#
cd /
ll

/bin #用户命令
/ boot #内核程序
/dev  #设备文件
/home  #用户家目录, 有几个用户
/root #管理员
/lib #库文件
/media #挂载点目录
/mnt ##挂载点目录
/opt #第三缸程序安装目录
/proc # 伪文件系统(内核映射文件)
/sys #伪文件系统
/temp #临时文件, /var/temp
/var # 可变化文件,
/bin #可执行文件
/sbin #管理命令

df -h #查看分区菜单
pwd
```





### 6文件系统相关命令

```bash
date 时间
# 加上h 是单位
df -h
#查看子文件夹大小
du -h home
# 显示etc目录下的文件
ls /etc
ls -a #查看隐藏文件
ls -l
man ls  # 查看帮助文档

drwxr-xr-x.
#rwxr-xr-x　　每三位是一个组
#r:read  w : write  x :执行
#d开头: 文件夹
#ｃ开头:　字符设备文件
#ｂ开头:　块设备文件
#ｐ开头:　pipleline 管道文件
# s开头:　socket 套接字文件
#l开头:　软连接（快捷方式）
```



7.文件系统相关命令

```bash
#
cd /
cd .
cd ~
cd ..
pwd

mkdir abc
#级联创建目录
mkdir -p /a/adir/bdir
#创建多个文件夹
mkdir a/xdir a/adir a/bdir
#简写
mkdir a/{1,2,3}dir

#复制文件
cp /etc/profile /etc
#复制目录
cp -r aa /etc


#移动文件
mv profile 1dir
#重命名
mv profile profile.bak
```



### 8.文件系统相关命令 

```bash
#删除文件
rm hello
rm -rf hello  #删除

#硬链接, 删除后没有影响
ln profile abc
#显示文件node号
ll -i 

#软连接, 删除后找不到依赖了, 找不到该文件
ln -s profile  123
ll
rm -rf profile

```



### 9.文件系统相关命令 stat touch 

```bash
#查看文件的详细信息
stat  abc
#同意别人写的权限
chmod o+w abc
#再次查看文件的change时间
stat abc 

#之后可以看到文件的Access Modify Change 时间改变, 能统一下来, 还可以创建一个文件
touch abc

```



### 10.文件系统相关命令 cat more less head tail

```bash
#查看文件的详细信息, 适用于一屏的日志
cat msg
man cat

#可以分页显示,空格显示下一屏, 但是看完以后就退出
more /etc/profile

#空格下一页, b: 上一页, 回车:一行一行的,
#已经放到linux的内存中了, 不建议大文件
less ／etc/profile

#默认前10行的文件
head /etc/profile
#查看前五行的数据
head -5 /etc/profile

#默认后五行的数据
tail /etc/profile
#指定后五行的数据
tail -5 /etc/profile
#增量输出
tail -f 
#不知道这个命令的话问一下这个男人吧!!
man tail

#演示一下tail -f 
touch zfg
tail -f zfg
#新开一个窗口,将内容重定向到zfg,此时在另一个bash窗口可以看到文件被输出出来了
echo "123" >> zfg
#内容已经被写入到zfg文件中了
vi zfg

```



### 11.文本相关命令(管道符) cat b.txt|head -3 

```bash
#查看文件的前三行数据,
cat zfg | head -3

#打印根目录的详细信息
ls -l /

# ls -l 并没有接收到echo的输出流
echo "/" | ls -l
man xargs
#来个骚操作,还是查看跟目录,不过没有颜色哦, xargs强制接受前面的输出流
echo "/" | xargs ls -l

#打印出第五行数据
head -5 zfg | tail -1
#再来个骚操作第六行数据
head -6 zfg | tail -1 


```



### 12.文本编辑器vi

```bash
#编辑模式
#默认光标在第一行
vi profile
#默认光标再第10行
vi +10 profile
#光标默认最后一行
vi + prifile

#光标快捷键
#移动字符: h:左;j:下;k:上;l:右
#单词: 
#  w:移动到下一个单词词首
#  e:跳到下一个单词词尾
#  b:跳到当前或前一个单词的词首行内
#  0:绝对行首
#  ^:行首第一个空白符
#  $:绝对行位
#  G:文章末尾
#  3G:第三行
#  gg:文章开头
# 
#翻页:paup  pgdn 或者: ctr+  f/b
# x: 删除字符
# 5x: 删除5个字符
# r:  替换光标字符
# dd: 删除一行数据
# dw: 删除一个单词
# 
#复制粘贴:
# yw 复制一个单词  p 粘贴
# yy 复制一行     p
#
#撤销重做
# u
# ctr+r
# . 重复上一步操作
#
#
#
#
#输入模式
# i 
# 退回 esc 
# a 当前光标所在字符的后方
# A 行尾
# o 下方增加一行进入输入模式
# O 上方增加一行进入输入模式
# 
#
#保存
# 快捷保存: shift + zz
# :q
# :wq
# :q! 
# 



```

![image-20210116233724663](image\image-20210116233724663.png)







### vi命令末行模式

```bash
#显示行号
set nu
#取消显示行号
set nonu

#这个貌似没起作用, 应该是root权限的问题
#在保存时候会出现'readonly' option is set (add ! to override, 大意了!!!!
set readonly

#查找向下
/after
  n:查找下一个
　N: 查找上一个
#向上查找　
:?after　

#末行模式下执行查找命令(我曹, 用于在编辑模式下, 查找文件的安装路径, 再也不用先退出了,想哭想哭)
:!ls -l /javahome
#返回后再回车就可以了, 老子再也不用退出找安装路径了


#查找替换
#替换当前光标行的after
:s/after/before (只会替换第一个匹配到的)
:s/after/before/g (会替换当前行的所有after)
:s/after/before/gi (会替换当前行的所有after,并忽略大小写)
:.,+2s/after/before/gi (向下两行全部替换,after替换before)
:%s/before/after/gi  (全文替换before)
0,$s/before/after/gi (全文替换)



#删除
#全删除: 0,$d
#删除前三行:  
   #  .,+2d
#删除3-5行数据: 
   :3,5d
#删除倒数第二行:
   :$-1d
#保留最后一行
   :1,$-1d
#复制:
   :1,3y   (复制)
   p       (粘贴)
   :0,$y   (全部复制)
```

不说了, 睡觉!!