---
layout: post
title: 'springboot使用自定义filter过滤特殊字符'
subtitle: '使用filter过滤特殊字符，防止xss攻击'
date: 2018-09-06
categories: 技术
tags: springboot filter
---
## 1.重写 HttpServletRequestWrapper
```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class XssHttpServletRequestWrapper extends HttpServletRequestWrapper {

	public XssHttpServletRequestWrapper(HttpServletRequest request) {
		super(request);
	}
	 @Override
    public String getParameter(String name) {
        // 返回值之前 先进行过滤
        return XssShieldUtil.stripXss(super.getParameter(XssShieldUtil.stripXss(name)));
    }

    @Override
    public String[] getParameterValues(String name) {
        // 返回值之前 先进行过滤
        String[] values = super.getParameterValues(XssShieldUtil.stripXss(name));
        if(values != null){
            for (int i = 0; i < values.length; i++) {
                values[i] = XssShieldUtil.stripXss(values[i]);
            }
        }
        return values;
    }
}

```

## 2. 创建字符过滤工具
```java

import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.apache.commons.lang3.StringUtils;

public class XssShieldUtil {
	
	private static List<Pattern> patterns = null;

    private static List<Object[]> getXssPatternList() {
        List<Object[]> ret = new ArrayList<Object[]>();

        ret.add(new Object[]{"<(no)?script[^>]*>.*?</(no)?script>", Pattern.CASE_INSENSITIVE});
        ret.add(new Object[]{"eval\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL});
        ret.add(new Object[]{"expression\\((.*?)\\)", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL});
        ret.add(new Object[]{"(javascript:|vbscript:|view-source:)*", Pattern.CASE_INSENSITIVE});
        ret.add(new Object[]{"<(\"[^\"]*\"|\'[^\']*\'|[^\'\">])*>", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL});
        ret.add(new Object[]{"(window\\.location|window\\.|\\.location|document\\.cookie|document\\.|alert\\(.*?\\)|window\\.open\\()*", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL});
        ret.add(new Object[]{"<+\\s*\\w*\\s*(oncontrolselect|oncopy|oncut|ondataavailable|ondatasetchanged|ondatasetcomplete|ondblclick|ondeactivate|ondrag|ondragend|ondragenter|ondragleave|ondragover|ondragstart|ondrop|onerror=|onerroupdate|onfilterchange|onfinish|onfocus|onfocusin|onfocusout|onhelp|onkeydown|onkeypress|onkeyup|onlayoutcomplete|onload|onlosecapture|onmousedown|onmouseenter|onmouseleave|onmousemove|onmousout|onmouseover|onmouseup|onmousewheel|onmove|onmoveend|onmovestart|onabort|onactivate|onafterprint|onafterupdate|onbefore|onbeforeactivate|onbeforecopy|onbeforecut|onbeforedeactivate|onbeforeeditocus|onbeforepaste|onbeforeprint|onbeforeunload|onbeforeupdate|onblur|onbounce|oncellchange|onchange|onclick|oncontextmenu|onpaste|onpropertychange|onreadystatechange|onreset|onresize|onresizend|onresizestart|onrowenter|onrowexit|onrowsdelete|onrowsinserted|onscroll|onselect|onselectionchange|onselectstart|onstart|onstop|onsubmit|onunload)+\\s*=+", Pattern.CASE_INSENSITIVE | Pattern.MULTILINE | Pattern.DOTALL});
        return ret;
    }

    private static List<Pattern> getPatterns() {

        if (patterns == null) {

            List<Pattern> list = new ArrayList<Pattern>();

            String regex = null;
            Integer flag = null;
            int arrLength = 0;

            for(Object[] arr : getXssPatternList()) {
                arrLength = arr.length;
                for(int i = 0; i < arrLength; i++) {
                    regex = (String)arr[0];
                    flag = (Integer)arr[1];
                    list.add(Pattern.compile(regex, flag));
                }
            }

            patterns = list;
        }

        return patterns;
    }

    public static String stripXss(String value) {
        if(StringUtils.isNotBlank(value)) {

            Matcher matcher = null;

            for(Pattern pattern : getPatterns()) {
                matcher = pattern.matcher(value);
                // 匹配
                if(matcher.find()) {
                    // 删除相关字符串
                    value = matcher.replaceAll("");
                }
            }

            value = value.replaceAll("<", "&lt;").replaceAll(">", "&gt;");
        }

//      if (LOG.isDebugEnabled())
//          LOG.debug("strip value: " + value);

        return value;
    }

}

```

## 3. 创建过滤器
```java
import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

import org.apache.commons.lang3.StringUtils;
//@WebFilter(filterName = "xssAndCsrfFilter", urlPatterns = "/*")
public class XssAndCsrfFilter implements Filter {
	
	private String webUrl;

	@Override
	public void destroy() {

	}

	@Override
	public void doFilter(ServletRequest req, ServletResponse response, FilterChain filterChain)
			throws IOException, ServletException {
		System.out.println("进入xssFilter");
		HttpServletRequest request = (HttpServletRequest) req;
		String referer = request.getHeader("referer");
		String origin = request.getHeader("origin");
		System.out.println("referer:"+referer);
		System.out.println("origin:"+origin);
		if(origin != null && StringUtils.isNotBlank(origin) && origin.equalsIgnoreCase(webUrl)) {
			// 过滤特殊字符
			XssHttpServletRequestWrapper xssRequest = new XssHttpServletRequestWrapper(request);
			filterChain.doFilter(xssRequest, response);
		} else {
			response.setContentType("text/html;charset=UTF-8");
			PrintWriter writer = response.getWriter();
			writer.write("非法访问");
			writer.flush();
			writer.close();
		}
		
	}

	@Override
	public void init(FilterConfig config) throws ServletException {
		webUrl = config.getInitParameter("webUrl");
	}

}

```
## 4. 创建过滤器配置文件
过滤器可以使用注解，也可使用配置文件配置的方式
使用注解在过滤器上加
@WebFilter(filterName = "xssAndCsrfFilter", urlPatterns = "/*")
并且在启动类上添加@ServletComponentScan
```java 
package com.sgcc.pssc.sm.security;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;


@Configuration
public class XssAndCsrfFilterConfig {
	@Value("${webUrl}")
	private String webUrl;
	
	@Bean
    public FilterRegistrationBean xssAndCsrfFilterRegister() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        //注入过滤器
        registration.setFilter(new XssAndCsrfFilter());
        //拦截规则
        registration.addUrlPatterns("/*");
        //过滤器名称
        registration.setName("xssAndCsrfFilter");
        //过滤器顺序
        registration.setOrder(1);
        // 添加初始化参数
        registration.addInitParameter("webUrl", webUrl);
        return registration;
    }
	
}

```

