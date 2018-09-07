---
layout: post
title: 'springboot配置tomcat禁用不安全的http方法'
subtitle: '使用filter过滤特殊字符，防止xss攻击'
date: 2018-09-07
categories: 技术
tags: springboot tomcat
---
## 1. HTTP 方法描述
不安全的HTTP方法一般包括：TRACE、PUT、DELETE、COPY 等。其中最常见的为TRACE方法可以回显服务器收到的请求，主要用于测试或诊断，恶意攻击者可以利用该方法进行跨站跟踪攻击（即XST攻击），从而进行网站钓鱼、盗取管理员cookie等

## 2. tomcat 通过配置web.xml禁用http方法
```java
	<security-constraint>
		<web-resource-collection>
		<web-resource-name>fortune</web-resource-name>
		<url-pattern>/*</url-pattern>
		<http-method>PUT</http-method>
		<http-method>DELETE</http-method>
		<http-method>HEAD</http-method>
		<http-method>OPTIONS</http-method>
		<http-method>TRACE</http-method>
		</web-resource-collection>
		<auth-constraint></auth-constraint>
	</security-constraint>
	<login-config>
		<auth-method>BASIC</auth-method>
	</login-config>
```

## 3. springboot 配置内置tomcat 禁用不安全方法

```java
package com.sgcc.pssc.sm.security;

import org.apache.catalina.Context;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TomcatConfig {
 
    @Bean
    public EmbeddedServletContainerFactory servletContainer() {
    	TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory() {// 1  
            protected void postProcessContext(Context context) {  
                SecurityConstraint securityConstraint = new SecurityConstraint();  
                securityConstraint.setUserConstraint("CONFIDENTIAL");  
                SecurityCollection collection = new SecurityCollection();  
                collection.addPattern("/*");  
                collection.addMethod("HEAD");  
                collection.addMethod("PUT");  
                collection.addMethod("DELETE");  
//                collection.addMethod("OPTIONS");  
                collection.addMethod("TRACE");  
                collection.addMethod("COPY");  
                collection.addMethod("SEARCH");  
                collection.addMethod("PROPFIND");  
                securityConstraint.addCollection(collection);  
                context.addConstraint(securityConstraint);  
            }  
        };
        return tomcat;
    }
 
}

```