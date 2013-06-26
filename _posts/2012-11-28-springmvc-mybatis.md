---
layout : post
category: notes
title : 学习Maven搭框架ing
---

**下面开始SpringMVC + Mybatis 部分**

1.  WEB-INF下建立web.xml
	
		<?xml version="1.0" encoding="UTF-8"?>
		<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
			http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
		
			<display-name>Test</display-name>
			<welcome-file-list>
				<welcome-file>index.html</welcome-file>
			</welcome-file-list>
			<!-- Character Encoding Filter -->
			<filter>
				<filter-name>Set Character Encoding</filter-name>
				<filter-class>
					org.springframework.web.filter.CharacterEncodingFilter
				</filter-class>
				<init-param>
					<param-name>encoding</param-name>
					<param-value>utf8</param-value>
				</init-param>
				<init-param>
					<param-name>forceEncoding</param-name>
					<param-value>true</param-value>
				</init-param>
			</filter>
			<filter-mapping>
				<filter-name>Set Character Encoding</filter-name>
				<url-pattern>/*</url-pattern>
			</filter-mapping>
		
		
			<servlet>
				<servlet-name>springmvc</servlet-name>
				<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
				<init-param>
					<param-name>contextConfigLocation</param-name>
					<param-value>classpath:app*.xml</param-value>
				</init-param>
				<load-on-startup>2</load-on-startup>
			</servlet>
			
			<servlet-mapping>
				<servlet-name>springmvc</servlet-name>
				<url-pattern>/</url-pattern>
			</servlet-mapping>
		</web-app>
	
2. 建立一些必要的包

3. 编写Spring配置文件app*.xml数据库配置文件

4. 写测试Controller
	
**UPDATE: 框架整出来了... 刚上传 [SpringMVC+Mybatis](https://github.com/dylanvivi/springmybatis) 笔记有空再补...**

{% include references.md %}