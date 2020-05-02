---
title: springCloud 入门步骤和概念
date: 2020-02-07 13:52:07
tags:
	- springcloud
	- springboot
---

## 1. SpringBoot 入门

### 核心概念

注册中心

服务的消费者

服务的提供者

### SpringCloud 五大组件

• 服务发现——Netflix Eureka

• 客服端负载均衡——Netflix Ribbon

• 断路器——Netflix Hystrix

• 服务网关——Netflix Zuul

• 分布式配置——Spring Cloud Config

### SpringCloud的重要概念

配置管理

服务发现

熔断

路由

微代理

控制总线

一次性token

全局琐

leader选举

分布式session

集群状态

### 入门步骤

– 1、创建provider

– 2、创建consumer

– 3、引入Spring Cloud

– 4、引入Eureka注册中心

– 5、引入Ribbon进行客户端负载均衡

– 6、引入Feign进行声明式HTTP远程调用

## 2. SpringCloud 的入门实践

### 工程的基本组成

三个SpringBoot的模块

第一个是服务的注册中心

第二个是服务的提供者

第三个是服务的消费者

### 编写服务的注册中心

编写注册中心的配置

```yaml
server:
  port: 8761
eureka:
  instance:
    hostname: server  # 注册中心实例的名字
  client:
    register-with-eureka: false  # 作用就是不把自己注册在注册中心
    fetch-registry: false # 不获取服务的注册信息
    service-url:
      defaultZone:  http://localhost:8761/eureka/
```

编写启动注册功能的注解

```java
/**
 * 注册中心
 */
@SpringBootApplication
//启动注册中心的服务
@EnableEurekaServer
public class ServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServerApplication.class, args);
    }
}
```

### 编写服务提供者注册功能

正常编写业务逻辑 service and controller

编写注册提供者的配置信息 ，将服务注册到注册中心

编写注册的配置信息

```yaml
# 注册服务到注册中心

server:
  port: 8001


spring:
  application:
    name: provider

# 注册自己
eureka:
  instance:
    prefer-ip-address: true  #注册服务的时候使用服务的IP地址
  client:
    service-url:
      defaultZone:  http://localhost:8761/eureka/
```



### 编写服务消费者订阅功能



编写业务逻辑 service and controller 

编写注册信息将自己注册到注册中心  **注册信息 消费者 和 提供者的注册信息一样**

```yaml
server:
  port: 8002
spring:
  application:
    name: ticket-user

# 发现服务
eureka:
  instance:
    prefer-ip-address: true  #注册服务的时候使用服务的IP地址
  client:
    service-url:
      defaultZone:  http://localhost:8761/eureka/


```

编写启动类 添加注解 以及注册 用到的类信息

```java
@EnableDiscoveryClient
@SpringBootApplication
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    //发送http请求
    @LoadBalanced //平衡加载
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

实现业务逻辑的调用

```java
@RestController
public class UserController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/buy")
    public String buyTicket(String name) {
        
        //url 是 提供者在注册中心的主机名 + controller里面的请求地址
        String url =  "http://TICKET-PROVIDER/ticket";
        String ticket = restTemplate.getForObject(url, String.class);
        return name + "购买了" + ticket;
    }
}
```

### 入门实践的总结

他们进行引入pom 文件的时间引入的类名不一样 启用的作用不同 

一类是注册中心功能

一类是发现服务功能

代码路径 

https://github.com/garxin/SpringBoot-Advanced



