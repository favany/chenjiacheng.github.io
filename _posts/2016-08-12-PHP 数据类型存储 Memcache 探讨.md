---
layout: post
title: PHP 数据类型存储 Memcache 探讨
categories: Memcache
taps: Memcache,PHP
---

## 一、标量类型：整型 浮动型 布尔 字符串

```
// 实例化一个memcache的类
$mem = new Memcache();
// 连接memcache的服务器
$mem->connect('localhost', 11211);
// 设置数据
$mem->set('int',100,0,3600);
$mem->set('float',10.20,0,3600);
$mem->set('boolean',false,0,3600);
$mem->set('string',"welcome to guangzhou",0,3600);
// 获取数据
var_dump($mem->get('int'));
var_dump($mem->get('float'));
var_dump($mem->get('boolean'));
var_dump($mem->get('string'));

/*输出结果：
string(3) "100" string(4) "10.2" string(0) "" string(20) "welcome to guangzhou"*/
```

说明：标量类型是可以存储到memcache里面的，都是以字符串的形式存储的，最后输出也变成了字符串。

## 二、非标量类型：数组 对象 资源 null

```
// 实例化一个memcache的类
$mem = new Memcache();
// 连接memcache的服务器
$mem->connect('localhost', 11211);
// 设置数据
$mem->set('array',array('apple','orange','tree'),0,3600);
class dog{
}
$dog = new dog();
$dog->name='哮天犬';
$dog->age=4000;
$mem->set('object',$dog,0,3600);
$conn = mysql_connect('localhost','root','123');
$mem->set('resource',$conn,0,3600);
$mem->set('null',null,0,3600);
// 获取数据
var_dump($mem->get('array'));
var_dump($mem->get('object'));
var_dump($mem->get('resource'));
var_dump($mem->get('null'));

/*输出结果：
array(3) { [0]=> string(5) "apple" [1]=> string(6) "orange" [2]=> string(4) "tree" }
object(dog)#3 (2) { ["name"]=> string(9) "哮天犬" ["age"]=> int(4000) }
int(0)
NULL*/
```

**数组 对象 资源 null在 memcache 里面存储的形式**

![01.png](/static/images/2016/08/12/01.png)

说明：

数组 对象 资源 是以序列化之后的结果存储到memcache里面的。

但是在取出数据时，又自动反序列化之后显示的。

序列化与反序列化的过程是由memcache的客户端完成的，无需我们自己干预。

说明则memcache里面存储的数据，是以字符串的形式来存储的。

注意：不能把资源类型存储到memcache里面，因为在取出资源类型时，把资源类型变成了整型。在实际应用中，存储数组的情况居多。