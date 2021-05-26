---
layout: post
title: 在 Linux 系统中使用 rpm 包安装 FTP 服务
categories: Linux
---

注意：rpm 包在 Linux 的安装光盘里面。

## 挂载光驱，读取光盘里面的数据。

![01.png](/static/images/2016/11/29/01.png)

## 开启安装，前提要进入到光盘里面的 Packages 文件夹。

![02.png](/static/images/2016/11/29/02.png)

## 要启动 ftp 服务

![03.png](/static/images/2016/11/29/03.png)

## 使用 ftp 客户端软件，进行连接，上传文件。

注意：root 用户默认不能登录 ftp 服务的。

![04.png](/static/images/2016/11/29/04.png)

如果出现提示 500 的错误提示。

原因：redhat 面向目标是企业，为了安全，增加了一个 selinux 服务，关闭该服务即可。

在 `/etc/selinux/config`，把 `enforcing=>disabled`，运行 `setenforce 0` 命令使立即生效。

## 关闭 selinux 服务的步骤：

第一步：使用 vim 打开配置文件 `/etc/selinux/config`

![05.png](/static/images/2016/11/29/05.png)

![06.png](/static/images/2016/11/29/06.png)

第二步：让该配置立即生效，运行 `setenforce 0` 命令使立即生效

![07.png](/static/images/2016/11/29/07.png)

再次连接 ftp 服务，就成功了。

![08.png](/static/images/2016/11/29/08.png)

**如果连接不成功。请注意：**

1. 查看 vsftpd 服务是否开启。

2. 关闭防火墙 `service iptables stop`

3. 用户名或密码是否输入错误。