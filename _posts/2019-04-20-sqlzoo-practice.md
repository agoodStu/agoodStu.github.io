---
layout: post
title: sqlzoo练习题目
date: 2019-04-20
categories: blog
tags: [sql, sqlzoo]
---

#### 1 SELECT within SELECT

#### List each country **name** where the **population** is larger than that of 'Russia'.

```sql
SELECT name 
FROM
	world 
WHERE
	population > (
	SELECT
		population 
	FROM
		world 
WHERE
	name = 'Russia')
```

#### Show the countries in Europe with a per capita GDP greater than 'United Kingdom'.

```sql
SELECT name 
FROM
	world 
WHERE
	continent = 'Europe' 
	AND gdp / population > (
	SELECT
		gdp / population 
	FROM
		world 
WHERE
	name = 'United Kingdom')
```

#### List the name and continent of countries in the continents containing either Argentina or Australia. Order by name of the country.

```sql
SELECT name,
	continent 
FROM
	world 
WHERE
	continent IN (
	SELECT
		continent 
	FROM
		world 
	WHERE
	NAME IN ( 'Argentina', 'Australia' )) 
ORDER BY
NAME
```

#### Which country has a population that is more than Canada but less than Poland? Show the name and the population.

```sql
SELECT
	name,
	population 
FROM
	world 
WHERE
	population > ( SELECT population FROM world WHERE name = 'Canada' ) 
	AND population < (
	SELECT
		population 
	FROM
		world 
WHERE
	NAME = 'Poland')
```

#### Show the name and the population of each country in Europe. Show the population as a percentage of the population of Germany.

```sql
SELECT 
	name,
	CONCAT( ROUND( population / ( SELECT population FROM world WHERE name = 'Germany' )* 100, 0 ), '%' ) 
FROM
	world 
WHERE
	continent = 'Europe'
```

#### Which countries have a GDP greater than every country in Europe? [Give the name only.] (Some countries may have NULL gdp values)

```sql
SELECT
	name
FROM
	world 
WHERE
	gdp > ALL (
	SELECT
		gdp 
	FROM
		world 
	WHERE
		continent = 'Europe' 
	AND gdp > 0 
	)
```

#### Find the largest country (by area) in each continent, show the continent, the name and the area:

```sql
SELECT
	continent,
	name,
	area 
FROM
	world x 
WHERE
	x.area >= ALL (
	SELECT
		area 
	FROM
		world y 
	WHERE
	x.continent = y.continent 
	AND y.area > 0)
```

#### List each continent and the name of the country that comes first alphabetically.

```sql
SELECT
	continent,
NAME 
FROM
	world x 
WHERE
	x.NAME =(
	SELECT NAME 
	FROM
		world y 
	WHERE
		y.continent = x.continent 
	ORDER BY
NAME 
	LIMIT 1)
```

#### Find the continents where all countries have a population <= 25000000. Then find the names of the countries associated with these continents. Show name, continent and population.

```sql
SELECT
	name,
	continent,
	population 
FROM
	world x 
WHERE
	25000000 >= ALL (
	SELECT
		population 
	FROM
		world y 
	WHERE
	x.continent = y.continent 
	AND y.population > 0)
```

#### Some countries have populations more than three times that of any of their neighbours (in the same continent). Give the countries and continents.

```sql
SELECT
	name,
	continent 
FROM
	world x 
WHERE
	population / 3 >= ALL (
	SELECT
		population 
	FROM
		world y 
	WHERE
		x.continent = y.continent 
	AND x.NAME != y.NAME 
	AND y.population > 0)
```

### 2. SUM and COUNT

#### Show the total population of the world.

```mysql
SELECT
	SUM( population ) 
FROM
	world
```

#### List all the continents - just once each.

```sql
SELECT DISTINCT
	continent 
FROM
	world
```

### Give the total GDP of Africa

```sql
SELECT
	SUM( gdp ) 
FROM
	world 
WHERE
	continent = 'Africa'
```

#### How many countries have an area of at least 1000000

```sql
SELECT
	COUNT( NAME ) 
FROM
	world 
WHERE
	area >= 1000000
```

### What is the total population of ('Estonia', 'Latvia', 'Lithuania')

```sql
SELECT
	SUM( population ) 
FROM
	world 
WHERE
	NAME IN (
		'Estonia',
	'Latvia',
	'Lithuania')
```

#### For each continent show the continent and number of countries.

```sql
SELECT
	continent,
	COUNT( NAME ) 
FROM
	world 
GROUP BY
	continent
```

#### For each continent show the continent and number of countries with populations of at least 10 million.

```sql
SELECT
	continent,
	COUNT( NAME ) 
FROM
	world 
WHERE
	population > 10000000 
GROUP BY
	continent
```

#### List the continents that have a total population of at least 100 million.

```sql
SELECT
	continent 
FROM
	world 
GROUP BY
	continent 
HAVING
	SUM( population ) >= 100000000
```



#### 总结：

注意理解WHERE，GROUP BY和HAVING的不同。

### 3. The Nobel table can be used to practice more SUM and COUNT functions

#### Show the total number of prizes awarded.

```
SELECT
	COUNT( winner ) 
FROM
	nobel
```

#### List each subject - just once.

```sql
SELECT DISTINCT
	subject
FROM
	nobel
```

#### Show the total number of prizes awarded for Physics.

```sql
SELECT
	COUNT( winner ) 
FROM
	nobel 
WHERE
	SUBJECT = 'Physics'
```

#### For each subject show the subject and the number of prizes.

```sql
SELECT
	subject,
	COUNT( winner ) 
FROM
	nobel 
GROUP BY
SUBJECT
```

#### For each subject show the first year that the prize was awarded.

```sql
SELECT
	subject,
	MIN( yr ) 
FROM
	nobel 
GROUP BY
SUBJECT
```

#### For each subject show the number of prizes awarded in the year 2000.

```sql
SELECT
	subject,
	COUNT( yr ) 
FROM
	nobel 
WHERE
	yr = 2000 
GROUP BY
SUBJECT
```

#### Show the number of different winners for each subject.

```sql
SELECT
	subject,
	COUNT(
	DISTINCT ( winner )) 
FROM
	nobel 
GROUP BY
	subject
```

#### For each subject show how many years have had prizes awarded.

```sql
SELECT
	subject,
	COUNT(
	DISTINCT ( yr )) 
FROM
	nobel 
GROUP BY
	subject
```

#### Show the years in which three prizes were given for Physics.

```sql
SELECT
	yr 
FROM
	nobel 
WHERE
	subject = 'Physics' 
GROUP BY
	yr 
HAVING
	COUNT(*) = 3
```

#### Show winners who have won more than once.

```sql
SELECT
	winner 
FROM
	nobel 
GROUP BY
	winner 
HAVING
	COUNT( winner ) >= 2
```



## Update log

- 二〇一九年四月二十日 12:54:28，首发在CSDN的[博客](<https://blog.csdn.net/zjjoebloggs>)上，现挪在此处
- 2019-08-30 21:42:38，修改代码格式