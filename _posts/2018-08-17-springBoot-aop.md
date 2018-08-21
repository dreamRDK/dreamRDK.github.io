---
layout: post
title: 'springboot使用aop实现日志记录'
subtitle: '通过aop记录请求信息发送kafka'
date: 2018-08-17
categories: 技术
tags: java springboot
---
## AOP拦截器创建
这里使用的是注解模式，因为业务需求只需要拦截部分请求，也可以使用扫描包的方式

### InterfaceStatistics 

    统计接口调用次数的注解
```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface InterfaceStatistics {

}
```
### LogInterceptor 
``` java 
@Component
@Aspect
public class LogInterceptor {
	
	@Pointcut("@annotation(com.sgcc.pssc.interfacestatistics.InterfaceStatistics)")
	// @Pointcut("execution(* com.jiaobuchong.web.*Controller.*(..))") 拦截指定包下的controller
	public void controllerAspect() {
		
	}
	
	@Around("controllerAspect()") // 环绕通知
	public Object aroundMethod(ProceedingJoinPoint pjd){
		Object result = null;
//		String methodName = pjd.getSignature().getName();
		Object[] args = pjd.getArgs();
		RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        HttpServletRequest request = sra.getRequest();

        String uri = request.getRequestURI();
        String ip = getIPAddress(request);
        Map<String,Object> dataMap = new HashMap<>();
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        dataMap.put("requestUri", uri);
		dataMap.put("requestIp", ip);
		dataMap.put("requestParams", args[0]);
		dataMap.put("requestTime",format.format(new Date()));
		dataMap.put("isSuccess","1");
		try {
			//前置通知
			result = pjd.proceed();
			//后置通知
		} catch (Throwable e) {
			e.printStackTrace();
			//异常通知
			dataMap.put("exception",e.getMessage());
			dataMap.put("isSuccess","0");
			throw new RuntimeException(e);
			
		}
		//返回通知
		try {
			dataMap.put("returnResult",result);
			String jsonData = JSONObject.fromObject(dataMap).toString();
			System.out.println("接口统计数据："+ jsonData);
			publishToKafka(jsonData, "interfaceStatistics");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			return result;
		}
		
	}
	
	// 获取信息后发送kafka消息，推送给消费者保存到数据库
	public boolean publishToKafka(String jsonData, String topic) {
		RestTemplate restClient = new RestTemplate();
		// 请求地址，这里调用了另一个微服务发送kafka
		String url = config.getKafkaUrl();
		Map<String, String> map = new HashMap();
		map.put("topic", topic);
		map.put("key", "test");
		map.put("data", jsonData);
		
		Map result = new HashMap();
		try {
			result = restClient.postForObject(url, map, Map.class);
		} catch (RestClientException e) {
			e.printStackTrace();
			logger.error("kafka主题：" + topic + "接口统计信息发送失败");
			return false;
		}
		
		System.out.println("接口统计数据推送到kafka主题：" + topic + "状态=" + result.get("success"));
		logger.info("接口统计数据发布到kafka主题：" + topic + "状态=" +"成功" );
		return true;
	}
	// 获取用户ip方法
	public  String getIPAddress(HttpServletRequest request) {
	    String ip = null;

	    //X-Forwarded-For：Squid 服务代理
	    String ipAddresses = request.getHeader("X-Forwarded-For");
	    if (ipAddresses == null || ipAddresses.length() == 0 || "unknown".equalsIgnoreCase(ipAddresses)) {
	        //Proxy-Client-IP：apache 服务代理
	        ipAddresses = request.getHeader("Proxy-Client-IP");
	    }
		if (ipAddresses == null || ipAddresses.length() == 0 || "unknown".equalsIgnoreCase(ipAddresses)) {
	        //WL-Proxy-Client-IP：weblogic 服务代理
	        ipAddresses = request.getHeader("WL-Proxy-Client-IP");
	    }
		if (ipAddresses == null || ipAddresses.length() == 0 || "unknown".equalsIgnoreCase(ipAddresses)) {
	        //HTTP_CLIENT_IP：有些代理服务器
	        ipAddresses = request.getHeader("HTTP_CLIENT_IP");
	    }
		if (ipAddresses == null || ipAddresses.length() == 0 || "unknown".equalsIgnoreCase(ipAddresses)) {
	        //X-Real-IP：nginx服务代理
	        ipAddresses = request.getHeader("X-Real-IP");
	    }

	    //有些网络通过多层代理，那么获取到的ip就会有多个，一般都是通过逗号（,）分割开来，并且第一个ip为客户端的真实IP
	    if (ipAddresses != null && ipAddresses.length() != 0) {
	        ip = ipAddresses.split(",")[0];
	    }

	    //还是不能获取到，最后再通过request.getRemoteAddr();获取
	    if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ipAddresses)) {
	        ip = request.getRemoteAddr();
	    }
	    return ip;
	}
	
}
```