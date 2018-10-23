---
layout: post
title: 'EffectiveJava读书笔记-02'
subtitle: '对象通用方法'
date: 2018-09-12
categories: 技术
tags: EffectiveJava
---
## 1. 覆盖 equals 方法的通用约定
> 以下情形可以不覆盖 equals 方法

* 类的每个实例本质上是唯一的
* 使用者并不关心类是否“逻辑相等”，比如 Random 类
* 父类已经覆盖 equals 方法并且适用于子类
* 类是私有的或者包级私有的，equals 方法永远不会被调用
> 覆盖 equals 方法时需要遵循的约定

* 自反性：对于任何非 null 的引用值 x ，x.equals(x) 必须返回 true
* 对称性： 对于任何非 null 的引用值 x 和 y，当且仅当 x.equals(y) 返回 true 时，y.equals(x) 必须返回 true
* 传递性：对于任何非 null 的引用值 x , y 和 z ，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true ，那么 x.equals(z) 必须返回 true。
* 一致性：比较的对象 x ，y 信息没有修改，多次调用 equals 方法的返回值必须一致
> 覆盖 equals 方法的一些诀窍

* 使用 == 检查参数是否为对象的引用，如果是返回 true
* 使用 instanceof 检查参数是否为正确的类型，即判断是否属于这一类
* 将参数转换成正确的类型
* 判断对象的成员变量值是否相等
* 编写完之后，检查是否对称，一致，可传递
* 不要将 equals 方法参数 Object 改为其他类型
* 覆盖 equals 时必须覆盖 hashCode
## 2. 覆盖 equals 时总要覆盖 hashCode
两个对象根据 equals 方法比较是相等的，那么他们通过hashCode 返回的值也必须相等
可以使用自动生成的。
## 3. 始终要覆盖 toString
## 4. 谨慎覆盖 clone
## 5. 考虑实现 Comparable 接口

