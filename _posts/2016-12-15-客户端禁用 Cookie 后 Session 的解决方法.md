---
layout: post
title: 客户端禁用 Cookie 后 Session 的解决方法
categories: PHP
---

如果禁用cookie理论上来讲, 会影响session: **session基于cookie实现**, session的SESSID利用cookie保存.

但是如果禁用了cookie之后, session就不能用了. 如果有用户恶意(选择)禁用cookie: 可以理解为用户不愿意使用我们的网站.

**如果非要实现: 可以通过a链接实现**

## 一、手动a链接: 用户自己给a链接的href

session_name(): 获取或者设置session名字(没有参数就是获取,有参数就是设置)
session_id(): 获取或者设置sessionID(没有就是获取,有就是设置)

### 1. 在保存session数据界面, 获取session信息(name和id), 嵌入到URL中.

```
// 保存session数据

// 开启session
session_start();

// 获取session信息
$name = session_name();
$id = session_id();

// 保存session数据
$_SESSION['name'] = 'jason';

// 给出一个链接
echo "<a href='session.php?{$name}={$id}'>点我</a>";
```

### 2. 在需要共享session数据的界面: 先获取session信息, 然后再获取session数据

```
// 获取session数据

// 改变session_start从cookie获取id的机制
// 接受数据
$id = $_GET[session_name()];

// 修改session_start的开启机制: 使用当前制定SESSID
session_id($id); // 注册sessionid

// 开启session
session_start(); // 不再从cookie获取，也不会自动创建新的

// 获取
print_r($_SESSION); // Array ( [name] => jason )
```

浏览器输出效果:

![01.png](/static/images/2016/12/15/01.png)

## 二、自动a链接: 通过修改session默认的机制(PHP配置文件php.ini)

### 1. 修改session默认的只允许使用cookie的原则

```
session.use_only_cookies = 1 改为 session.use_only_cookies = 0
```

![02.png](/static/images/2016/12/15/02.png)

### 2. 开启URL转换sessionid信息的配置

```
session.use_trans_sid = 0 改为 session.use_trans_sid = 1
```

![03.png](/static/images/2016/12/15/03.png)

以上修改完了之后: 系统会自动在a标签中加入sessionid信息, 而且还能在获取session数据界面自动的从a标签去获取sessionid信息

**效果1: 所有的a标签都会自动加上session信息**

```
// 保存session数据

// 开启session
session_start();

// 保存session数据
$_SESSION['name'] = 'jason';

// 给出一个链接
echo "<a href='session.php'>点我</a>";
```

![04.png](/static/images/2016/12/15/04.png)

**效果2: session使用的时候,会自动的从a标签读取session信息**

```
// 获取session数据

// 开启session
session_start(); // 不再从cookie获取，也不会自动创建新的

// 获取
print_r($_SESSION); // Array ( [name] => jason )
```

![05.png](/static/images/2016/12/15/05.png)