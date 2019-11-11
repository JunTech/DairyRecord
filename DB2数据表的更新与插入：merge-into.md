---
title: DB2数据表的更新与插入：merge into
author: Juntech
top: false
cover: false
toc: true
mathjax: false
copyright: true
summary: DB2数据表的更新与插入：merge into
categories: MYSQL
tags: DB2
keywords: merge into
abbrlink: f8b2a8b3
date: 2019-11-09 13:35:00
---

## DB2数据库Merge的用法

​	我们都知道，数据库原有一张表，但是里面不能动，要给他加入状态，就只有新建一张表，但是问题又来了，今天把数据导入了，明天难道又要把数据重新导入一遍，就不能实现没有这些数据时就插入数据，有这些数据，如果有改动就更新数据吗？当然有，就是我们的merge方法。

​	下面我们来具体看看merge的用法：

​	简单的说就是，判断表中有没有符合on（）条件中的数据，有了就更新数据，没有就插入数据。　　

有一个表T，有两个字段a、b，我们想在表T中做Insert/Update，如果条件满足，则更新T中b的值，否则在T中插入一条记录。在IBM的SQL语法中，很简单的一句判断就可以了，语法如下：　　

```sql lite
merge into 目标表 a
 
using 源表 b
 
on(a.条件字段1=b.条件字段1 and a.条件字段2=b.条件字段2 ……)  
 
when matched then update set a.更新字段=b.字段
 
when  not matched then insert into a(字段1,字段2……)values(值1,值2……)
```

