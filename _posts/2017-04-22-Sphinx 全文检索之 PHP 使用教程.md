---
layout: post
title: Sphinx 全文检索之 PHP 使用教程
categories: PHP
---

## 一、Sphinx简介

### 1. Sphinx是什么？

中文名：全文索引引擎。只支持英文和俄文。但是只要有相应的语言包也可支持任何语言。国内有一团队在Sphinx基础上封装了一个带中文包的软件：coreseek。

### 2. 为什么要用Sphinx？

在mysql数据库中，对于如下sql语句，`select * from xxx where like xxx '%xxx';` (以%开头的like查询)，无法使用到任何索引优化，导致如果数据量非常大，查询速度会非常慢。而这种sql语句在很多功能中都要用到，如根据歌词查询歌曲，根据剧情查询电影等。如果要加快查询只能使用第三方软件，**Sphinx和lucence**。mysql中也提供了全文索引的功能，但是有两个问题：（1）只有myisam引擎支持（2）对中文支持不好。不过现在最新的mysql5.6版本中的innodb1.2的版本也同样支持全文索引。

### 3. Sphinx的使用原理

(1)先创建数据源。

(2)根据数据源创建索引，使用分词技术。

(3)php把查询的关键词给Sphinx服务器，Sphinx根据关键词查找到关键字在mysql表里面的记录的id.Sphinx把id返回给php查询端。

(4)php根据返回的id，查询mysql服务器。

![01.png](/static/images/2017/04/22/01.png)

## 二、安装使用

### 1. 下载，进行解压

官网下载：<http://www.Sphinxsearch.com>

支持中文分词：<http://www.coreseek.com>

解压后拷贝到指定的目录，一般和其他的环境程序在同一级目录（便于管理）

![02.png](/static/images/2017/04/22/02.png)

### 2. 拷贝配置文件

把etc目录下面的csft_mysql.conf文件拷贝到上一级目录，并改名为sphinx.conf

![03.png](/static/images/2017/04/22/03.png)

![04.png](/static/images/2017/04/22/04.png)

## 三、使用配置

### 1. 对查询数据创建索引

主要是配置sphinx.conf配置文件

(1)配置数据源（被查询的数据，就是sql语句执行的结果）

配置语法：

```
source 数据源名称 {

}
```

![05.png](/static/images/2017/04/22/05.png)

注意：在一个配置文件中，可以配置多个数据源。

(2)配置数据源生成的索引文件存放的位置。

配置语法：

```
index 索引的名字 {

}
```

![06.png](/static/images/2017/04/22/06.png)

注意：该索引必须与一个数据源相对应。

(3)配置Sphinx服务器的信息

![07.png](/static/images/2017/04/22/07.png)

### 2. 创建索引

执行Sphinx下的一个程序 `indexer.exe –c 配置文件(全路径) --all | 索引的名字` (--all：为配置文件中所有的索引创建索引文件。也可以使用索引的名字只为某一个索引创建索引文件)

![08.png](/static/images/2017/04/22/08.png)

(1)以管理员的方式打开cmd窗口，执行indexer.exe命令。

![09.png](/static/images/2017/04/22/09.png)

(2)查看创建的索引文件。

![10.png](/static/images/2017/04/22/10.png)

### 3. 启动Sphinx服务器

(1)把Sphinx软件安装为一个系统一个服务。

![11.png](/static/images/2017/04/22/11.png)

```
语法：searchd.exe –c 配置文件 --install
```

可以通过 searchd --help 查看帮助

![12.png](/static/images/2017/04/22/12.png)

![13.png](/static/images/2017/04/22/13.png)

(2)启动该服务。

![14.png](/static/images/2017/04/22/14.png)

查看端口是否启动：

![15.png](/static/images/2017/04/22/15.png)

### 4. 通过php查询使用

(1)在php使用中，需要一个Sphinx的一个接口文件。

![16.png](/static/images/2017/04/22/16.png)

使用Sphinx时，需要把sphinxapi.php文件拷贝到项目中来即可。

(2)代码如下。

![17.png](/static/images/2017/04/22/17.png)

![18.png](/static/images/2017/04/22/18.png)

```
require 'sphinxapi.php';
// 使用Sphinx来完成查询
$sc = new SphinxClient();             // 生成客户端
$sc->setServer('localhost', 9312);    // 设置服务器
// $sc->query('查询的关键词', 索引文件的名称);
$keyword="电影";
$indexname ='movie';
$res = $sc->query($keyword,$indexname);
$ids   = $res['matches'];
$id = array_keys($ids);
$id = implode(',',$id);
mysql_connect("localhost",'root','root');
mysql_query('use php');
mysql_query('SET NAMES UTF8');
$sql="select id,title,description from movie where id in($id)";
$res = mysql_query($sql);
$list=array();
while($row=mysql_fetch_assoc($res)){
    $list[]=$row;
}
foreach($list as $v){
    echo $v['title'].'<br/>'.$v['description'].'<hr>';
}
```

## 四、匹配模式

**1. SPH_MATCH_ALL：完全匹配所有的词**

如“冬天 的 雪”，并不会匹配 “我爱冬天”，但可以匹配 “我的朋友，爱冬天，和雪”。

因为“冬天的雪” 被分成 “冬天”，“的”，“雪”三个词，匹配条件是同时包含这三个词，“我爱冬天”里只包含一个“冬天”

**2. SPH_MATCH_PHRASE：必须匹配整个短语**

如“冬天的雪”，不会匹配 “我的朋友，爱冬天，和雪”，虽然都包含同样的需要严格匹配不再健忘，只匹配“冬天的雪”

**3. SPH_MATCH_ANY：匹配任意一个词**

如“冬天 的 雪”，并会匹配 “我爱冬天”。

"冬天的雪“ -》 ”冬天“ ”的“ ”雪“

因为“我爱冬天”里有一个“冬天”相匹配。

**4. SPH_MATCH_EXTENDED：支持一些扩展的语法**

支持 @字段 查询

如，查询title包含 abc , content 包含 bcd的：

'@title abc @content bcd'

SPH_MATCH_BOOLEAN：与，或，非，分组 &,or,!,()

如：hello | world

查询“手机”，或“冬天”

## 五、查到得关键词添加样式显示

主要使用：buildExcerpts：创建文档摘要，对关键词添加样式显示。

![19.png](/static/images/2017/04/22/19.png)

## 六、增加索引

### 1. 增量索引的原理

![20.png](/static/images/2017/04/22/20.png)

### 2. 实现方式

(1)新建一张表，记录一下上次已经创建好索引的最后一条记录的id

(2)当索引时，然后从数据库中取出所有大于id的数据，这些就是新的数据然后创建一个小的索引文件。

(3)把增量这部分数据生成的小的索引合并到主索引文件上去。

(4)把最后一条记录的id更新到第一步创建的表中。

### 3. 实现步骤

(1)创建一个表，用于记录创建好索引的最后一条记录的id

![21.png](/static/images/2017/04/22/21.png)

(2)修改Sphinx的配置文件，主索引的数据源

![22.png](/static/images/2017/04/22/22.png)

(3)创建增量索引的数据源，以及增量索引文件存储的位置

![23.png](/static/images/2017/04/22/23.png)

### 4. 把Sphinx的服务器停止，并删除原来的索引文件。

(1)创建主索引

![24.png](/static/images/2017/04/22/24.png)

(2)查看a表里面是否记录最大的id

![25.png](/static/images/2017/04/22/25.png)

### 5. 启动Sphinx的服务器，进行查询测试。

![26.png](/static/images/2017/04/22/26.png)

### 6. 添加一部新的电影。

### 7. 创建增量索引。

![27.png](/static/images/2017/04/22/27.png)

### 8. 把创建的增量索引合并到主索引里面。

使用的语法：`indexer -c 配置文件 --merge 主索引文件 增量索引文件 --rotate --rotate` 强制合并，可以不关闭Sphinx服务的情况下合并。

![28.png](/static/images/2017/04/22/28.png)