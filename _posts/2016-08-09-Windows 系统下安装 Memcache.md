---
layout: post
categories: Memcache
tags: [Memcache,开发环境]
---

**方式一：直接使用，无需安装。（在开发时推荐使用）**

把软件拷贝到指定位置，一般和其他的安装软件（比如 Apache 等）在同级目录下面，主要是便于管理。

![01.png](/static/images/20160809/01.png)

以 cmd 的方式，运行 Memcache

![02.png](/static/images/20160809/02.png)

![03.png](/static/images/20160809/03.png)

启动后，该窗口不要关闭，一旦关闭，则服务就停止了。

![04.png](/static/images/20160809/04.png)

**方式二：把 Memcache 安装成 Window 的一个服务。（在生产环境中推荐使用）**

通过查看 Memcached 的帮助。

![05.png](/static/images/20160809/05.png)

注意：在把 Memcache 安装成 Window 的一个服务时，要以管理员的方式启动 cmd。

![06.png](/static/images/20160809/06.png)

查看服务是否安装成功

![07.png](/static/images/20160809/07.png)

**安装可能失败的原因：**

1. 如果你是用 win7, win8 系统，他对安全性要求高，因此，需要大家使用管理员的身份来安装和启动. 具体是 `程序开始 => 所有程序 => 附件=> cmd` （单击右键，选择以管理员的身份来执行）。

2. 存放 `memcache.exe` 目录不要有中文或者特殊字符。

3. 安装成功，但是启动会报告一个错误信息，提示缺少 `xx.dll`，你可以从别的机器拷贝该 `dll` 文件，然后放入到 `system32` 下即可。

如果上面三个方法都不可以，你可以直接这样启动 `cmd > memcached.exe -p 端口`（这种方式不能关闭窗口）