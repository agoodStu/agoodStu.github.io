---
layout: post
title: MySQL笔试题练习
date: 2019-10-28
categories: blog
tags: [MySQL, 数据分析]
---

## MySQL笔试题练习

> 三天不练习，SQL更垃圾。

这篇文章里面准备放一下MySQL的笔试题。这些题目都是从网上看到的。

### Pratice 1

> 用户注册表table_user，包含两个字段：user_id(用户id)、reg_tm(注册时间)。订单表table_order，包含三个字段：order_id(订单号)、order_tm(下单时间)、user_id(用户id)。
>
> 查询从2019年1月1日至今，每天的注册用户数，下单用户数，以及注册当天即下单的用户数（请尽量在一个 sql语句中实现）。
>
> 原题链接：<https://zhuanlan.zhihu.com/p/70400671>

##### 思路分析：

- 需要每天的注册用户数，那就需要以reg_tm进行group by。
- 需要连接两个表，虽然两张表都有user_id，但不能以该字段连接。因为两个表reg