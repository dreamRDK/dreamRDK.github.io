---
layout: post
title: '设计模式之单例模式'
subtitle: '单例模式简介及实现方式'
date: 2018-07-04
categories: 技术
tags: java 设计模式
---

## 1.单例模式的概念思路

单例模式是一种常见的软件设计模式，常常我们的应用中只需要一个对象实例，此时可以用单例模式，每次调用都是同一个实体。  
实现思路：

1.  构造方法私有化，防止被实例化。
2.  提供静态方法，保证每次返回的实例是同一个。
3.  多线程下的调用注意线程安全

## 2.单例模式的七种实现

### 1.饿汉式(静态常量)

优点：类加载时完成实例化,写法简单,避免线程同步问题。  
缺点：没有采用延迟加载，如果一直未使用会造成内存浪费。

```java
  public class Singleton {

      private final static Singleton INSTANCE = new Singleton();

      private Singleton() {}

      public static Singleton getInstance() {
          return INSTANCE;
      }
  }
```

   

### 2\. 饿汉式(静态代码块)

优缺点同上，只是将实例化的操作放在了静态代码块中
```java
    public class Singleton {

      private static Singleton instance;

      private Singleton(){}

      static {
        instance = new Singleton();
      }

      public Singleton getInstance() {
        return instance;
      }
```
### 3.懒汉模式(线程不安全)

优点:延迟加载,只能用于单线程  
缺点:多线程下,线程不安全可能出现多个实例
```java
    public class Singleton {

      private static Singleton instance;

      private Singleton(){}

      public static getInstance() {
        if(instance==null) {
          instance = new Singleton();
        }
        return instance;
    }
```
### 4.懒汉模式（线程安全，同步方法）

优点：增加sycronized关键字，线程同步  
缺点：效率太低
```java
    public class Singleton {

        private static Singleton singleton;

        private Singleton() {}

        public static synchronized Singleton getInstance() {
            if (singleton == null) {
                singleton = new Singleton();
            }
            return singleton;
        }
    }
```
### 5.双重检查（推荐使用）

优点：线程安全，延迟加载，效率高。执行两次if（singleton==null），在实例化时保证线程安全，第二次调用会直接返回实例，不执行同步代码块了，效率提升。
```java
    public class Singleton {

        private static volatile Singleton singleton;

        private Singleton() {}

        public static Singleton getInstance() {
            if (singleton == null) {
                synchronized (Singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }
    }
```
### 6\. 静态内部类（推荐）

优点：静态内部类方式在Singleton类被装载时并不会立即实例化，而是在需要实例化时，调用getInstance方法，才会装载SingletonInstance类，从而完成Singleton的实例化。避免了线程不安全，延迟加载，效率高。
```java
    public class Singleton {

        private Singleton() {}

        private static class SingletonInstance {
            private static final Singleton INSTANCE = new Singleton();
        }

        public static Singleton getInstance() {
            return SingletonInstance.INSTANCE;
        }
    }
```
### 7.枚举（推荐）
```java
    public enum Singleton {
        INSTANCE;
        public void whateverMethod() {

        }
    }
```