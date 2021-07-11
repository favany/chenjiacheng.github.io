---
layout: post
categories: Laravel
---

## 安装 VirtualBox

官网：[https://www.virtualbox.org](https://www.virtualbox.org)

## 安装 Vagrant

官网：[https://www.vagrantup.com](https://www.vagrantup.com)

## 导入 Homestead Box 虚拟机盒子

**方式一：官方直接下载导入**

安装最新版本：

```
vagrant box add laravel/homestead 
```

或者指定版本：

```
vagrant box add laravel/homestead --box-version=0.3.3
```

**方式二：第三方工具下载导入**

下载地址：[https://app.vagrantup.com/laravel/boxes/homestead](https://app.vagrantup.com/laravel/boxes/homestead)

下载完成后，运行以下命令导入（使用绝对路径）

```
vagrant box add laravel/homestead D:/21ba7076-0f69-4647-a1e7-1a7fb6b626bf
```

**方式三：通过 metadata.json 导入**

下载成功 `.box` 文件后，就可以本地导入盒子了。

在 `.box` 的同文件夹下创建一个 `metadata.json` 文件，内容为以下：

```
{
    "name": "laravel/homestead",
    "versions":
    [
        {
            "version": "0.4.4",
            "providers": [
                {
                  "name": "virtualbox",
                  "url": "homestead-virtualbox-0.4.4.box"
                }
            ]
        }
    ]
}
```

字段说明

* name - 这是导入盒子的名词，理论上，可以任意命名。当然，除非你知道自己在干嘛，否则建议使用 Laravel 默认的；
* version - 可以指定当前盒子导入的版本标示，你可以随意指定，不过建议保持你下载盒子时的版本；
* url - 指定盒子的路径，支持绝对文件路径和相对文件路径。

运行以下命令导入（使用绝对路径）


```
vagrant box add D:/metadata.json
```

> 注：导入成功后，`.box` 文件可任意删除。

**运行 list 命令查看是否添加成功**

```
vagrant box list
```

## 下载 Homestead 管理脚本

```
git clone https://github.com/laravel/homestead.git Homestead
```

下载完成之后我们使用命令行进入 `Homestead` 目录，也可以使用 Git 检出我们需要的 Homestead 版本：

```
cd ~/Homestead
git checkout {{homestead_version}}
```

接下来我们需要初始化 Homestead：

```
bash init.sh
```

运行以上命令后，会在 `~/Homestead` 目录下生成以下三个文件：

- Homestead.yaml - 主要配置信息文件，我们可以在此文件中配置 Homestead 的站点和数据库等信息；
- after.sh - 每一次 Homestead 盒子重置后（provision）会调用的 shell 脚本文件；
- aliases - 每一次 Homestead 盒子重置后（provision），会被替换至虚拟机的 `~/.bash_aliases` 文件中，`aliases` 里可以放一些快捷命令的定义。

## Homestead.yaml 配置文件

```
---

# 虚拟机设置
ip: "192.168.10.10"
memory: 2048
cpus: 2
provider: virtualbox

# SSH 秘钥登录配置
authorize: ~/.ssh/id_rsa.pub

keys:
    - ~/.ssh/id_rsa

# 共享文件夹配置
folders:
    - map: D:/www # 本地目录
      to: /home/vagrant/code # 对应目录

# 站点配置
sites:
    - map: homestead.test
      to: /home/vagrant/code/laravel/public

# 数据库配置
databases:
    - homestead
    
# 自定义变量
variables:
    - key: APP_ENV
      value: local

# 第三方服务
features:
    - mysql: true
    - mariadb: false
    - postgresql: false
    - ohmyzsh: false
    - webdriver: false

#services:
#    - enabled:
#        - "postgresql@12-main"
#    - disabled:
#        - "postgresql@11-main"

# ports:
#     - send: 50000
#       to: 5000
#     - send: 7777
#       to: 777
#       protocol: udp

```

`C:\Windows\System32\drivers\etc\hosts` 需要配置

```
192.168.10.10  homestead.test
```

## Vagrant 常用命令

| 命令行 | 说明 |
| ----------------- | ------------------------------------------- |
| vagrant init      | 初始化 vagrant                              |
| vagrant up        | 启动 vagrant                                |
| vagrant halt      | 关闭 vagrant                                |
| vagrant ssh       | 通过 SSH 登录 vagrant（需要先启动 vagrant） |
| vagrant provision | 重新应用更改 vagrant 配置                   |
| vagrant destroy   | 删除 vagrant                                |

## 客户端连接 MySQL

```
host: 192.168.10.10
port: 3306
user: homestead
pass: secret
```

## 切换 PHP 版本

```
update-alternatives --display php 查看所有 php 版本和当前版本
update-alternatives --config php 执行后，会列出当前 php 所有版本和编号，输入编号，切换到执行的版本
```

```
vagrant@homestead:~$ update-alternatives --display php
php - manual mode
  link best version is /usr/bin/php8.0
  link currently points to /usr/bin/php7.3
  link php is /usr/bin/php
  slave php.1.gz is /usr/share/man/man1/php.1.gz
/usr/bin/php5.6 - priority 56
  slave php.1.gz: /usr/share/man/man1/php5.6.1.gz
/usr/bin/php7.0 - priority 70
  slave php.1.gz: /usr/share/man/man1/php7.0.1.gz
/usr/bin/php7.1 - priority 71
  slave php.1.gz: /usr/share/man/man1/php7.1.1.gz
/usr/bin/php7.2 - priority 72
  slave php.1.gz: /usr/share/man/man1/php7.2.1.gz
/usr/bin/php7.3 - priority 73
```

```
vagrant@homestead:~$ update-alternatives --config php
There are 7 choices for the alternative php (providing /usr/bin/php).

  Selection    Path             Priority   Status
------------------------------------------------------------
  0            /usr/bin/php8.0   80        auto mode
  1            /usr/bin/php5.6   56        manual mode
  2            /usr/bin/php7.0   70        manual mode
  3            /usr/bin/php7.1   71        manual mode
  4            /usr/bin/php7.2   72        manual mode
* 5            /usr/bin/php7.3   73        manual mode

Press <enter> to keep the current choice[*], or type selection number: 3
vagrant@homestead:~$
```

**加载配置文件时指定**

```
# 站点配置
sites:
    - map: homestead.test
      to: /home/vagrant/code/laravel/public
      php: "7.3"
```