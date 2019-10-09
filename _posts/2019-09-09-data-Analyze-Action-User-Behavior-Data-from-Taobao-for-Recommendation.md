---
layout: post
title: 数据分析实战-User Behavior From Taobao-天池
date: 2019-09-09
categories: blog
tags: [数据分析实战]
---

## 数据分析实战——User Behavior Data from Taobao for Recommendation [天池]

### 一 分析指标和模型

#### 指标

#### 模型

##### AAARRR

##### RMF模型

### 二 数据分析

数据分析步骤：

- 提出问题
- 理解数据
- 数据清洗
- 模型构建
- 数据可视化

#### 0 数据导入 

解压后的数据有3.4G，这样基本就告别EXCEL了。所以可以加载到SQL里面观察一下。

先在SQL里面建立一个表用来放数据，字段名设置好。不要使用Navicat直接导入，太慢了。可以使用使用`LOAD DATA INFILE`。在新版的Sql里面，一般会提示secure_file_priv的权限问题。通过`show variables like '%secure%' `，可以看到secure_file_priv的状态，如果为null，则意味着不允许SQL写入和写出。

关于改权限的方式，网上有很多教程，可以参考。我说一下我碰到的坑。目前我使用的SQL是8.0.16解压版。网上有些教程说，可以改`C:\ProgramData\MySQL`下面的`my.ini`文件。但这个版本在C盘下没有这个文件。那么需要改的就是安装目录下的配置文件。在`[mysqld]`下增加`secure_file_priv=''`语句。这个一个**大坑**是：安装目录下的配置文件名必须为`my.ini`，如果为`my-default.ini`则无效。之后重启MySQL服务即可。

通过下面语句将数据文件添加到之前建立的表。

**教训：**为了提高LOAD DATA的速度，不要在设计表时**建立任何索引**。

```mysql
USE 数据库名称;
LOAD DATA INFILE '文件名称'
INTO TABLE 表名
FIELDS TERMINATED BY ','  -- csv文件
```

#### 1 提出问题



#### 2 理解数据

UserBehavior是阿里巴巴提供的一个淘宝用户行为数据集，用于隐式反馈推荐问题的研究。

下载地址：[戳这里](https://tianchi.aliyun.com/dataset/dataDetail?dataId=649&userId=1)

> 本数据集包含了2017年11月25日至2017年12月3日之间，有行为的约一百万随机用户的所有行为（行为包括点击、购买、加购、喜欢）。数据集的组织形式和MovieLens-20M类似，即数据集的每一行表示一条用户行为，由用户ID、商品ID、商品类目ID、行为类型和时间戳组成，并以逗号分隔。

每列详细情况：

| 列名称     | 说明                                               |
| ---------- | -------------------------------------------------- |
| 用户ID     | 整数类型，序列化后的用户ID                         |
| 商品ID     | 整数类型，序列化后的商品ID                         |
| 商品类目ID | 整数类型，序列化后的商品所属类目ID                 |
| 行为类型   | 字符串，枚举类型，包括('pv', 'buy', 'cart', 'fav') |
| 时间戳     | 行为发生的时间戳                                   |

注意到，用户行为类型共有四种，它们分别是

| 行为类型 | 说明                     |
| -------- | ------------------------ |
| pv       | 商品详情页pv，等价于点击 |
| buy      | 商品购买                 |
| cart     | 将商品加入购物车         |
| fav      | 收藏商品                 |

关于数据集大小的一些说明如下

| 维度         | 数量        |
| ------------ | ----------- |
| 用户数量     | 987,994     |
| 商品数量     | 4,162,024   |
| 商品类目数量 | 9,439       |
| 所有行为数量 | 100,150,807 |

在MySQl中新建表格—tb2017，具体如下：

| 字段名       | 类型 | 主键  |
| ------------ | ---- | ----- |
| UserID       | int  | True  |
| ItemID       | int  | True  |
| CategoryID   | int  | False |
| BehaviorType | var  | False |
| TimeStamps   | int  | True  |

同时，因为需要对BehaviorType字段进行搜索，所以为改字段建立一般索引。

#### 3 数据清洗

##### 数据选择

五个字段全部有效，选取全部字段。

##### 去重和子集

这个数据集共有100,150,807条记录，这个数据量就告别excel了。但在导入的过程中，我们发现了重复记录，所以只能使用pandas来处理了。对于过于大的数据，我们可以考虑分块读取，通过pd.read_csv(iterator=True)开启迭代读取，通过get_chunk()读取数据。

在最后的连接过程中，还是出现了MemoryError的错误。😭。我的内存是8G，只好含泪修改虚拟内存了（通过实验发现，最大虚拟内存开到10G就不会出现MemoryError）。

我们定义了UseID，ItemID和TimeStamps字段为主键，因此以这三列为基准在python中去重。

该数据集中，第五列TimeStamps为时间，采用的是Unixtime。通过发现，时间记录有错误，即有部分记录不在给定的时间范围内。通过自定义函数获取正确的时间范围并筛选。

由于数据集过于庞大，所以我们抽取了大约20, 000, 000条记录用于下一步分析。

Python代码如下。

```python
import pandas as pd
import time

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

# 对df列命名
cols = ['UserID', 'ItemID', 'CatogoryID', 'BehaviorType', 'TimeStamps']
df.columns = cols

# 针对主键去重
df2 = df.drop_duplicates(subset=['UserID', 'ItemID', 'TimeStamps'])


# 获取unixtime
def get_unixtime(timeStr):
    formatStr = "%Y-%m-%d %H:%M:%S"
    tmObject = time.strptime(timeStr, formatStr)
    tmStamp = time.mktime(tmObject)
    
    return int(tmStamp)

# 数据集描述的时间范围
startTime = get_unixtime("2017-11-25 00:00:00")
endTime = get_unixtime("2017-12-3 23:59:59")

# 筛选
df_out = df2.loc[(df2['TimeStamps'] >= startTime) & (df2['TimeStamps'] <= endTime)]

# 抽取子集
df_out = df_out.sample(frac=0.2, random_state=1)

# 去重前的数据维度
print(df.shape)  # (100150807, 5)
# 去重和抽取后的数据维度
print(df_out.shape)  # (20019035, 5)

# 输出为csv
df_out.to_csv('UserBehavior_20.csv', index=False, header=False)
```

##### 列名重命名

导出的数据集没有header，所以在MySQL创建表格时，定义'UserID', 'ItemID', 'CategoryID', 'BehaviorType', 'TimeStamp'五个字段。

##### 缺失值

在创建表格时，五个字段均定义为NOT NULL。

##### 一致化处理

添加datestimes字段，其内容是timestamps字段转换后的时间：

```mysql
ALTER TABLE tb2017 ADD COLUMN datestimes TIMESTAMP ( 0 );
UPDATE tb2017 
SET datestimes = FROM_UNIXTIME( timestamps );
```

将日期单独做一个字段dates：

```mysql
ALTER TABLE tb2017 ADD COLUMN dates char(10) NOT null;
UPDATE tb2017
set dates = SUBSTRING(datestimes FROM 1 FOR 10);
```

将具体时间单独做一个字段hours：

```mysql
ALTER TABLE tb2017 ADD COLUMN hours CHAR(10) NOT null;
UPDATE tb2017
set hours = SUBSTRING(datestimes FROM 12 FOR 8)
```

完整的表格如下：

| UseID | ItemID | CategoryID | BehaviorType | TimeStamps | datestimes | dates | hours |
| ----- | ------ | ---------- | ------------ | ---------- | ---------- | ----- | ----- |
|1|	266784|	2520771	|pv	|1511884553|	2017-11-28 23:55:53|	2017-11-28|	23:55:53|
|1|	266784	|2520771|	pv|1511909676|	2017-11-29 06:54:36|	2017-11-29|	06:54:36|
|1|	818610|	411153|	pv	|1512268502	|2017-12-03 10:35:02|	2017-12-03|	10:35:02|
|1|	929177	|4801426|	pv	|1512252443|	2017-12-03 06:07:23|	2017-12-03|	06:07:23|

##### 检查异常值

虽然已经在数据清洗部分排除在时间上有异常的记录，但再检查一遍更为保险。

```mysql
SELECT
	MAX( datestimes ),
	MIN( datestimes ) 
FROM
	tb2017
```

时间上正常，结果如下：

| MAX(datestimes)     | MIN(datestimes)     |
| ------------------- | ------------------- |
| 2017-12-03 23:59:59 | 2017-11-25 00:00:00 |

#### 	4 模型构建

##### 漏斗分析——分析用户在使用淘宝过程中的转化率

用户使用淘宝的常见过程是：浏览（PV），之后可以收藏（fav）、加入购物车（cart）或者购买（buy）。其中，fav、cart不是buy的必须过程。转化过程如下：

![u5t13Q.png](https://s2.ax1x.com/2019/10/09/u5t13Q.png)

> 上图每一个箭头都表示转化

查看四个过程的用户数，代码如下：

```mysql
SELECT DISTINCT
	BehaviorType,
	COUNT( DISTINCT UserID ) AS USER 
FROM
	tb2017 
GROUP BY
	BehaviorType 
ORDER BY
	BehaviorType DESC
```

结果如下：

|BehaviorType|	USER|
|-|-|
|pv|	972217|
|fav|	221218|
|cart|	464701|
|buy|	283121|

在972217名浏览用户中，最终有283121名用户付钱，占比29.12%。

**可插入漏斗**

接下来再计算平均访问转化率，即buy/pv，代码如下：

```mysql
SELECT
	BehaviorType,
	COUNT( UserID ) AS USER 
FROM
	tb2017 
WHERE
	BehaviorType = 'pv' 
	OR BehaviorType = 'buy' 
GROUP BY
	BehaviorType
```

结果如下：

|BehaviorType|	user|
|-|-|
|buy|	402343|
|pv|	17935715|

平均访问转化率为2.24%。

可以看出，在付费用户为29.12%的情况下，平均访问转化率只有2.24。针对该情况提出以下**假设**：非付费用户比付费用户贡献了更多的PV。



## Update Log

- 2019-09-09 22:18:12，首次更新
- 2019-09-10 21:31:31，更新数据导入部分
- 2019年9月15日17:57:02，更新数据清洗部分
- 2019-10-09 22:52:14，更新模型部分