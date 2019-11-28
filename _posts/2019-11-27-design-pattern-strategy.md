---
layout: post
title: '(6) 23种设计模式之策略模式'
subtitle: '策略模式简介及实现方式'
date: 2019-11-26
categories: 技术
tags: java 设计模式
---

## 1. 基本概念
### 1.1 定义
在策略模式（Strategy Pattern）中，一个类的行为或其算法可以在运行时更改。这种类型的设计模式属于行为型模式。
在策略模式中，我们创建表示各种策略的对象和一个行为随着策略对象改变而改变的 context 对象。策略对象改变 context 对象的执行算法。
简单说就是根据对象不同选择不同的算法。

### 1.2 应用场景
 1、如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。 
 2、一个系统需要动态地在几种算法中选择一种。
 3、如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。
### 1.3 结构及相关角色
1. 策略接口：定义抽象的策略方法
2. 具体策略实现类：实现策略接口，一般有多个具体策略
3. 策略上下文：执行具体的策略
## 2.具体代码实现
以实现加减法为例

策略接口
```java
public interface CalculateStrategy {
    int doOpertion(int num1,int num2);
}
```
具体策略实现类
加法策略
```java
public class AddStrategy implements CalculateStrategy{
    @Override
    public int doOpertion(int num1,int num2){
        return num1 + num2;
    }
}
```
减法策略
```java
public class SubstractStrategy implements CalculateStrategy{
    @Override
    public int doOpertion(int num1,int num2){
        return num1 - num2;
    }
}
```
策略上下文
```java
public class StrategyContext {
   private CalculateStrategy strategy;
     
   public StrategyContext(CalculateStrategy strategy){
      this.strategy = strategy;
   }
 
   public int executeStrategy(int num1, int num2){
      return strategy.doOperation(num1, num2);
   }
}
```

具体使用
```java
public class UserDemo {
    public void main(String[] args) {
        // 执行加法
        StrategyContext strategyContext = new StrategyContext(new AddStrategy());
        int result1 = strategyContext.executeStrategy(10, 5);
        // 执行减法
        StrategyContext strategyContext2 = new StrategyContext(new SubstractStrategy());
        int result2 = strategyContext2.executeStrategy(10, 5);

    }   
}
```

## 3.优缺点

优点： 1、算法可以自由切换。 2、避免使用多重条件判断。 3、扩展性良好。

缺点： 1、策略类会增多。 2、所有策略类都需要对外暴露。