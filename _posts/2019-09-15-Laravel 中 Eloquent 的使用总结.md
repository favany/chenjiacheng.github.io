---
layout: post
categories: PHP
tags: [PHP,Laravel]
---

## 创建 Eloquent 模型

`Artisan` 命令

```
php artisan make:model Order
```

为模型创建一个迁移

```
php artisan make:model Order --migration
```

创建成功后看到 `app/Order.php`

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    //
}
```

## Eloquent 相关属性

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Order extends Model
{
    // 开启软删除
    use SoftDeletes;

    // 设置表名，默认 orders
    protected $table = 'orders';

    // 设置主键，默认 id
    protected $primaryKey = 'id';
    public $incrementing = false; // 取消主键自增

    // 时间戳
    const CREATED_AT = 'created_at';
    const UPDATED_AT = 'updated_at';
    public $timestamps = false; // 关闭 created_at 和 updated_at 时间戳列
    protected $dateFormat = 'U'; // 自定义时间戳类型

    // 日期转换
    protected $dates = [
        'created_at', // 默认
        'updated_at', // 默认
        'pay_time',
        'deleted_at'
    ];

    // 设置可填充字段
    protected $fillable = [
        'name',
        'phone',
    ];

    // 设置防护字段
    protected $guarded = [
        'id',
        'created_at',
        'updated_at',
    ];

    // 属性转换器
    protected $casts = [
        'id'          => 'int',
        'name'        => 'string',
        'total_price' => 'double',
    ];

    // 从 JSON 中隐藏属性
    public $hidden = ['client', 'remark'];

    // 从 JSON 中指定显示属性
    public $visible = ['name', 'phone', 'order_status'];
}
```

**可能的属性转换器列类型**

| 类型 | 描述 |
| -- | -- |
| int \| integer | 通过 PHP 转换（int） |
| real \| float \| double | 通过 PHP 转换（float） |
| string | 通过 PHP 转换（string） |
| bool \| boolean | 通过 PHP 转换（bool） |
| object | 作为一个 stdClass 对象，从 JSON 解析或被解析为 JSON |
| array | 作为一个数组，从 JSON 解析或被解析为 JSON |
| collection | 作为一个集合，从 JSON 解析或被解析为 JSON |
| date \| datetime | 从数据库 DATATIME 解析为 Carbon 类型，然后返回 |
| timestamp | 从数据库 TIMESTAMP 解析为 Carbon 类型，然后返回 |

## Eloquent 相关操作

### 获取数据

```
// 获取单个数据
Order::where('id', 1)->first();

// 获取多个数据
Order::get(); // 建议
Order::all();

// 用 chunk() 进行响应分块
Order::chunk(100, function ($orders) {
    foreach ($orders as $order) {
        // code ...
    }
});

// 聚合
Order::count();
Order::sum('total_price');
Order::avg('total_price');

// 序列化
$orderArray = Order::all()->toArray();
$orderJson = Order::all()->toJson();
```

### 插入和更新

```
// 插入
$order = new Order();
$order->name = 'jason';
$order->phone = '13800138000';
$order->save();

$order = new Order([
    'name'  => 'jason',
    'phone' => '13800138000',
]);
$order->save();

$order = new Order::create([
    'name'  => 'jason',
    'phone' => '13800138000',
]);

// 更新
$order = Order::find(1);
$order->name = 'jason';
$order->phone = '13800138000';
$order->save();

$order = Order::find(1);
$order->update([
    'name'  => 'jason',
    'phone' => '13800138000',
]);

// 查询如果不存在就新建
$order = Order::firstOrCreate(['name' => 'jason']);
$order = Order::firstOrNew(['name' => 'jason']);
```

### 删除数据

```
// 删除
$order = Order::find(1);
$order->delete();

Order::destroy(1); // 通过ID删除
Order::destroy([1, 2, 3]); // 删除多个

// 软删除相关
$all = Order::withTrashed()->get(); // 查询包括软删除

// 判断是否已经被软删除
if ($order->trashed()) {
    $order->restore(); // 从软删除中恢复实体
    $order->forceDelete(); // 强制删除软删除实体
}
```

## 访问器和修改器

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    // 访问器（存在的属性）
    public function getNameAttribute($value)
    {
        return $value ?: 'xx';
    }

    // 访问器（不存在的属性）
    public function getFullNameAttribute()
    {
        return $this->first_name . ' ' . $this->last_name;
    }

    // 修改器（存在的属性）
    public function setAmountAttribute($value)
    {
        $this->attributes['amount'] = $value > 0 ? $value : 0;
    }

    // 修改器（不存在的属性）
    public function setWorkgroupNameAttribute($workgroupName)
    {
        $this->attributes['email'] = "{$workgroupName}@qq.com";
    }
}
```

## Eloquent 作用域

### 本地作用域

**定义**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    // 本地作用域
    public function scopeDefaultStatus($query)
    {
        return $query->where('status', 1);
    }
    
    // 本地作用域（传参）
    public function scopeStatus($query, $status)
    {
        return $query->where('status', $status);
    }
}
```

**使用**

```
$orders = Order::defaultStatus()->get();
$orders = Order::status(1)->get();
```

### 全局作用域

**定义**

```
<?php

namespace App;

use App\Scope\StatusScope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    protected static function boot()
    {
        parent::boot();

        // 定义全局作用域
        static::addGlobalScope('status', function (Builder $builder) {
            $builder->where('status', 1);
        });
        
        // 或者使用全局作用域类
        static::addGlobalScope(new StatusScope);
    }
}
```

创建全局作用域类 `app/Scope/StatusScope.php`

```
<?php

namespace App\Scope;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class StatusScope implements Scope
{
    public function apply(Builder $builder, Model $model)
    {
        return $builder->where('status', 1);
    }
}
```

**使用**

```
// 关闭全局作用域
$orders = Order::query()->withoutGlobalScope('status')->get();
$orders = Order::query()->withoutGlobalScope(StatusScope::class)->get();
$orders = Order::query()->withoutGlobalScopes([StatusScope::class])->get(); // 关闭多个全局作用域
$orders = Order::query()->withoutGlobalScopes()->get(); // 关闭所有的全局作用域
```

## Eloquent 集合

### 基本使用

```
$orders = Order::all();

$names = $orders
    // 移除 status 为 null 的数据
    ->reject(function ($item) {
        return $item->status === null;
    })
    // 返回 name 字段集合
    ->map(function ($item) {
        return $item->name;
    });

dd($names);
```

### 自定义集合

**定义**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    public function newCollection(array $models = [])
    {
        return new OrderCollection($models);
    }
}
```

创建自定义集合类 `app/Collection/OrderCollection.php`

```
<?php

namespace App\Collection;

use Illuminate\Support\Collection;

class OrderCollection extends Collection
{
    public function sumBillableAmount()
    {
        return $this->reduce(function ($carry, $order) {
            return $carry + ($order->billable ? $order->amount : 0);
        }, 0);
    }
}
```

**使用**

```
$order = Order::all();
$billableAmount = $order->sumBillableAmount();

dd($billableAmount);
```

## Eloquent 关联模型

### 一对一

**`hasOne` 用法**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    public function orderDetail()
    {
        return $this->hasOne(OrderDetail::class, 'order_id', 'id');
    }
}
```

```
$order = Order::first();
$orderDetail = $order->orderDetail; // 或者 $users->order_detail;

dd($orderDetail);
```

**`belongsTo` 用法**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class OrderDetail extends Model
{
    public function order()
    {
        return $this->belongsTo(Order::class, 'order_id', 'id');
    }
}
```

```
$orderDetail = OrderDetail::first();
$order = $orderDetail->order;

dd($order);
```

**插入关联条目**

```
$order = Order::create(['name' => 'jason']);

$orderDetail = new OrderDetail();
$orderDetail->content = '11';
$order->orderDetail()->save($orderDetail);

// 或

$order->orderDetail()->saveMany([
    OrderDetail::find(1), // 修改对应数据的关联ID
    OrderDetail::find(2), // 修改对应数据的关联ID
]);

// 或

$order->orderDetail()->create([
    'content' => '22'
]);
```

### 一对多

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    public function orderGoods()
    {
        return $this->hasMany(OrderGoods::class, 'order_id', 'id');
    }
}
```

```
$order = Order::first();
$orderGoods = $order->orderGoods; // 或者 $users->order_goods;

dd($orderGoods);
```

**返回时一个集合而不是模型的实例，所以可以直接使用 Eloquent 集合操作**

```
$seckills = $order->orderGoods->filter(function ($orderGoods) {
    return $orderGoods->goods_type === 'seckill';
});

$total = $order->orderGoods->reduce(function ($carry, $orderGoods) {
    return $carry + $orderGoods->money;
});
```

**定义一对多反向关系**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class OrderGoods extends Model
{
    public function order()
    {
        return $this->belongsTo(Order::class, 'order_id', 'id');
    }
}

```

**从附加内容中附加或分离关联内容**

```
$orderGoods = OrderGoods::first();

// 附加
$orderGoods->user()->associate(Order::first());
$orderGoods->save();

// 分离
$orderGoods->user()->dissociate();
$orderGoods->save();
```

**将关系用作查询构造器**

```
$seckills = $order->orderGoods()->where('goods_type', 'seckill')->get();
```

**仅查询有关连条目的记录**

```
$order = Order::has('orderGoods')->get();

// 关联数量 >= 5
$order = Order::has('orderGoods', '>=', 5)->get();

// orderGoods 关联 goods 中有记录
$order = Order::has('orderGoods.goods')->get();

// 自定义查询条件
$order = Order::whereHas('orderGoods', function ($query) {
    $query->where('goods_type', 'seckill');
})->get();
```

### 远程一对多关系

假定 `order_goods` 表的 `order_id` 列用于与 `order` 表进行关联，而 `cart` 表的 `order_goods_id` 用于与 `order_goods` 表关联

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Order extends Model
{
    public function goods()
    {
        return $this->hasManyThrough(Cart::class, OrderGoods::class);
    }
}
```

### 多对多

假定 `user` 表和 `contact` 表是多对多关系，则需要有个透视表 `contacts_users` 存储，字段为 `contact_id` 和 `user_id`

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function contacts()
    {
        return $this->belongsToMany(Contact::class);
    }
}
```

```
$user = User::first();

$user->contacts->each(function ($contact) {
    // code ...
});

$contact = $user->contacts()->where('contact_name', 'jason')->get();
```

**多对多关系内容中附加和分离的独特性**

```
$user = User::first();
$contact = Contact::first();

// 添加内容
$user->contacts()->save($contact, ['status' => 1]);

// 附加
$user->contacts()->attach(2); // 2 为 contact 表中的ID
$user->contacts()->attach(3, ['status' => 1]);
$user->contacts()->attach([1, 2, 3]);

// 分离
$user->contacts()->detach(2); // 2 为 contact 表中的ID
$user->contacts()->detach([1, 2]);
$user->contacts()->detach(); // 分离所有

// 修改透视表中的内容
$user->contacts()->updateExistingPivot(2, [ // 2 为 contact 表中的ID
    'status' => 2,
]);

// 清空之前的，并添加新的
$user->contacts()->sync(2); // 2 为 contact 表中的ID
$user->contacts()->sync(3, ['status' => 1]);
$user->contacts()->sync([1, 2, 3]);
```

**从透视表中获取数据**

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function contacts()
    {
        return $this->belongsToMany(Contact::class)
            ->withTimestamps() // 返回 created_at 和 updated_at 字段
            ->withPivot('status', 'type'); // 返回自定义字段
    }
}
```

### 多态

以 `Stars`（收藏） 为例，一个用户可以给 `Articles` 和 `Comments` 收藏，`articles` 和 `comments` 就是普通的表，`stars` 表会包含 `id`、`starrable_id`、`starrable_type` 字段

**创建模型**

```
class Star extends Model
{
    public function starrable()
    {
        return $this->morphTo();
    }
}

class Article extends Model
{
    public function stars()
    {
        return $this->morphMany(Star::class, 'starrable');
    }
}

class Comment extends Model
{
    public function stars()
    {
        return $this->morphMany(Star::class, 'starrable');
    }
}
```

**添加数据**

```
$article = Articles::first();

$article->stars()->create();
```

**获取数据**

```
$article = Articles::first();

$article->stars->each(function ($star) {
    // code ...
});
```

```
$stars = Stars::all();

$stars->each(function ($star) {
    var_dump($star->starrable);
});
```

### 多对多的多态

```
class Article extends Model
{
    public function stars()
    {
        return $this->morphMany(Stars::class, 'starrable');
    }
}

class Comment extends Model
{
    public function stars()
    {
        return $this->morphMany(Stars::class, 'starrable');
    }
}

class Star extends Model
{
    public function articles()
    {
        return $this->morphedByMany(Articles::class, 'starrable');
    }
    
    public function comments()
    {
        return $this->morphedByMany(Comments::class, 'starrable');
    }
}
```

**添加数据**

```
$star = Stars::firstOrCreate(['name' => 'xx']);
$aricle = Article::first();
$aricle->stars()->attach($star->id);
```

**获取数据**

```
$aricle = Article::first();

$aricle->stars->each(function ($star) {
    // code ...
});
```

```
$stars = Stars::all();

$stars->articles->each(function ($star) {
    // code ...
});
```

### 通过子类更新父类时间戳

```
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class OrderDetail extends Model
{
    protected $touches = ['order'];

    public function order()
    {
        return $this->belongsTo(Order::class, 'order_id', 'id');
    }
}
```

### 预载入

```
$order = Order::with('orderGoods')->get();

// 带条件的预载入
$order = Order::with(['orderGoods' => function ($query) {
    $query->where('goods_type', 'seckill');
}])->get();

// 懒惰的预载入
$order = Order::all();
$order->load('orderGoods');

// 仅预载入计数
$order = Order::withCount('orderGoods')->first();
dd($order->order_goods_count);
```