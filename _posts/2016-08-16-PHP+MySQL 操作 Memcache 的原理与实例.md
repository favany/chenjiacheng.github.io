---
layout: post
title: PHP+MySQL 操作 Memcache 的原理与实例
categories: Memcache
taps: PHP,MySQL,Memcache
---

## 一、实现原理

memcache是一个高性能的分布式内存对象缓存系统，原理如下图：

![01.png](/static/images/2016/08/16/01.png)

首次访问：从数据库中取得数据保存到服务器内存中

再次访问：从服务器内存中取得数据返回

**总结：apache服务器没有能力把数据保存到内存中，所以要借助memcache完成。**

## 二、实例操作

想把一个sql语句的执行结果，给缓存到memcache里面。要注意说明的，sql语句执行的结果数据要小于1MB。

在memcache里面，键与值是有要求的，键的长度要小于250字节。数据值的大小要小于1MB。

```
// 实例化一个memcache的类
$mem = new Memcache();
// 连接memcache的服务器
$mem->connect("localhost",11211);
// 获取传递过来的id
$id = intval($_GET['id']);
// 构建存储到memcache里面的键
$key = 'news_'.$id;
// 使用该键从memcache里面获取数据
$data = $mem->get($key);
// 判断是否获取数据成功，如果成功，就直接显示数据。
// 如果没有成功，则查询数据库，并写入到memcache缓存里面。
if(!$data){
    $conn = mysql_connect('localhost','root','123');
    mysql_query('use test');
    mysql_query('set names utf8');
    $sql = "select * from news where id = $id";
    $res = mysql_query($sql);
    $data = mysql_fetch_assoc($res);
    // 把查询出的数据，给存储到memcache里面
    $mem->set($key,$data,0,3600);
}

var_dump($data);
```

ps：如果存到内存中的数据有修改，则清除缓存即可。