---
title: SpringBoot 整合安全框架Spring Security 快速如门
date: 2020-02-13 17:06:36
tags:
	- SpringBoot 
	- 安全框架
	- SpringBoot整合
---

# 快速开始 Quick Start

## 第一步：引入Jar 包 POM文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.2.RELEASE</version>
        <!--<version>2.2.4.RELEASE</version>-->
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com</groupId>
    <artifactId>security</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>security</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <!--pom 依赖-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <!--<dependency>-->
        <!--<groupId>org.thymeleaf.extras</groupId>-->
        <!--<artifactId>thymeleaf-extras-springsecurity4</artifactId>-->
        <!--<version>3.0.2.RELEASE</version>-->
        <!--</dependency>-->
        <!--版本太低 导致无法加载sec 标签 换为高版本的即可-->
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--springBoot 热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
            <scope>true</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

### 出现的问题

导入的<dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity4</artifactId>
       	 </dependency> 

版本过低无法使用 安全标签在页面中无法起作用

 应该导入的版本是

​			<dependency>
​            <groupId>org.thymeleaf.extras</groupId>
​            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
​       	 </dependency> 

## 第二步是：开始进行编写配置类文件

```java
package com.security.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@EnableWebSecurity
@Configuration
/**
 *  必须继承  WebSecurityConfigurerAdapter
 *  必须加入 @EnableWebSecurity注解
 *  从写 configuration() 方法
 */
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    // 制定请求授权规则
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/").permitAll()
                //赋予每个请求的用户 级别以及身份
                .antMatchers("/level1/**").hasRole("vip1")
                .antMatchers("/level2/**").hasRole("vip2")
                .antMatchers("/level3/**").hasRole("vip3");
        //开启自动配置的登录功能
        // 如果没有权限就会回到登录页面
        //  http.formLogin().usernameParameter("user").passwordParameter("password").loginPage("/userLogin").loginProcessingUrl("/userLogin"); //是自定义登录页面以及登录的参数设置（密码。用户名）
        http.formLogin();
        //开启注销功能
        // 消除session 注销成功会返回这个页面:/login?logout
        http.logout().logoutSuccessUrl("/");
        //开启记住我功能
        //创建cookie
        //记住我的参数的名字   http.rememberMe().rememberMeParameter("remember-me");
        http.rememberMe();
        /**
         *  记录一下规则
         *  1. /login 请求来到登录页面
         *  2. 重定向到 /login?error 表示登录失败
         *  3. 更多详细规定
         */

    }

    //定义认证规则
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 使用内存查询用户
        auth.inMemoryAuthentication().passwordEncoder(new MyPasswordEncoder())
                .withUser("wanzicong").password("123456").roles("vip1", "vip2")
                .and()
                .withUser("mayun").password("123456").roles("vip2", "vip3")
                .and()
                .withUser("liuqiandong").password("123456").roles("vip3", "vip1");
        //使用jdbc查询用户
        //auth.jdbcAuthentication();
    }
}
```

### 出现的问题

进行路径访问时间导致参数 id 为空，登录的问题

### 解决办法

解决问题的网站： https://blog.csdn.net/weixin_39220472/article/details/80865411

编写MyPasswordEncoder类 ，对进行登录的密码 进行编码上的改造

并且继承 **PasswordEncoder 类**

代码如下：

```java 
package com.security.config;

import org.springframework.security.crypto.password.PasswordEncoder;

/**
 * 添加到认证当中去 防止登录时间报错
 * 这是SpringBoot 2的特性
 * 解决报错信息的网站: https://blog.csdn.net/weixin_39220472/article/details/80865411
 */
public class MyPasswordEncoder implements PasswordEncoder {

    @Override
    public String encode(CharSequence charSequence) {
        return charSequence.toString();
    }

    @Override
    public boolean matches(CharSequence charSequence, String s) {
        return s.equals(charSequence.toString());
    }
}

```



## 第三步 进行页面的Controller的编写 以及页面主页的编写

### 主页：

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<head>
    <meta charset="UTF-8">
    <title>Welcome Page</title>
</head>
<body>
<h1 align="center">欢迎光临武林秘籍管理系统</h1>
<div sec:authorize="!isAuthenticated()">
    <h2 align="center">游客您好，如果想查看武林秘籍<a th:href="@{/login}">请登录</a></h2>
</div>
<div sec:authorize="isAuthenticated()">
    <h2>
        <span sec:authentication="name"></span>您好，您的角色有：
        <span sec:authentication="principal.authorities"></span>
    </h2>
    <form th:action="@{/logout}" method="post">
        <input type="submit" value="注销">
    </form>
</div>
<hr>
<div sec:authorize="hasRole('vip1')">
    <h3>普通武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level1/1}">罗汉拳</a></li>
        <li><a th:href="@{/level1/2}">武当长拳</a></li>
        <li><a th:href="@{/level1/3}">全真剑法</a></li>
    </ul>
</div>
<div sec:authorize="hasRole('vip2')">
    <h3>高级武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level2/1}">太极拳</a></li>
        <li><a th:href="@{/level2/2}">七伤拳</a></li>
        <li><a th:href="@{/level2/3}">梯云纵</a></li>
    </ul>
</div>
<div sec:authorize="hasRole('vip3')">
    <h3>绝世武功秘籍</h3>
    <ul>
        <li><a th:href="@{/level3/1}">葵花宝典</a></li>
        <li><a th:href="@{/level3/2}">龟派气功</a></li>
        <li><a th:href="@{/level3/3}">独孤九剑</a></li>
    </ul>
</div>
</body>
</html>


```

### Controller 类

```java 
package com.security.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Controller
public class SecurityController {

    //定义页面跳转的前缀
    private final String PREFIX = "pages/";

    // 欢迎页面
    @GetMapping("/")
    public String index() {
        return "welcome";
    }

    // 登录页面
    @GetMapping("/userLogin")
    public String loginPage() {
        return PREFIX + "login";
    }

    // 返回级别为一的页面
    @GetMapping("/level1/{path}")
    public String level1(@PathVariable("path") String path) {
        return PREFIX + "level1/" + path;
    }

    // 返回级别为二的页面
    @GetMapping("/level2/{path}")
    public String level2(@PathVariable("path") String path) {
        return PREFIX + "level2/" + path;
    }

    // 返回级别为三的页面
    @GetMapping("/level3/{path}")
    public String level3(@PathVariable("path") String path) {
        return PREFIX + "level3/" + path;
    }
}
```



## 第四步：总结

安全框架分为两大步骤：

1. **进行安全的授权规则**

   对页面的地址进行访问权限的控制，进行分级别

2. **进行安全的** **认证规则**

   对用户的权限的设置，以及登录的设置

   每个用户会有不同的身份，不同的身份也会有不同的，访问权限

3. **他们这些设置全部体现在代码的配置类当中**

