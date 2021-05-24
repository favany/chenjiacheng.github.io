---
layout: post
title: MyISAM 和 InnoDB 中索引的数据结构
categories: MySQL
---

## 1. MyISAM引擎的索引

索引的节点中存储的是数据的物理地址（磁道和扇区）

在查找数据时，查找到索引后，根据索引节点中的物理地址，查找到具体的数据内容

![01.png](/static/images/2017/02/22/01.png)

## 2. InnoDB引擎的索引

innodb的主键索引文件上直接存放该行数据，称为聚簇索引，非主索引指向对主键的引用

myisam中，主索引和非主索引，都指向物理行(磁盘位置)

![02.png](/static/images/2017/02/22/02.png)

注意：对于innodb来说

(1)主键索引，既存储索引值，又在叶子中存储行的数据

(2)如果没有主键，则会Unique key做主键

(3)如果没有unique，则系统生成一个内部的rowid做主键

(4)像innodb中，主键的索引结构中，既存储了主键值，又存储了行数据，这种结构称为“聚簇索引”

**先创建一个myisam引擎的表，并插入数据**

```
create table t5(
    id int primary key auto_increment,
    name varchar(12) not null comment '名称'
)engine myisam charset utf8;
```

![03.png](/static/images/2017/02/22/03.png)

说明：mysiam引擎的表的数据是按照插入的顺序显示的。

**再创建一个inodb引擎的表，并插入数据**

```
create table t6(
    id int primary key auto_increment,
    name varchar(12) not null  comment '名称'
)engine innodb charset utf8;
```

![04.png](/static/images/2017/02/22/04.png)

说明：innodb引擎的表的数据是按照主键的顺序插入的。