---
layout: post
categories: ThinkPHP
---

hasOne 和 belongsTo 在模型中都是一对一关联，那么区别是什么呢？

## hasOne：外键在子关联对象中

可以理解为 `包含` 的意思，参数如下：

```
hasOne('关联模型名','外键名','主键名',['模型别名定义'],'join类型');
```

默认的 join 类型为 INNER。

## belongsTo：外键在你父联对象中

可以理解为 `属于` 的意思，参数如下：

```
belongsTo('关联模型名','外键名','关联表主键名',['模型别名定义'],'join类型');
```

## hasOne 和 belongsTo 使用示例

例如我们有两张表

* user 表字段：`id` `name` `password`

* user_address 表字段：`id` `user_id` `address`

**在 User 模型中关联 user_address 表的时候使用 hasOne，因为在 user 表中没有关联两个表的外键**

```
<?php

namespace app\index\model;

use think\Model;

class User extends Model
{
    public function userAddress()
    {
        return $this->hasOne('UserAddress', 'user_id', 'id');
    }
}

```

**在 UserAddress 模型中关联 user 表的时候使用 belongsTo，因为在 user_address 表中有关联两个表的外键 user_id**

```
<?php

namespace app\index\model;

use think\Model;

class UserAddress extends Model
{
    public function user()
    {
        return $this->belongsTo('User', 'user_id', 'id');
    }
}

```

