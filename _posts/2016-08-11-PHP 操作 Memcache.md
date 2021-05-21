---
layout: post
title: PHP 操作 Memcache
categories: Memcache
taps: Memcache,PHP
---

## 一、先安装memcache的扩展，让php支持。

1.准备php支持的扩展文件，要注意要和php的版本对应。

![01.png](/static/images/2016/08/11/01.png)

2.把扩展文件拷贝到php的安装目录下面的ext目录里面。

![02.png](/static/images/2016/08/11/02.png)

3.打开php.ini的配置文件，引入扩展，重启apache。

![03.png](/static/images/2016/08/11/03.png)

4.使用phpinfo 函数测试是否引入成功。

![04.png](/static/images/2016/08/11/04.png)

## 二、php操作memcache简单案例

```
// 实例化一个memcache的类
$mem = new Memcache;
// 连接memcache的服务器
$mem->connect('localhost', 11211);
// 设置数据
$mem->set('name', 'jason', 0, 60); // $mem->set(key,value,是否压缩,失效时间);
$val = $mem->get('name');
var_dump($val);
// 替换数据
$mem->replace('name', 'laura', 0, 60); // $mem->replace(key,value,是否压缩,失效时间);
$val = $mem->get('name');
var_dump($val);
// 添加数据
$mem->add('age', '24', 0, 60); // $mem->add(key,value,是否压缩,失效时间);
$val = $mem->get('age');
var_dump($val);
// 删除数据
$mem->delete('age'); // $mem->delete(key);
$val = $mem->get('age');
var_dump($val);
// 清除所有数据
$mem->flush();
// 关闭连接
$mem->close();

/*输出结果：
string(5) "jason" string(5) "laura" string(2) "24" bool(false) bool(false)*/
```

## 三、php的memcache客户端所有方法总结

memcache函数所有的方法列表如下：

| 方法 | 作用 |
| -- | -- |
| Memcache::add | 添加一个值，如果已经存在，则返回false |
| Memcache::addServer | 添加一个可供使用的服务器地址 |
| Memcache::close | 关闭一个Memcache对象 |
| Memcache::connect | 创建一个Memcache对象 |
| memcache_debug | 控制调试功能 |
| Memcache::decrement | 对保存的某个key中的值进行减法操作 |
| Memcache::delete | 删除一个key值 |
| Memcache::flush | 清除所有缓存的数据 |
| Memcache::get | 获取一个key值 |
| Memcache::getExtendedStats | 获取进程池中所有进程的运行系统统计 |
| Memcache::getServerStatus | 获取运行服务器的参数 |
| Memcache::getStats | 返回服务器的一些运行统计信息 |
| Memcache::getVersion | 返回运行的Memcache的版本信息 |
| Memcache::increment | 对保存的某个key中的值进行加法操作 |
| Memcache::pconnect | 创建一个Memcache的持久连接对象 |
| Memcache::replace | 对一个已有的key进行覆写操作 |
| Memcache::set | 添加一个值，如果已经存在，则覆写 |
| Memcache::setCompressThreshold | 对大于某一大小的数据进行压缩 |
| Memcache::setServerParams | 在运行时修改服务器的参数 |