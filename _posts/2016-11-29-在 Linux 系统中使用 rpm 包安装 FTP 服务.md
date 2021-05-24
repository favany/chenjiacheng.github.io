---
layout: post
title: 在 Linux 系统中使用 rpm 包安装 FTP 服务
categories: Linux
---

注意：rpm包在linux的安装光盘里面。

**1. 挂载光驱，读取光盘里面的数据。**

![01.png](/static/images/2016/11/29/01.png)

**2. 开启安装，前提要进入到光盘里面的Packages文件夹。**

![02.png](/static/images/2016/11/29/02.png)

**3. 要启动ftp服务**

![03.png](/static/images/2016/11/29/03.png)

**4. 使用ftp客户端软件，进行连接，上传文件。**

注意：root用户默认不能登录ftp服务的。

![04.png](/static/images/2016/11/29/04.png)

如果出现提示500的错误提示。

原因：redhat面向目标是企业，为了安全，增加了一个selinux 服务，关闭该服务即可。

在/etc/selinux/config

把enforcing=>disabled

运行setenforce 0命令使立即生效

**关闭selinux服务的步骤：**

第一步：使用vim打开配置文件/etc/selinux/config

![05.png](/static/images/2016/11/29/05.png)

![06.png](/static/images/2016/11/29/06.png)

第二步：让该配置立即生效，运行setenforce 0命令使立即生效

![07.png](/static/images/2016/11/29/07.png)

再次连接ftp服务，就成功了。

![08.png](/static/images/2016/11/29/08.png)

如果连接不成功。请注意：

(1)查看vsftpd服务是否开启。

(2)关闭防火墙(service iptables stop)

(3)用户名或密码是否输入错误。