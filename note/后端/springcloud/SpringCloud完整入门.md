## SpringCloud 的入门学习



### 0. 在使用SPringCloud时间提前准备的环境

1. 提前确定Springboot 的版本和SpringCloud的版本 ===SpingCloud的版本是<version>2.0.4.RELEASE</version> 		 SpringCloud的版本是  <version>Finchley.SR1</version>

2. 导入Springboot的jar包  和SpringCloud的jar包

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.cloud</groupId>
       <artifactId>cloud</artifactId>
       <version>1.0-SNAPSHOT</version>
       <modules>
           <module>Erauke服务注册中心</module>
           <module>Erauke客户端1-服务的被调用方</module>
           <module>Feign调用别的微服务</module>
           <module>配置网关</module>
       </modules>
       <packaging>pom</packaging>
       <name>cloud</name>
       <description>夫工程</description>
   
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.0.4.RELEASE</version>
           <relativePath/>
       </parent>
   
       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
           <java.version>1.8</java.version>
       </properties>
   
       <!--版本锁定  dependencyManagement -->
       <dependencyManagement>
           <dependencies>
               <!--SpringCloud 的jar包-->
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>Finchley.SR1</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <!--<dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
           </dependency>-->
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
               <scope>test</scope>
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

### 1. 注册中心 Eurake

1.   作用 ：将服务注册进来 来让别人发现

2. 使用的顺序

   -   导入jar包坐标

     ```xml
            <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-netflix-eureka-                             server</artifactId>
             </dependency>
     ```

   -  编写启动类

     ```java
     @SpringBootApplication
     /*开启注册的服务*/
     @EnableEurekaServer
     public class Application {
         public static void main(String[] args) {
             SpringApplication.run(Application.class,args);
         }
     }
     ```

   -  编写配置文件application.yml

     ```yml
     server:
       #服务端口
       port: 8761
     eureka:
       instance:
         #注册中心服务的主机，默认是localhost
         hostname: localhost
       client:
         #是否将当前微服务注册到Eureka服务中。自己是注册中心，因此无需注册。
         register-with-eureka: false
         #是否从Eureka中获取注册信息。自己是注册中心，因此无需获取。
         fetch-registry: false
         #Eureka客户端与与Eureka服务端进行交互的地址Map表
         service-url:
           #默认http://localhost:8761/eureka/
           defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
     ```

   -   可以打开浏览器 可以看见注册中心的页面 http://localhost:8761/eureka/

   - 创建之后就可以想被别人调用的服务注册进来了

3.  将自己注册到服务中心的步骤

   - 导入jar包坐标

     ```xml
       	 <!--被调用的微服务-->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-netflix-eureka-    						client</artifactId>
             </dependency>
     ```

   -   编写启动类

     ```java
     
     @SpringBootApplication
     
     @EnableEurekaClient
     public class AppicationClient01 {
         public static void main(String[] args) {
             SpringApplication.run(AppicationClient01.class, args);
         }
     }
     ```

   -  编写配置文件application.yml

     ```yml
     server:
       #服务端口
       port: 9001
     spring:
       application:
         #指定服务名
         name:  cloud-client01
     
     eureka:
       client:
         #Eureka服务的地址
         service-url:
           defaultZone: http://localhost:8761/eureka/
       instance:
         #用于表示在猜测主机名时，服务器的IP地址应该与操作系统报告的主机名相对应。(注册服务和客户端如果在一台机器上则无需配置)
         prefer-ip-address: true
     ```

   -  编写controller代码等待别人的调用

### 2. 发现注册的服务并调用  Feign 

1.  作用 : 发现被注册到eurake 的服务  并实现调用

2.   使用的步骤

   -   导入jar包坐标 

     ```xml
      		<!--被调用的微服务-->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-netflix-eureka-								client</artifactId>
             </dependency>
     
             <!--调用微服务模块的时间用的-->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-openfeign</artifactId>
             </dependency>
     
     
     说明: 这个微服务也要将自己注册到eurake服务中
     ```

     

   -  编写启动类

     ```java
     @SpringBootApplication
     @EnableEurekaClient
     /*能够发现服务*/
     @EnableDiscoveryClient
     @EnableFeignClients
     public class Feign {
         public static void main(String[] args) {
             SpringApplication.run(Feign.class, args);
         }
     }
     
     ```

   -  编写配置文件application.yml

     ```yml
     server:
       #服务端口
       port: 9002
     spring:
       application:
         #指定服务名
         name:  cloud-client02
     
     eureka:
       client:
         #Eureka服务的地址
         service-url:
           defaultZone: http://localhost:8761/eureka/
       instance:
         #用于表示在猜测主机名时，服务器的IP地址应该与操作系统报告的主机名相对应。(注册服务和客户端如果在一台机器上则无需配置)
         prefer-ip-address: true
     ```

   -  编写调用别的服务的接口

     ```java
     /*实现微服务之间的调用*/
     /*填写服务的名字 =====在yml中定义的*/
     @FeignClient("cloud-client01")
     @Repository
     public interface Client {
         /*别的服务的controller的方法和请求路径,要保持一致*/
         @RequestMapping("/client/1")
         public Map index();
     }
     ```

   -  在自己的controller中调用 这个接口并得到返回值

     ```java
     @RestController
     public class IndexController {
         @Autowired
         private Client client;
     
         @RequestMapping("/find/client/1")
         public Map index() {
             return client.index();
         }
     }
     ```

     

### 3. 熔断器的设置

1.   feign中是自带熔断器的，需要在配置文件中设置，在调用别人的微服务里面设置

   ```yml
   server:
     #服务端口
     port: 9002
   spring:
     application:
       #指定服务名
       name:  cloud-client02
   
   eureka:
     client:
       #Eureka服务的地址
       service-url:
         defaultZone: http://localhost:8761/eureka/
     instance:
       #用于表示在猜测主机名时，服务器的IP地址应该与操作系统报告的主机名相对应。(注册服务和客户端如果在一台机器上则无需配置)
       prefer-ip-address: true
   
   #设置熔断器
   feign:
    hystrix:
     enabled: true
   ```

   

2.   配置完之后，使用这个调用接口的实现类配置 熔断之后执行的方法

   ```java
   @Component
   public class ClientImpl implements Client {
   
       @Override
       public Map index() {
           Map<String,String> map  = new HashMap<String, String>();
           map.put("error","你他妈的出错了");
           return map;
       }
   }
   
   ```

   

3.    在调用这个接口面加上熔断器的注解

   ```java
   import java.util.Map;
   
   /*实现微服务之间的调用*/
   /*填写服务的名字 =====在yml中定义的*/
   @FeignClient(value = "cloud-client01" ,fallback = ClientImpl.class)
   @Repository
   public interface Client {
       /*别的服务的controller的方法和请求路径,要保持一致*/
       @RequestMapping("/client/1")
       public Map index();
   }
   
   ```

4.  配置完之后的结果就是，在这个被调用的服务断开时间后执行实现类里面的方法



### 4. 配置网关  

1. ​     导入jar包的坐标

   ```java
       	<dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
           </dependency>
   ```

   

2.   编写启动类

   ```java
   @SpringBootApplication
   @EnableZuulProxy
   public class ApplicationManager {
       public static void main(String[] args) {
           SpringApplication.run(ApplicationManager.class,args);
       }
   }
   ```

   

3.   编写配置文件

   ```yml
   server:
     #服务端口
     port: 9009
   spring:
     application:
       #指定服务名
       name: cloud-manager
   eureka:
     client:
       #Eureka服务的地址
       service-url:
         defaultZone: http://localhost:8761/eureka/
     instance:
       #用于表示在猜测主机名时，服务器的IP地址应该与操作系统报告的主机名相对应。(注册服务和客户端如果在一台机器上则无需配置)
       prefer-ip-address: true
   
   # 网关的配置
   zuul:
     routes:
       clientone:
         path: /client/**
         serviceId: cloud-client01
       clienttow:
         path: /find/**
         serviceId: cloud-client02
   ```

   

4.   注意1 : 当我们的项目controller 路径是 /client/1 时  我们经过网关端口时间，路径要变为/client/client/1

5.  注意2 : 我们的controller里面要加上跨域访问的注解 ===>不加上也可以













