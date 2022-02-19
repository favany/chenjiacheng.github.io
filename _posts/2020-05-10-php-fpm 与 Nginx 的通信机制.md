---
layout: post
categories: PHP
tags: [PHP]
---

我们来看一段Nginx中fastcgi_pass的配置：

```
location ~ .php$ {  
root /home/wwwroot;  
fastcgi\_pass 127.0.0.1:9000;
    #fastcgi_pass  unix:/var/run/php-fpm/php-fpm.sock;
    #fastcgi_pass  unix:/tmp/php-cgi.sock;
    try_files $uri /index.php =404;
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```

## 两种方式

Nginx 和 PHP-FPM 的进程间通信有两种方式，一种是TCP，一种是 UNIX Domain Socket.

## tcp

允许通过网络进程之间的通信，也可以通过loopback进行本地进程之间通信。

## UNIX Domain Socket

允许在本地运行的进程之间进行通信。

其中TCP是IP加端口,可以跨服务器.而UNIX Domain Socket不经过网络,只能用于Nginx跟PHP-FPM都在同一服务器的场景.用哪种取决于你的PHP-FPM配置:

```
方式1:
php-fpm.conf: listen = 127.0.0.1:9000
nginx.conf: fastcgi_pass 127.0.0.1:9000;
方式2:
php-fpm.conf: listen = /tmp/php-fpm.sock
nginx.conf: fastcgi_pass unix:/tmp/php-fpm.sock;
```

其中php-fpm.sock是一个文件,由php-fpm生成,类型是srw-rw—-.

UNIX Domain Socket和tcp链接的区别

一方面：

UNIX Domain Socket可用于两个没有亲缘关系的进程,是目前广泛使用的IPC机制。

UNIX Domain Socket减少了不必要的tcp开销，而tcp需要经过loopback，还要申请临时端口和tcp相关资源。

另一方面：

unix socket高并发时候不稳定，连接数爆发时，会产生大量的长时缓存，在没有面向连接协议的支撑下，大数据包可能会直接出错不返回异常。tcp这样的面向连接的协议，多少可以保证通信的正确性和完整性。

```
UNIX Domain Socket（只能是本地）:
Nginx <=> socket <=> PHP-FPM
TCP Socket(本地回环):
Nginx <=> socket <=> TCP/IP <=> socket <=> PHP-FPM
TCP Socket(Nginx和PHP-FPM位于不同服务器):
Nginx <=> socket <=> TCP/IP <=> 物理层 <=> 路由器 <=> 物理层 <=> TCP/IP <=> socket <=> PHP-FPM
```

## 类似的软件

像mysql命令行客户端连接mysqld服务也类似有这两种方式:

使用Unix Socket连接(默认):

```bash
mysql -uroot -p --protocol=socket --socket=/tmp/mysql.sock
```

使用TCP连接:

```bash
mysql -uroot -p --protocol=tcp --host=127.0.0.1 --port=3306
```

总结

此处，用unix domain socker的方式，比tcp方式速度更快，但是tcp是面向连接的协议，稳定性更高，这是一个区别点。

## 进一步优化

提高nginx和php-fpm使用的 unix socket稳定性(单机能力有限)

### 1.修改内核参数

```
net.unix.max_dgram_qlen = 4096
net.core.netdev_max_backlog = 4096
net.core.somaxconn = 4096
```

### 2.提高backlog

backlog默认位128，1024这个值最好换算成自己正常的QPS。

```
nginx.conf
server{
  listen 80 default backlog=1024;
}

php-fpm.conf
listen.backlog = 1024
```

### 3.增加sock文件和php-fpm实例

在/dev/shm新建一个sock文件，在nginx中通过upstream魔抗将请求负载均衡到两个sock文件，并且将两个sock文件分别对应到两套php-fpm实例上。