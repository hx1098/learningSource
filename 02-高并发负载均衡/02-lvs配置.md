





```bash
#第一台配置隐藏的vip
ifconfig ens33:8 192.168.99.149/24

#第二台修改调协议
cd /proc/sys/net/ipv4/conf/ens33/
#修改arp_ignore,arp_announce文件
#arp_ignore 默认值为0, 修改不能用vi
cat arp_ignore

#重定向修改该值
echo 1 > arp_ignore
cat arp_ignore
echo 2 > arp_announce
cat arp_announce

cd  /proc/sys/net/ipv4/conf/all
echo 1 > arp_ignore
echo 2 > arp_announce

#这里是4个255:  他们掩码要按位与的运算,规避死循环
ifconfig lo:2  192.168.99.149 netmask 255.255.255.255
ifconfig


#第三台修改调协议
 echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_ignore 
 echo 2 > /proc/sys/net/ipv4/conf/all/arp_ignore 
 echo 1 > /proc/sys/net/ipv4/conf/ens33/arp_announce 
 echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
ifconfig lo:8 192.168.99.149 netmask 255.255.255.255 
ifconfig

```

### 2.搭建RS中的server

```bash
#node02,node03安装http
#重启后会失效
yum install httpd -y 
service httpd start
vi /var/www/html/index.html
   from 192.168.190.67
vi /var/www/html/index.html
   from 192.168.190.67   


#永久关闭防火墙
systemctl disable firewalld

```

