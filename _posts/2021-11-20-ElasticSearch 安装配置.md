---
layout: post
categories: ElasticSearch
tags: [ElasticSearch]
---

我们需要下载和安装 ElasticSearch 的服务端和客户端

> 注意：ElasticSearch 是使用 Java 开发的，且本版本的 ElasticSearch 需要的 JSK 版本要是 1.8 以上，所以安装 ElasticSearch 之前保证 JDK1.8+ 安装完毕，并正确的配置好 JDK 环境变量，否则启动 ElasticSearch 失败。

官方地址：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)（国内访问比较慢）

中文地址：[https://www.elastic.co/cn/downloads/elasticsearch](https://www.elastic.co/cn/downloads/elasticsearch)（推荐）

## window 下安装使用

### 1、解压 window 的压缩包

![01.png](/static/images/20211120/01.png)

文件夹说明

```
bin：启动文件
config：配置文件
	log4j2.properties：日志配置文件
	jvm.options：java虚拟机的配置
	elasticsearch.yml：es的配置文件
data：索引数据目录
lib：相关类库Jar包
logs：日志目录
modules：功能模块
plugins：插件
```

### 2、双击 bin 目录中的 elasticsearch.bat 启动

![02.png](/static/images/20211120/02.png)

### 3、验证是否安装成功

浏览器访问：[http://localhost:9200](http://localhost:9200) 得到如下信息，说明安装成功了

![03.png](/static/images/20211120/03.png)

## 安装 ES 图形化界面

Head 是 ElasticSearch 的集群管理工具，可以用于数据的浏览查询，被托管在 Github上面。

地址： [https://github.com/mobz/elasticsearch-head/](https://github.com/mobz/elasticsearch-head/)

> 注意：需要 NodeJS 的环境

### 1、下载 elasticsearch-head-master.zip 并解压

![04.png](/static/images/20211120/04.png)

### 2、解压之后安装依赖

```bash
cnpm install
npm run start
```

这将启动在端口 [http://localhost:9100](http://localhost:9100)  上运行的本地 web 服务器，为 elasticsearch-head 服务！访问测试：

![05.png](/static/images/20211120/05.png)

注意：由于 ES 进程和客户端进程端口号不同，存在跨域问题，所以我们要在 ES 的配置文件中配置下跨域问题

![06.png](/static/images/20211120/06.png)

```bash
# 跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```

## 了解 ELK

ELK 是 ElasticSearch、Logstash、Kibana 三大开源框架首字母大写简称。市面上也被成为 Elastic Stack。其中 ElasticSearch 是一个基于 Lucene、分布式、通过 Restful 方式进行交互的近实时搜索平台框架。像类似百度、谷歌这种大数据全文搜索引擎的场景都可以使用 ElasticSearch 作为底层支持框架，可见 ElasticSearch 提供的搜索能力确实强大，市面上很多时候我们简称 ElasticSearch 为 es。Logstash 是 ELK 的中央数据流引擎，用于从不同目标（文件/数据存储/MQ）收集的不同格式数据，经过过滤后支持输出到不同目的地（文件/MQ/redis/elasticsearch/kafka等）。Kibana 可以将 ElasticSearch 的数据通过友好的页面展示出来，提供实时分析的功能。

市面上很多开发只要提到 ELK 能够一致说出它是一个日志分析架构技术栈总称，但实际上 ELK 不仅仅适用于日志分析，它还可以支持其它任何数据分析和收集的场景，日志分析和收集只是更具有代表性。并非唯一性。

## 安装 Kibana

Kibana 是一个针对 ElasticSearch 的开源分析及可视化平台，用来搜索、查看交互存储在 ElasticSearch 索引中的数据。使用 Kibana，可以通过各种图表进行高级数据分析及展示。Kibana 让海量数据更容易理解。它操作简单，基于浏览器的用户界面可以快速创建仪表板（dashboard）实时显示 ElasticSearch 查询动态。设置 Kibana 非常简单。无需编码或者额外的基础架构，几分钟内就可以完成 Kibana 安装并启动 ElasticSearch 索引监测。

官网：[https://www.elastic.co/cn/kibana](https://www.elastic.co/cn/kibana)

1、下载 Kibana [https://www.elastic.co/cn/downloads/kibana](https://www.elastic.co/cn/downloads/kibana)（注意版本对应关系）

2、将压缩包解压即可

![07.png](/static/images/20211120/07.png)

3、然后进入到 bin目录下，点击 kibana.bat 启动服务

![08.png](/static/images/20211120/08.png)

4、然后访问 [http://localhost:5601](http://localhost:5601) ，kibana 会自动去访问 9200，也就是 ElasticSearch 的端口号（当然 ElasticSearch 这个时候必须启动着），然后就可以使用 kibana了！

![09.png](/static/images/20211120/09.png)

5、界面默认是英文的，可配置为中文的！

中文包在 `kibana\x-pack\plugins\translations\translations\zh-CN.json`，只需要在配置文件 `config/kibana.yml` 中加入

```bash
i18n.locale: "zh-CN"
```

6、重启查看效果，成功切换为中文的了！

![10.png](/static/images/20211120/10.png)