---
layout: post
title: '数据库相关问题'
subtitle: '开发中遇到的数据库问题'
date: 2018-08-17
categories: 技术
tags: database
---
## 说明

这里记录我使用数据库遇到的相关问题

## 1.日期转字符串 or 字符串转日期


> mysql
data_format ( sysdate(), '%Y-%m-%d %H:%I:%S')
str_to_date ('2018-08-20 13:23:23',  '%Y-%m-%d %H:%I:%S')

> oracle
to_char (sysdate, 'yyyy-mm-dd hh24:mi:ss' )
to_date ('2018-08-20 13:23:23', 'yyyy-mm-dd hh24:mi:ss')

## 2. 当前时间及时间计算

> mysql
当前时间 sysdate()  或者 now()
> oracle
当前时间 sysdate

## 3. 空值替换

> mysql
IFNULL(字段名，替换值)
> oracle
NVL (字段名，替换值)

## 4.时区查询
> mysql
show VARIABLES like '%time_zone%'
> oracle
select dbtimezone from dual
## 5.连接字符串
> mysql
concat('%','字符串','%')
> oracle
concat(concat('%', '字符串'), '%') 

