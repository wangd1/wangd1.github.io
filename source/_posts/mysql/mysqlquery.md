---
title: MySQL查询指定日期的数据
author: wangd1
avatar: /img/tux1.jpg
authorLink: wangd1.top
comments: true
tags:
  - MySQL
abbrlink: 63302
date: 2020-07-28 00:00:00
---

> 记录下查询MySQL本周，上周，本月，最近N天的日期

<!-- more -->

1. 最近7天：

   ```sql
   -- 查询七天前的日期 
   -- 返回2020-07-21
   select DATE_SUB( CURDATE(), INTERVAL 7 DAY )
   ```

   

2. 最近15天：

   ```sql
   -- 查询十五天前的日期 
   -- 返回2020-07-13
   select DATE_SUB( CURDATE(), INTERVAL 15 DAY )
   ```

   

3. 最近1个月：

   ```sql
   -- 查询一个月前的日期 
   -- 返回2020-06-28
   select DATE_SUB( CURDATE(), INTERVAL 1 MONTH )
   ```

   

4. 最近一年：

   ```sql
   -- 查询一年前的日期 
   -- 返回2019-07-28
   select DATE_SUB( CURDATE(), INTERVAL 1 YEAR )
   ```

   

5. 查询本周：

   ```sql
   -- 查询当前日期是第多少周，因为我们一周从周一开始，所以需要设置参数1  
   -- 返回202031，表示第31周
   select YEARWEEK(DATE_FORMAT(CURDATE(),'%Y-%m-%d'),1)
   ```

   

6. 查询上周：

   ```sql
   -- 查询上一周是第多少周，因为我们一周从周一开始，所以需要设置参数1  
   -- 获取七天前的日期，然后计算
   -- 返回202030，表示第30周
   select YEARWEEK(DATE_FORMAT(DATE_SUB(CURDATE(), INTERVAL 7 DAY),'%Y-%m-%d'),1)
   ```

   

7. 查询本月：

   ```sql
   -- 查询本月，就格式化
   select DATE_FORMAT(CURDATE(), '%Y%m')
   ```

8. 查询周一到周五：

   ```sql
   -- 减1是周一，以此类推
   -- 返回2020-07-27周一
   select SUBDATE(CURDATE(),DATE_FORMAT(CURDATE(),'%w')-1)
   -- 返回2020-07-27周二
   select SUBDATE(CURDATE(),DATE_FORMAT(CURDATE(),'%w')-2)
   ```

   

9. 查询指定日期：就直接比较