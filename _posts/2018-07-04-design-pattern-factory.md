---
layout: post
title: '(1) 23种设计模式之工厂模式'
subtitle: '工厂模式简介及实现方式'
date: 2018-07-04
categories: 技术
tags: java 设计模式
---

# 1. 基本概念
为对象创建提供过渡接口，将创建对象的具体过程屏蔽，达到灵活性的目的。
# 2. 工厂模式分类
工厂模式主要分为简单工厂模式，工厂方法模式和抽象工厂模式
## 1. 简单工厂模式
简单工厂模式又称为静态工厂方法模式。
组成：
- 工厂类角色：核心，含有一定的商业逻辑和判断逻辑，负责创建所需的产品
- 抽象产品角色：工厂类所创建的类的父类，用来定义具体产品所具有的公共接口
- 具体产品角色：工厂所创建的具体实例
示例代码：
```java
// 抽象产品角色
public interface Sample{
	void get();
}
// 具体产品角色A
public class SampleA implements Sample {
	public void get() {
		system.out.println("I'm SampleA");
	}
}
// 具体产品角色B
public class SampleB implements Sample {
	public void get() {
		system.out.println("I'm SampleB");
	}
}
// 工厂类角色
public class Factory{
  public static Sample creator(int which) {
  if(which==1) {
    return new SampleA();
  } else if(which==2) {
    return new SampleB();
  }
}
```
简单工厂模式的优缺点
  在该模式中，工厂是核心，它包含必要的判断逻辑，根据外界给定的条件，去创建具体的产品实例，而使用者不需要考虑实例是如何创建的。
  由于工厂类集中了所有的创建逻辑，导致产品类增多时，工厂必须跟着修改，扩展性不够好。

## 2. 工厂方法模式
工厂方法模式是简单工厂模式的进一步抽象化又被称为多态工厂模式，工厂方法模式将实例化的具体操作交给了具体工厂角色
组成：
- 抽象工厂角色：具体工厂角色的父类，可以是抽象类或接口
- 具体工厂角色：含有具体的业务逻辑是对抽象工厂角色的具体实现，负责具体产品的创建
- 抽象产品角色：它是具体产品继承的父类或者是实现的接口。在java中一般有抽象类或者接口来实现。
- 具体产品角色：具体工厂角色所创建的对象就是此角色的实例。在java中由具体的类来实现。
示例代码：
```java
// 抽象产品角色
public interface Moveable {
  void run();  
}
// 具体产品角色
public class Plane implements Moveable {
    @Override
    public void run() {
        System.out.println("plane....");
    }
}
//具体产品角色
public class Broom implements Moveable {
    @Override
    public void run() {
        System.out.println("broom.....");
    }
}  
// 抽象工厂角色
public abstract class VehicleFactory {
  abstract Moveable create();
}
//具体工厂
public class PlaneFactory extends VehicleFactory{
    public Moveable create() {
        return new Plane();
    }
}
//具体工厂
public class BroomFactory extends VehicleFactory{
    public Moveable create() {
        return new Broom();
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        VehicleFactory factory = new BroomFactory();
        Moveable m = factory.create();
        m.run();
    }
}
```
工厂方法模式，符合开放-封闭原则，当需要新的产品时，只需要创建产品的具体实现类和工厂具体实现类即可。

## 3. 抽象工厂模式
示例代码:
```java
//抽象工厂类
public abstract class AbstractFactory {
    public abstract Vehicle createVehicle();
    public abstract Weapon createWeapon();
    public abstract Food createFood();
}
//具体工厂类，其中Food,Vehicle，Weapon是抽象类，
public class DefaultFactory extends AbstractFactory{
    @Override
    public Food createFood() {
        return new Apple();
    }
    @Override
    public Vehicle createVehicle() {
        return new Car();
    }
    @Override
    public Weapon createWeapon() {
        return new AK47();
    }
}
//测试类
public class Test {
    public static void main(String[] args) {
        AbstractFactory f = new DefaultFactory();
        Vehicle v = f.createVehicle();
        v.run();
        Weapon w = f.createWeapon();
        w.shoot();
        Food a = f.createFood();
        a.printName();
    }
}

```