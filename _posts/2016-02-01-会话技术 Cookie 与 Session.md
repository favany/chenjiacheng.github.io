---
layout: post
title: 会话技术 Cookie 与 Session
categories: PHP
taps: PHP,Cookie,Session
---

会话: 在一次联通过程中(浏览器没有关闭),所进行的多次请求(请求了多个脚本)

会话技术: 在浏览器不关闭的情况下, 对一个网站进行多次访问, 整个的访问过程称之为一次会话. 因为HTTP协议的无状态和无连接, 导致虽然浏览器没有关闭,但是在多个脚本之间请求的时候, 脚本之间无法实现数据的共享: 会话技术就是为了解决在浏览器进行多次请求的过程中,能够实现数据跨脚本共享.

## 会话技术分类

会话技术因为有两个不同的技术,实现的目标也不怎么一致: 整个将会话技术分为两类.

* cookie技术: cookie是一种服务器将数据保存到浏览器, 然后浏览器能够将数据重新携带回服务器,从而实现服务器识别浏览器的多次请求的技术.
* session技术: session是一种将数据保存在服务器端,实现跨脚本共享数据的技术. session技术依赖cookie存在.

## cookie技术

cookie技术: 将数据保存到浏览器的技术.

cookie技术的实现: HTTP协议中提供了一对cookie操作协议

* set-cookie: HTTP响应, 允许服务器把数据通过HTTP协议携带给浏览器.
* cookie: HTTP请求, 允许浏览器将服务器保存的数据携带给服务器.

### 1. header设置cookie

header函数用来修改HTTP协议的: HTTP响应

header('set-cookie:cookie名字=值');

~~~
// 直接设置cookie
header('set-cookie:name=jason'); // 设置一个名字为name的cookie，值为jason
~~~

查看浏览器保存的cookie

![01.png](/static/images/2016/02/01/01.png)

验证: 会话技术cookie会失效(被浏览器清除): 关闭浏览器再重新打开

![02.png](/static/images/2016/02/01/02.png)

header没有办法很好的控制cookie, header可以实现任何http协议的修改.

### 2. setcookie函数

PHP提供了一个专门设置cookie的函数: setcookie

setcookie(名字,值);

~~~
// 直接设置cookie
setcookie('name','jason'); // 设置一个名字为name的cookie，值为jason
~~~

### 3. 获取cookie

浏览器能够在访问同一个网站的时候,会自动将以前的cookie数据携带给服务器.

Apache不能获取cookie数据, 只能PHP来做.

PHP会将所有的cookie数据(浏览器携带的)保存到超全局预定义数组中: $_COOKIE

~~~
// $_COOKIE自动接收
var_dump($_COOKIE);
~~~

### 4. cookie生命周期

生命周期: cookie会在什么时间内失效(失效的cookie才会被浏览器清除)

默认的: cookie的生命周期是会话技术(浏览器关闭)

setcookie函数的第三个参数: 就是用来设置cookie的生命周期: 没有给定第三个参数, 是因为第三个参数有默认值: 0, 就代表会话技术(浏览器关闭)

第三个参数是时间戳: 生命长度

~~~
// 设置cookie(默认生命周期)
setcookie('cookie','cookie');

// 显示的指定生命周期为会话结束
setcookie('default','default',0);

// 指定生命周期：10秒之后就过期
setcookie('ten',10,10);
// 错误: 代表早在1970年1月1日,0点0分10秒过期

// 合理10秒
setcookie('active_ten',10,time() + 10);

var_dump($_COOKIE);
/*输出结果证明: 10秒的cookie已经过期: 获取一次
array (size=4)
  'name' => string 'jason' (length=5)
  'cookie' => string 'cookie' (length=6)
  'default' => string 'default' (length=7)
  'active_ten' => string '10' (length=2)*/
~~~

### 5. cookie作用范围

cookie作用域: 默认的, cookie只针对自己所在目录及其子目录生效, 上级目录不能访问.

通常的解决方案是: cookie整站有效: 网站根目录: setcookie的第四个参数.

~~~
// 设置一个简单cookie：当前目录有效
setcookie('local','local',0);

// 设置全局cookie：整站有效
setcookie('global','global',0,'/');
~~~

### 6. cookie跨域

cookie对不同的主机名可以共享, 但是默认的不支持.

域名: Domain Name, IP地址的别名

顶级域名: .com / .cn / .net / .org

一级域名: 在顶级域名的左边,增加一个字段: baidu.com / itcast.cn /apache.org / php.net

二级域名: 在一级域名又增加一个字段: www.itcast.cn / gz.itcast.cn / bj.itcast.cn

cookie跨域: cookie可以在不同的二级域名下共享数据. 通过setcookie的第五个参数来实现.指定方式是共享的一级域名

~~~
// cookie跨域
setcookie('wwwglobal','global',0,'/','shop.com');
// 表示只要是以shop.com结尾的主机都可以共享当前数据
~~~

### 7. cookie特性

cookie特性主要是从$_COOKIE中处理

* cookie设置是通过setcookie函数实现: 没有办法存储数组: setcookie不论名字还是值都只能是普通数据(字符串数据)
* $_COOKIE永远是一维数组: 但是可以想办法让$_COOKIE变成二维数组(不是手动修改)
* 在服务器端(PHP)不要修改$_COOKIE里面的内容: 修改无效

## session技术

session技术: 将数据保存在服务器端,实现跨脚本共享数据.

session技术依赖cookie实现.

session默认的将数据保存到文件中: 文件不会随着脚本执行的结束而删除, 就可以实现将数据放到文件中, 下一个脚本又从文件中读取数据.

![03.png](/static/images/2016/02/01/03.png)

session文件的保存位置: 默认没有指定,就是操作系统的临时目录(C:/windows/temp)

![04.png](/static/images/2016/02/01/04.png)

### 1. session原理

session在PHP中有一套内部机制, 只需要用户(程序员)去简单的触发即可.

![05.png](/static/images/2016/02/01/05.png)

### 2. 实现session

session要使用: 基本上使用手动模式

**(1)激活session系统: 开启session: session机制对外提供了一个操作接口: 函数session_start()**

~~~
// 开启session
session_start();
开启session后，系统不管用户存不存东西，都会给一把"钥匙"
~~~

![06.png](/static/images/2016/02/01/06.png)

**(2)存储数据: 系统提供了一个专门存储数据的容器,只要用户将数据保存到对应的容器中,系统就会自动的将容器内部的内容,保存到session文件中. $_SESSION**

~~~
// 将数据存放到容器中
$_SESSION['name'] = 'jason';
~~~

![07.png](/static/images/2016/02/01/07.png)

注意: session序列化是选择性序列化: 不会对$_sESSION中的下标序列化,只对值进行序列化

**(3)使用session数据: 跨脚本: 在一个新的脚本中,拿着”钥匙”开锁后就可以取出里面的内容.**

~~~
// 访问$_SESSION
var_dump($_SESSION);
/*输出结果
array (size=1)
  'name' => string 'jason' (length=5)*/
注意: 即便只是访问$_SESSION,系统提示session文件被修改
~~~

### 3. session实现具体原理

session机制内部有很多功能, 内部自己协调.

开启session: session_start()

* 激活session: session机制开始工作
* 获取sessionID: PHPSESSID
    * 第一次: 系统自动生成,而且会将sessionID保存到cookie
    * 第二次: 从cookie中获取sessionID
* 初始化变量$_SESSION: 空数组
* 系统会拿着sessionID取到session文件夹寻找对应的session文件(没有就创建一个)
* 将session文件中保存的数据进行反序列化保存到$_SESSION中
* session系统会设定一个监听程序: 监听脚本执行结束

使用session: $_SESSION

脚本执行结束: session机制监听到脚本要结束了: 将$_SESSION中的数据进行序列化,然后写入到sessionID对应的session文件.

![08.png](/static/images/2016/02/01/08.png)

证明: session在开启之后, 一个脚本中一定会对session文件操作两次

### 4. 删除session

删除session指的是删除session对应的文件.

session_destroy(): 删除当前sessionID对应的session(凡是要操作session需要先开启session)

~~~
// 开启session
session_start();

// 删除session
session_destroy();
~~~

### 5. session常用配置

session的表现特性都是由php.ini中进行配置实现的(大部分都是用默认配置)

(1)session默认是必须使用cookie实现

![09.png](/static/images/2016/02/01/09.png)

(2)session名字: cookie对应的sessionID名字叫做: PHPSESSID

![10.png](/static/images/2016/02/01/10.png)

(3)session默认是需要手动开启, 自动session关闭掉了

![11.png](/static/images/2016/02/01/11.png)

使用自动session: 不需要手动开启session(session_start()不用了)

因为不是所有的地方都需要使用session: session会产生session文件占用服务器的磁盘空间(从来不用自动session)

(4)session的生命周期其实受cookie的影响: 默认的cookie的生命周期是会话结束

![12.png](/static/images/2016/02/01/12.png)

(5)session对应的ID(cookie)的作用范围都是网站根目录

![13.png](/static/images/2016/02/01/13.png)

(6)session数据的序列化方式使用PHP的序列化实现.

![14.png](/static/images/2016/02/01/14.png)

(7)session垃圾回收机制: 一旦触发垃圾回收,系统会自动一次性清理所有的过期的session

![15.png](/static/images/2016/02/01/15.png)

(8)session有自己的生命周期: 1440秒

![16.png](/static/images/2016/02/01/16.png)

(9)session可以通过配置实现将sessionid信息绑定到URL(a标签中)

![17.png](/static/images/2016/02/01/17.png)

### 6. session特点

session是利用$_SESSION来进行数据保存.

$_SESSION中不允许使用索引下标 