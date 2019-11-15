---
layout: post
title: '(3)26种设计模式之原型模式'
subtitle: '原型模式简介及实现方式'
date: 2019-11-05
categories: 技术
tags: java 设计模式
---

## 1. 基本概念
### 1.1 定义
原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

### 1.2 主要作用
这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。

### 1.3 应用场景
1、资源优化场景。 
2、类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。 
3、性能和安全要求的场景。 
4、通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式。
5、一个对象多个修改者的场景。 
6、一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。 
7、在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone 的方法创建一个对象，然后由工厂方法提供给调用者。原型模式已经与 Java 融为浑然一体，大家可以随手拿来使用。

## 2.具体代码实现
1.创建一个实体类实现Cloneable接口
```java
public class Shape implements Cloneable {
   
   private String id;
   protected String type;
   
   public String getType(){
      return type;
   }
   
   public String getId() {
      return id;
   }
   
   public void setId(String id) {
      this.id = id;
   }
   
   public Object clone() {
      Object clone = null;
      try {
         clone = super.clone();
      } catch (CloneNotSupportedException e) {
         e.printStackTrace();
      }
      return clone;
   }
}
```
2.在需要使用时调用clone() 方法获取对象的拷贝
3.对象拷贝分为浅拷贝和深拷贝，浅拷贝只会拷贝对象中属性的引用，深拷贝需要自己重新实现将引用改为真正的值。

## 3.优缺点

优点： 1、性能提高。 2、逃避构造函数的约束。

缺点： 1、配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。 2、必须实现 Cloneable 接口。