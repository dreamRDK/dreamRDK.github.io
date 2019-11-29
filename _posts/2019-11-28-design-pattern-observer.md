---
layout: post
title: '(7) 23种设计模式之观察者模式'
subtitle: '观察者模式简介及实现方式'
date: 2019-11-28
categories: 技术
tags: java 设计模式
---

## 1. 基本概念
### 1.1 定义
在观察者模式（Observer Pattern）中，当一个对象依赖多个对象,当这个对象状态发生变化时，依赖的其他对象都会跟随改变。这种类型的设计模式属于行为型模式。

### 1.2 应用场景
 一个对象（目标对象）的状态发生改变，所有的依赖对象（观察者对象）都将得到通知，进行广播通知
 
### 1.3 结构及相关角色
1. subject被观察对象：就算是被观察对象，它会引用所有观察者
2. observer抽象类：它定义了一个更新接口，使得在被观察者状态发生改变时通知自己
3. observer实现类：继承observer抽象类实现方法。

## 2. 具体代码实现
以粉丝追星为例

被观察者也就是明星
```java
public class IdolSubject {
    // 状态
    private String state;
    // 观察者集合
    private List<FanObserver> fans;
    public String getState(){
        return state;
    }   
    public void setState(String newState) {
        this.state = newState;
        notifyEveryOne(newState);
    }
    //注册观察者对象,粉丝关注明星
    public void attach(FanObserver fan) {
        fans.add(fan);
    }
    //注销观察者对象,粉丝脱粉
    public void detach(FanObserver fan) {
        fans.remove(fan);
    }
    //通知所有注册的观察者对象,明星发通知
    public void notifyEveryOne(String newState) {
        for (FanObserver fan : fans) {
            fan.update(newState);
        }
    }
}
```
观察者抽象类
```java
public abstract class FanObserver {
    protected String fanName;
    public abstract void update(String newState);
}
```
具体观察者 粉丝
```java
public class Fan implements FanObserver{
    @Override
    public Fan(String name){
        this.fanName = name;
    }
    @Override
    public void update(String newState) {
        System.out.println(fanName+":接收到爱豆消息："+newState);
    }
}
```

具体使用
```java
public class UserDemo {
    public void main(String[] args) {
        // 创造一个明星
        IdolSubject idol = new IdolSubject();
        //粉丝
        Fan fan1 = new Fan("坤坤最棒");
        Fan fan2 = new Fan("坤坤加油");
        Fan fan3 = new Fan("坤坤死忠");
        // 粉丝关注明星
        idol.attach(fan1);
        idol.attach(fan2);
        idol.attach(fan3);
        // 明星发布消息
        idol.notifyEveryOne("hello,我叫坤坤,会唱、跳、rap、篮球，music~~");
    }   
}
```

## 3. 优缺点

优点： 1、观察者和被观察者是抽象耦合的。 2、建立一套触发机制。

缺点： 1、如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。 
      2、如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。 
      3、观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。