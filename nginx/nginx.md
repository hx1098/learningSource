[TOC]

#### 目录

#### **一、代理服务器**

1、什么是代理服务器

代理服务器，客户机在发送请求时，不会直接发送给目的主机，而是先发送给代理服务器，代理服务接受客户机请求之后，再向主机发出，并接收目的主机返回的数据，存放在代理服务器的硬盘中，再发送给客户机。

2、为什么要使用代理服务器

1）提高访问速度

  由于目标主机返回的数据会存放在代理服务器的硬盘中，因此下一次客户再访问相同的站点数据时，会直接从代理服务器的硬盘中读取，起到了缓存的作用，尤其对于热门站点能明显提高请求速度。

2）防火墙作用

  由于所有的客户机请求都必须通过代理服务器访问远程站点，因此可在代理服务器上设限，过滤某些不安全信息。

3）通过代理服务器访问不能访问的目标站点

  互联网上有许多开发的代理服务器，客户机在访问受限时，可通过不受限的代理服务器访问目标站点，通俗说，我们使用的翻墙浏览器就是利用了代理服务器，虽然不能出国，但也可直接访问外网。

#### **二、反向代理 VS 正向代理**

1、什么是正向代理？什么是反向代理？

正向代理，架设在客户机与目标主机之间，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器,并将本来要直接发送到Web服务器上的http请求发送到代理服务器中。

![img](https://img-blog.csdn.net/20160531205420201?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

反向代理服务器架设在服务器端，通过缓冲经常被请求的页面来缓解服务器的工作量，将客户机请求转发给内部网络上的目标服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器与目标主机一起对外表现为一个服务器。

![img](https://img-blog.csdn.net/20160531205433342?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2、反向代理有哪些主要应用？

现在许多大型web网站都用到反向代理。除了可以防止外网对内网服务器的恶性攻击、缓存以减少服务器的压力和访问安全控制之外，还可以进行负载均衡，将用户请求分配给多个服务器。

#### **三、方向代理服务器Nginx**

Nginx作为近年来较火的反向代理服务器，安装在目的主机端，主要用于转发客户机请求，后台有多个http服务器提供服务，nginx的功能就是把请求转发给后面的服务器，决定哪台目标主机来处理当前请求。下面演示如何进行配置使Nginx发挥作用。

1、模拟n个http服务器作为目标主机

用作测试，简单的使用2个tomcat实例模拟两台http服务器，分别将tomcat的端口改为88和8080

2、配置IP域名

C:\Windows\System32\drivers\etc\hosts中加入

127.0.0.1 77.xx.com

127.0.0.1 80.xx.com

3、配置nginx.conf

```nginx
user  www www;
worker_processes auto;
error_log  /www/wwwlogs/nginx_error.log  crit;
pid        /www/server/nginx/logs/nginx.pid;
worker_rlimit_nofile 51200;

events
    {
        use epoll;
        worker_connections 51200;
        multi_accept on;
    }

http
    {
        include       mime.types;
		#include luawaf.conf;
		include proxy.conf;

        default_type  application/octet-stream;

        server_names_hash_bucket_size 512;
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;
        client_max_body_size 50m;

        sendfile   on;
        tcp_nopush on;

        keepalive_timeout 60;

        tcp_nodelay on;

        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        fastcgi_buffer_size 64k;
        fastcgi_buffers 4 64k;
        fastcgi_busy_buffers_size 128k;
        fastcgi_temp_file_write_size 256k;
		fastcgi_intercept_errors on;

        gzip on;
        gzip_min_length  1k;
        gzip_buffers     4 16k;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types     text/plain application/javascript application/x-javascript text/javascript text/css application/xml;
        gzip_vary on;
        gzip_proxied   expired no-cache no-store private auth;
        gzip_disable   "MSIE [1-6]\.";

        limit_conn_zone $binary_remote_addr zone=perip:10m;
		limit_conn_zone $server_name zone=perserver:10m;

        server_tokens off;
        access_log off;

#主要是个地方,配置好你自己本地或者远程的ip或者端口.
upstream tomcatserver1 {  
    server 127.0.0.1:77;  
    }  
upstream tomcatserver2 {  
    server 127.0.0.1:8080;  
    }
server
    {   #我的nginx默认监控了77和80端口, 根据自己需要进行设置哈
        listen 77;
        #如果上面的77对应的是域名, 一定要写你的域名, 不是话就随便了, 我这里的php的代理, 所以写一个phpadmin
        server_name xxxx;  
        index index.html index.htm index.php;
        root  /www/server/phpmyadmin;

        #error_page   404   /404.html;
        include enable-php.conf;
        
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }

        location ~ /\.
        {
            deny all;
        }

        access_log  /www/wwwlogs/access.log;
        
     }
     server
      {
        listen 80;
        server_name  jd.xxx.com;
        root  usr/local/tomcat;
        location /v {
            proxy_pass http://127.0.0.1:8080/valentines;
        }

        access_log  /www/wwwlogs/access.log;
      }
include /www/server/panel/vhost/nginx/*.conf;
}
```

流程：

1）由于我使用的是phpadmin, 已经可以使用域名访问了, 浏览器访问 xxx.com，通过本地host文件域名解析，找到本机的服务器（安装nginx）, 然后再访问二级域名, jd.xxx.com.

2）nginx反向代理接受客户机请求，找到server_name为8081.max.com的server节点，根据proxy_pass对应的http路径，将请求转发到upstream tomcatserver2上，即端口号为8080的tomcat。

 

