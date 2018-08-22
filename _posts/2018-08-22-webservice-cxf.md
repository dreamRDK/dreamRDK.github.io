---
layout: post
title: 'springboot集成webService发布服务'
subtitle: '使用cxf发布webService'
date: 2018-08-22
categories: 技术
tags: database
---
## 1. 添加依赖

cxf 开发了 springboot starter 的包，所以只需要添加如下依赖即可
maven 依赖
```java
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-spring-boot-starter-jaxws</artifactId>
    <version>3.1.11</version>
</dependency>
```
gradle 的话是这个
```java
compile group: 'org.apache.cxf', name: 'cxf-spring-boot-starter-jaxws', version: '3.1.11'
```
## 2. 创建 webService 接口及实现类

### webService 接口示例

```java
import javax.jws.WebMethod;
import javax.jws.WebService;


@WebService(name = "ImsWebService", // 暴露服务名称
targetNamespace = "http://service.pssc.sgcc.com"// 命名空间,一般是接口的包名倒序
)
public interface ImsWebService {
	
	@WebMethod
	public String getKPIValue(String param);
	@WebMethod
	public String getStatus(String param);
}
```
### webService 接口实现类示例

```java
@WebService(serviceName = "ImsWebService", // 与接口中指定的name一致
targetNamespace = "http://service.pssc.sgcc.com", // 与接口中的命名空间一致,一般是接口的包名倒
endpointInterface = "com.sgcc.pssc.webservice.ImsWebService"// 接口地址
)
@Component
public class ImsWebServiceImpl implements ImsWebService {
	
	public String getKPIValue(String param){
		// 实现具体逻辑即可
	}

	
	public String getStatus(String param) {
		// 实现具体逻辑即可
	}
}
```
## 3. cxf 配置类

**warning**：这里有一个小坑，被我踩了。之前是 public ServletRegistrationBean disPachterServlet() ，使用这个之后除了 webService 的注册路径可以访问，其余的web访问路径，controller里的路径都不能访问，主要是这个方法覆盖了spring的默认映射导致的。改个名字就好了

```java
import javax.xml.ws.Endpoint;

import org.apache.cxf.Bus;
import org.apache.cxf.bus.spring.SpringBus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CxfConfig {
	@Autowired
	private ImsWebService imsWebService;
	
	@Bean
    public ServletRegistrationBean disServlet() { // 就是这个方法名不能写成dispachterServlet
        return new ServletRegistrationBean(new CXFServlet(),"/service/*");
    }
    @Bean(name = Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {
        return new SpringBus();
    }
	
	@Bean
    public Endpoint endpoint(){
        EndpointImpl endpoint = new EndpointImpl(springBus(),imsWebService);
        endpoint.publish("/ImsService");
        return endpoint;
    }
	
}

```
## 4. 调用webService服务

```java
@Test
	public void cxf(){
		JaxWsDynamicClientFactory dcf =JaxWsDynamicClientFactory.newInstance();
	    Client client =dcf.createClient("http://localhost:1919/service/ImsService?wsdl");
	    Object[] objects;
		try {
			objects = client.invoke("getKPIValue","方法参数");
			System.out.println(objects[0].toString());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```