---
layout: post
title: MySQL IF 和 CASE 的用法
categories: MySQL
---

```
select IF(sex = 1, '男', '女') as sex from ecs_users where user_id = 1;

select CASE sex WHEN 1 THEN '男' ELSE '女' END as sex from ecs_users where user_id = 1;

SELECT SUM(og.goods_number * IF(g.give_integral > -1, g.give_integral, og.goods_price)) AS custom_points, 
SUM(og.goods_number * IF(g.rank_integral > -1, g.rank_integral, og.goods_price)) AS rank_points 
FROM `maiquwang`.`ecs_order_goods` AS og, `maiquwang`.`ecs_goods` AS g 
WHERE og.goods_id = g.goods_id AND og.order_id = '10' AND og.goods_id > 0 AND og.parent_id = 0 AND og.is_gift = 0 AND og.extension_code != 'package_buy';
```