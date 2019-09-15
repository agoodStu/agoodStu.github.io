---
layout: post
title: 数据分析实战-User Behavior From Taobao-天池
date: 2019-09-09
categories: blog
tags: [数据分析实战]
---

## 数据分析实战——User Behavior Data from Taobao for Recommendation [天池]

### 0 数据导入 

解压后的数据有3.4G，这样基本就告别EXCEL了。所以可以加载到SQL里面观察一下。

先在SQL里面建立一个表用来放数据，字段名设置好。不要使用Navicat直接导入，太慢了。可以使用使用`LOAD DATA INFILE`。在新版的Sql里面，一般会提示secure_file_priv的权限问题。通过`show variables like '%secure%' `，可以看到secure_file_priv的状态，如果为null，则意味着不允许SQL写入和写出。

关于改权限的方式，网上有很多教程，可以参考。我说一下我碰到的坑。目前我使用的SQL是8.0.16解压版。网上有些教程说，可以改`C:\ProgramData\MySQL`下面的`my.ini`文件。但这个版本在C盘下没有这个文件。那么需要改的就是安装目录下的配置文件。在`[mysqld]`下增加`secure_file_priv=''`语句。这个一个**大坑**是：安装目录下的配置文件名必须为`my.ini`，如果为`my-default.ini`则无效。之后重启MySQL服务即可。

通过下面语句将数据文件添加到之前建立的表。

```mysql
USE 数据库名称;
LOAD DATA INFILE '文件名称'
INTO TABLE 表名
FIELDS TERMINATED BY ','  -- csv文件
```

### 2 数据理解

#### 数据源

UserBehavior是阿里巴巴提供的一个淘宝用户行为数据集，用于隐式反馈推荐问题的研究。

下载地址：[戳这里](https://tianchi.aliyun.com/dataset/dataDetail?dataId=649&userId=1)

#### 数据维度

| 维度         | 数量        |
| ------------ | ----------- |
| 用户数量     | 987,994     |
| 商品数量     | 4,162,024   |
| 商品类目数量 | 9,439       |
| 所有行为数量 | 100,150,807 |

#### 

### 数据清洗

#### 数据选择

选取全部数据

#### header重命名

原始数据不包含header，所以我们通过sql新建立一个表格，共6个字段

#### 设计表

| 字段名       | 类型 | 键    |
| ------------ | ---- | ----- |
| UserID       | int  | True  |
| ItemID       | int  | True  |
| CategoryID   | int  | True  |
| BehaviorType | var  | False |
| TimeStamps   | int  | True  |

#### 去重

这个数据集共有100,150,807条记录，这个数据量就告别excel了。但在导入的过程中，我们发现了重复记录，所以只能使用pandas来处理了。对于过于大的数据，我们可以考虑分块读取，通过pd.read_csv(iterator=True)开启迭代读取，通过get_chunk()读取数据。

在最后的连接过程中，还是出现了MemoryError的错误。😭。我的内存是8G，只好含泪修改虚拟内存了（通过实验发现，最大虚拟内存开到10G就不会出现MemoryError）。Python代码如下。

```python
import pandas as pd

reader = pd.read_csv('UserBehavior.csv', iterator=True)

loop = True
chunk_size = 100000  #chunk_size
chunks = []

while loop:
    try:
        chunk = reader.get_chunk(chunk_size)
        chunks.append(chunk)
    except:
        loop = False
        print('Iteration is stopped!')

df = pd.concat(chunks, ignore_index=True)
df2 = df.drop_duplicates()
# 去重前的数据维度
print(df.shape)  # (100150807, 5)
# 去重后的数据维度
print(df2.shape)  # (100150758, 5)
# 49行的重复值，差点让我崩溃，我恨。
```

#### 时间戳

TimeStamp列用的是UnixTime。我们做个简单的查询：

```mysql
SELECT
	min( TimeStamp),
	MAX( TimeStamp ) 
FROM
	userbehavior2017
```

结果如下：

|min(TIMESTAMP)|	MAX(TIMESTAMP)|
|-|-|
|-2134949234|	2122867355|

UnixTime是以1970年为起始时间，难道我们穿越了？让我们看一下一共又多少个TimeStamp为负值：

```mysql
SELECT
	COUNT( TIMESTAMP ) 
FROM
	userbehavior2017 
WHERE
	TIMESTAMP < 0
```

结果共有38个值。当我们选择TimeStamp大于0的最早时间呢？

```mysql
SELECT
	FROM_UNIXTIME( min( TIMESTAMP ), '%Y-%M-%D' ) 
FROM
	userbehavior2017 
WHERE
	TIMESTAMP > 0
```

返回结果是`1970-January-1st`，也就是UnixTime开始的时间。

同样，我们看下最大的时间是多少？

```
SELECT
	FROM_UNIXTIME( max( TIMESTAMP ), '%Y-%M-%D' ) 
FROM
	userbehavior2017
```

返回结果是`2037-April-9th`，好吧，还没有到达2038年1月19日3时14分08秒。

当不管是最小时间还是最大时间，都不是我们想要的数据。根据数据源描述，数据集包含了2017年11月25日至2017年12月3日之间的数据。我们可以新建一个在这个时间范围内的视图。

## Update Log

- 2019-09-09 22:18:12，首次更新
- 2019-09-10 21:31:31，更新数据导入部分
- 2019年9月15日17:57:02，更新数据清洗部分