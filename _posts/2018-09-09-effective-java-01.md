---
layout: post
title: '《Effective Java》读书笔记 01 '
subtitle: '创建和销毁对象'
date: 2018-09-09
categories: 技术
tags:EffectiveJava
---
## 1. 用静态工厂方法代替构造器

> 主要优点：

### 1.1 静态工厂方法不同于构造器,它有名称
静态工厂方法可以通过名称获取不同的对象示例，二构造器通过传参获取不同的实例不便于使用，容易搞混。

### 1.2 静态工厂方法不必每次调用的时候都创建新对象
静态工厂方法可以预先创建好实例，每次调用都返回相同的实例，从而实现重复利用,提升程序性能。

### 1.3 静态工厂方法可以返回任何子类对象
这种方式适用于接口框架，通过私有对象的构造方法，通过工厂提供对象，当对象有新的子类时对接口调用者来说，影响小。

例如java 的 JDBC 连接，它就是一个服务提供者框架，java 为各数据库厂商提供多个实现，各厂商实现的是同一个服务，即进行数据库连接操作，而客户端调用java 的API，只是注册的数据库连接驱动不同，实现解耦。

服务提供者框架包括四大部分：
1. 提供者注册 API （Provider Registration API）
2. 服务接口 （Service Interface）
3. 服务访问者 API （Service Access API）
4. 服务提供者接口 （Service Provider Interface）


一下是简单的实现
```java
// Service Interface 
public interface Service {
	// 定义服务的特有方法
}

// Service Provider Interface
public interface Provider {
	Service newService();
}

// Noninstantiable class for service registration and access
public class Services {
	private Services{} // 私有化构造方法
	
	// Maps service names to services
	private static finam Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
	public static final String DEFAULT_PROVIDER_NAME = "<def>";

	// Provider registration API
	public static void registerDefaultProvider(Provider p) {
		registerProvider(DEFAULT_PROVIDER_NAME, p);
	}
	public static void registerProvider(String name, Provider p) {
		providers.put(name,p);
	}
	
	// Service access API
	public static Service newInstance() {
		return newInstance(DEFAULT_PROVIDER_NAME);
	}
	public static Service newInstance(String name) {
		Provider p = providers.get(name);
		if(p==null) {
			throw new IllegalArgumentException("没有该名称的服务提供者："+name);
		}
		return p.newService();
	}
}
```
> 缺点：
### 1. 类被私有化不能被继承
### 2. 和其他静态方法没有区别，需要区分是否是静态工厂方法

## 2. 构造器有多个参数时考虑用构建器 Builder
这种方式更容易阅读和使用，当然缺点就是需要先创建构建器，对性能有影响。
> 示例
```java
// Builder 示例
public class NutritionFacts {
	private final int servingSize;
	private final int servings;
	private final int calories;
	private final int fat;
	private final int sodium;
	private final int carbohydrate;
	
	public static class Builder {
		// 必须参数
		private final int servingSize;
		private final int servings;
		// 可选参数 初始化为默认值
		private  int calories = 0;
		private  int fat = 0;
		private  int sodium = 0;
		private  int carbohydrate = 0;
		
		public Builder(int servingSize, int servings) {
			this.servingSize = servingSize;
			this.servings = servings;
		}
		
		public Builder calories(int val) {
			calories = val;
			return this;
		}
		public Builder fat (int val) {
			fat = val;
			return this;
		}
		public Builder sodium (int val) {
			sodium = val;
			return this;
		}
		public Builder carbohydrate (int val) {
			carbohydrate = val;
			return this;
		}

		public NutritionFacts build() {
			return new NutritionFacts(this);
		}
	}
	private NutritionFacts (Builder builder) {
		servingSize = builder.servingSize ;
		servings = bulider.servings ;
		calories = bulider.calories ;
		fat = bulider.fat ;
		sodium = bulider.sodium ;
		carbohydrate = bulider.carbohydrate ;
	}

// 客户端调用
NutritionFacts nf = new NutritionFacts.Builder(200,8).calories(100).fat(0).build();
```
## 3. 用私有构造器或者枚举强化 Singleton 属性
## 4. 私有构造器强化不可实例化能力
## 5.避免创建不必要的对象
尽量复用对象，而不是频繁创建
## 6. 消除过期的对象引用
及时清空对象引用
内存泄漏的三种常见类型
* 类自己管理内存
* 缓存
* 监听器和其他回调
## 7. 避免使用终结方法

