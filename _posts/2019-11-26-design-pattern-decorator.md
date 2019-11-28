---
layout: post
title: '(5) 23种设计模式之装饰模式'
subtitle: '装饰模式简介及实现方式'
date: 2019-11-26
categories: 技术
tags: java 设计模式
---

## 1. 基本概念
### 1.1 定义
装饰器模式（Decorator Pattern）又叫做包装模式，允许向一个现有的对象添加新的功能，同时又不改变其结构，是继承关系的一种替换方案

### 1.2 主要作用
动态地给一个对象添加一些额外的职责。就增加功能来说，装饰器模式相比生成子类更为灵活。一般，扩展一个类都是使用继承的方式，但是随着功能的增多，子类会异常庞大，当有新功能时可能需要修改子类。
### 1.3 应用场景
在一类对象有多种不同的特性，但又不想通过继承多个类实现时可以用装饰者模式。
比如奶茶，一般有原味奶茶，布丁奶茶，珍珠奶茶，红豆奶茶，甚至双拼奶茶。
如果使用继承机制，那么先要创建父类，原味奶茶，如果要布丁奶茶，那么需要继承原味奶茶再增加布丁。珍珠奶茶同样，创建新的子类添加珍珠。如果双拼的话就是添加两种辅料。
如果使用装饰者模式，先创建奶茶接口，原味奶茶实现奶茶接口就是被装饰类，创建奶茶装饰类接口。创建布丁奶茶装饰类实现奶茶接口，并增加加入布丁的方法。
### 1.4 相关角色
1. 基础接口：一类对象的抽象
2. 被装饰类：实现基础接口，完成基础功能
3. 抽象装饰类：定义装饰类的变量和方法
4. 装饰类：继承抽象装饰类，添加具体实现。
## 2.具体代码实现
1.创建一个实体类实现Cloneable接口
 奶茶接口
```java
public interface MilkTea {
    String getName();
}
```
 原味奶茶实现奶茶接口
```java
public class OriginalMilkTea implements MilkTea{
    public String getName(){
        return "原味奶茶";
    }

}
```
 奶茶抽象装饰类
```java
public abstract class MilkTeaDecorator implements MilkTea{
    private MilkTea milkTea;
    public MilkTeaDecorator(MilkTea milkTea) {
        this.milkTea = milkTea;
    }
    public String getName(){
        String name = this.milkTea.getName();
        return name;
    }

}
```
布丁奶茶装饰类
```java
public class PuddingMilkTeaDecorator implements MilkTeaDecorator{
    private MilkTea milkTea;
    public MilkTeaDecorator(MilkTea milkTea) {
        this.milkTea = milkTea;
    }
    public String getName(){
        String name = "布丁"+this.milkTea.getName();
        return name;
    }

}
```
珍珠奶茶装饰类
```java
public class PearlMilkTeaDecorator implements MilkTeaDecorator{
    private MilkTea milkTea;
    public MilkTeaDecorator(MilkTea milkTea) {
        this.milkTea = milkTea;
    }
    public String getName(){
        String name = "珍珠"+this.milkTea.getName();
        return name;
    }

}
```
具体使用
```java
public class UserDemo{
    public static void main(String[] args) {
        //原味奶茶
        OriginalMilkTea originalMilkTea = new OriginalMilkTea();
        // 珍珠奶茶
        PearlMilkTeaDecorator pearlMilkTeaDecorator = new PearlMilkTeaDecorator(originalMilkTea);
        System.out.println(pearlMilkTeaDecorator.getName());
        // 布丁奶茶
        PuddingMilkTeaDecorator puddingMilkTeaDecorator = new PuddingMilkTeaDecorator(originalMilkTea);
        System.out.println(pearlMilkTeaDecorator.getName());
        // 双拼奶茶 多层装饰
        PuddingMilkTeaDecorator ShuangPinMilkTeaDecorator = new PuddingMilkTeaDecorator(pearlMilkTeaDecorator);
        System.out.println(ShuangPinMilkTeaDecorator.getName());
    }
}
```
## 3.优缺点

优点： 装饰类和被装饰类可以独立发展，不会相互耦合，装饰模式是继承的一个替代模式，装饰模式可以动态扩展一个实现类的功能。
比继承更加灵活，符合开闭原则

缺点： 多层装饰比较复杂。