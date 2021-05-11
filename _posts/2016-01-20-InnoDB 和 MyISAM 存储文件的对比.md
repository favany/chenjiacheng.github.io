---
layout: post
title: InnoDB 和 MyISAM 存储文件的对比
categories: MySQL
taps: MySQL,存储引擎
---

**存储引擎: 数据在数据库存储的方式**

MySQL 提供了五种存储引擎: 其中 InnoDB 和 MyISAM (免费); 另外三种收费

![01.png](/static/images/2016/01/20/01.png)

**InnoDB 和 MyISAM 对比**

1、InnoDB 会将数据和索引统一存储到 ibdata1文件; MyISAM 会自有表数据

![02.png](/static/images/2016/01/20/02.png)

MyISAM 会创建三个文件: InnoDB 只有一个文件

![03.png](/static/images/2016/01/20/03.png)

2、MyISAM 的备份非常简单: 直接将三个文件移动到指定的数据库文件夹即可; InnoDB 必须还要将 ibdata1 文件同时带走

![04.png](/static/images/2016/01/20/04.png)

**存储引擎的选择: 默认（InnoDB）**

**在 Oracle 接收 MySQL 之后已经停止了对 MyISAM 的支持: 只继续支持 InnoDB.**