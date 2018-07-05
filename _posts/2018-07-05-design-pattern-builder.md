---
layout: post
title: '设计模式之建造模式'
subtitle: '建造模式简介及实现方式'
date: 2018-07-05
categories: 技术
tags: java 设计模式
---

## 1. 基本概念
### 1.1 定义
将一个复杂问题的构建和它的表示分离,使得同样的构建过程可以创建不同的表示。

### 1.2 主要作用
在用户不知道对象的建造过程和细节的情况下就可以创建复杂对象。
  1. 用户只需要给出指定复杂对象的类型和内容；
  2. 建造者模式负责按顺序创建对象（隐藏内部建造过程和细节）

### 1.3 建造模式结构及角色
![alt](http://tale.rdk.fun:9000/upload/2018/04/4ps423nqdojo5rl2qkff169rm1.gif)
建造者模式包括的角色：
  1. Builder：给出一个抽象类或者接口，规范产品的建造。规定了要实现哪些不减的建造，但具体实现交给子类；
  2. ConcreteBuilder：实现Builder接口以创建和构造产品部件，返回对象实例；
  3. Director：调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建。
  4. 要创建的复杂对象

## 2.具体代码实现
以组装电脑为例
### 1. 定义组装过程，Builder类
```java
public abstract class Builder {
  // 装cpu
  public abstract void buildCPU(String cpu);
  // 装主板
  public abstract void buildMainboard(String mb);
  // 装内存
  public abstract void buildRam(String ram);
  // 获取组装好的电脑
  public abstract Computer getComputer();
}
```  

### 2. Director 指挥Builder 按一定顺序完成操作
```java
public class Director {
  private Builder builder;
  public Director(Builder b){
    this.builder = b;
  }
  public  Computer createComputer(String cpu,String mb,String ram) {
    builder.buildCPU(cpu);
    builder.buildRam(ram);
    builder.buildMainboard(mb);
    return builder.getComputer;
    }
}
```  

### 3. Product产品类Computer
```java
public class Computer {
    private String mCpu;
    private String mMainboard;
    private String mRam;

    public void setmCpu(String mCpu) {
        this.mCpu = mCpu;
    }

    public void setmMainboard(String mMainboard) {
        this.mMainboard = mMainboard;
    }

    public void setmRam(String mRam) {
        this.mRam = mRam;
    }
}
```  

### 4.Builder具体实现类ConcreteBuilder
```java
public class MyComputerBuilder extends Builder {
    private Computer mComputer = new Computer();
    @Override
    public void buildCpu(String cpu) {
        mComputer.setmCpu(cpu);
    }

    @Override
    public void buildMainboard(String mainboard) {
        mComputer.setmMainboard(mainboard);
    }

    @Override
    public void buildRam(String ram) {
        mComputer.setmRam(ram);
    }

    @Override
    public Computer create() {
        return mComputer;
    }
}
```  

### 5. 实际调用
```java
Builder mBuilder = new MyComputerBuilder();
Direcror mDirecror=new Direcror(mBuilder);
Computor c = mDirecror.CreateComputer("i7","Intel主板","mRam");
```  

## 3.优缺点
**优点:**
- 易于解耦 
将产品本身与产品创建过程进行解耦，可以使用相同的创建过程来得到不同的产品。也就说细节依赖抽象。
- 易于精确控制对象的创建 
将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰
- 易于拓展 
增加新的具体建造者无需修改原有类库的代码，易于拓展，符合“开闭原则“。
**缺点:**
- 建造者模式所创建的产品一般具有较多的共同点，其组成部分相似；如果产品之间的差异性很大，则不适合使用建造者模式，因此其使用范围受到一定的限制。
- 如果产品的内部变化复杂，可能会导致需要定义很多具体建造者类来实现这种变化，导致系统变得很庞大