---
layout: post
title: 数据分析实战-User Behavior From Taobao-天池
date: 2019-09-09
categories: blog
tags: [数据分析实战]
---

## 数据分析实战——User Behavior Data from Taobao for Recommendation [天池]

> UserBehavior是阿里巴巴提供的一个淘宝用户行为数据集，用于隐式反馈推荐问题的研究。
>
> 下载地址：[戳这里](https://tianchi.aliyun.com/dataset/dataDetail?dataId=649&userId=1)

### 数据导入 

解压后的数据有3.4G，这样基本就告别EXCEL了。所以可以加载到SQL里面观察一下。

先在SQL里面建立一个表用来放数据，字段名设置好。不要使用Navicat直接导入，太慢了。可以使用使用`LOAD DATA INFILE`。在新版的Sql里面，一般会提示secure_file_priv的权限问题。通过`show variables like '%secure%' `，可以看到secure_file_priv的状态，如果为null，则意味着不允许SQL写入和写出。

关于改权限的方式，网上有很多教程，可以参考。我说一下我碰到的坑。目前我使用的SQL是8.0.16解压版。网上有些教程说，可以改`C:\ProgramData\MySQL`下面的`my.ini`文件。但这个版本在C盘下没有这个文件。那么需要改的就是安装目录下的配置文件。在`[mysqld]`下增加`secure_file_priv=''`语句。这个一个**大坑**是：安装目录下的配置文件名必须为`my.ini`，如果为`my-default.ini`则无效。之后重启MySQL服务即可。

通过`LOAD DATA INFILE`语句将数据文件添加到之前建立的表。

![1568121500273](C:\Users\agood\AppData\Roaming\Typora\typora-user-images\1568121500273.png)

导入成功。

### 分析

## Update Log

- 2019-09-09 22:18:12，首次更新
- 2019-09-10 21:31:31，更新数据导入部分