---
layout: post
categories: PHP
---

如果禁用 Cookie 理论上来讲，会影响 Session: **Session 基于 Cookie 实现**，Session 的 `SESSID` 利用 Cookie 保存。

但是如果禁用了 Cookie 之后，Session 就不能用了。如果有用户恶意（选择）禁用 Cookie： 可以理解为用户不愿意使用我们的网站。

**如果非要实现: 可以通过 a 链接实现**

## 手动 a 链接: 用户自己给 a 链接的 href

`session_name()`：获取或者设置 Session 名字（没有参数就是获取，有参数就是设置）
`session_id()`：获取或者设置 `sessionID`（没有就是获取，有就是设置）

### 在保存 Session 数据界面，获取 Session 信息（name 和 id），嵌入到 URL 中

```
// 保存 Session 数据

// 开启 Session
session_start();

// 获取 Session 信息
$name = session_name();
$id = session_id();

// 保存 Session 数据
$_SESSION['name'] = 'jason';

// 给出一个链接
echo "<a href='session.php?{$name}={$id}'>点我</a>";
```

### 在需要共享 Session 数据的界面：先获取 Session 信息，然后再获取 Session 数据

```
// 获取 Session 数据

// 改变 session_start 从 Cookie 获取 id 的机制
// 接受数据
$id = $_GET[session_name()];

// 修改 session_start 的开启机制：使用当前制定 SESSID
session_id($id); // 注册sessionid

// 开启 Session
session_start(); // 不再从 Cookie 获取，也不会自动创建新的

// 获取
print_r($_SESSION); // Array ( [name] => jason )
```

浏览器输出效果:

![01.png](/static/images/20161215/01.png)

## 自动 a 链接：通过修改 Session 默认的机制（PHP 配置文件 php.ini）

### 修改 Session 默认的只允许使用 Cookie 的原则

```
session.use_only_cookies = 1 改为 session.use_only_cookies = 0
```

![02.png](/static/images/20161215/02.png)

### 开启 URL 转换 sessionId 信息的配置

```
session.use_trans_sid = 0 改为 session.use_trans_sid = 1
```

![03.png](/static/images/20161215/03.png)

以上修改完了之后: 系统会自动在 a 标签中加入 `sessionId` 信息, 而且还能在获取 Session 数据界面自动的从 a 标签去获取 `sessionId` 信息

**效果1：所有的 a 标签都会自动加上 Session 信息**

```
// 保存 Session 数据

// 开启 Session
session_start();

// 保存 Session 数据
$_SESSION['name'] = 'jason';

// 给出一个链接
echo "<a href='session.php'>点我</a>";
```

![04.png](/static/images/20161215/04.png)

**效果2：Session 使用的时候，会自动的从 a 标签读取 Session 信息**

```
// 获取 Session 数据

// 开启 Session
session_start(); // 不再从 Cookie 获取，也不会自动创建新的

// 获取
print_r($_SESSION); // Array ( [name] => jason )
```

![05.png](/static/images/20161215/05.png)