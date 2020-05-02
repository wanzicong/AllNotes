---
title: Springboot 基础
date: 2020-02-05 14:25:56
tags: 
	- SpringBoot
	- SpringBoot基础

---

# springBoot基础

## 1. 配置数据源信息（Druid）

  1. 使用的数据源默认是 jdbc.pool.DataSources

  2. jdbc 整合 略

  3. 数据源的的相关配置都在DataSourceProperties 类里面

  4. 数据源的自动配置原理

     1. DataSourceConfiguration类 根据配置进行数据源的设置连接池
        2. spring.datasource.type 指定自定义的数据源配置
           3. springBoot 支持数据源（。。。。。。。。好多）
              4. 可以自己创建数据源
                 5. 自动配置类当中可以自动进行数据源的 配置并且支持自动执行sql语句
                    1. **放在resources路径下 并且名字 是schema- *.sql   --- data-*sql文件类型**
                      2. **也可以使用配置的方式指定 执行文件的路径 以及名字** 
                             	6. 也有自动配置的操作数据库的类 JDBCTemplate 

5. 高级配置-整合druid 数据源

   1. 引入数据源

   2. ```java
      spring.datasource.type= 数据源的全类名
      ```

   3. 数据源的其他属性（连接数 。最大连接数 ）

   4. 写一个数据源配置的类进行数据源的配置

6. 配置druid 数据源的监控 和 自己配置数据源

   1. 管理后台数据的servlet（ServletRegistrationBean）
   2. 监控web的filter（FilterRegistrationBean）

```java
/**
 * 整合数据源
 */
@Configuration
public class MyDruid {

    @Bean
    //配置映射配置文件 内容
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druid (){
        return  new DruidDataSource();
    }
    //配置Druid的监控
    //1、配置一个管理后台的Servlet
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/main/*");(路径不包含项目名字)
        Map<String, String> initParams = new HashMap<>();
        //初始化参数
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        initParams.put("allow", "");//默认就是允许所有访问
        //拒绝谁访问
        initParams.put("deny", "192.168.15.21");
        bean.setInitParameters(initParams);
        return bean;
    }
    //2、配置一个web监控的filter
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.css,/main/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}

```

## 2.整合Mybatis

1. **使用注解版的方式**

   1. 在Service 类上面加入@Mapper 注解
   2. 也可以在启动类里面加入 @MapperScan(value = "全类名") -----就不用都添加@Mapper注解了

2. **自动获取自增主键（在注解版当中）**

   1. 该注解作用在 方法上

   2. ```java
      @Options(useGeneratedKeys = true ,keyProperty = "id")
      ```

   3. useGeneratedKeys属性 是否使用-------keyProperty属性 指定主键的名字

3. **自定义配置文件类**

   1. **自定义Mybatis 配置规则** 

      1. 在容器中添加 ConfigurationCustomizer 类

      2. ```java
         @Configuration
         public class Mybatis {
             public ConfigurationCustomizer configurationCustomizer(){
                 return  new ConfigurationCustomizer() {
                     @Override
                     public void customize(org.apache.ibatis.session.Configuration configuration) {
                         //设置属性的值
                             configuration.setMultipleResultSetsEnabled(true);
                     }
                 };
             }
         }
         ```

   2. **使用mybatis全局配置文件   和  mapper.xml 文件进行配置**

      1. mybatis 全局配置文件可以替换 类配置文件

      2. 使用映射文件操作数据库 需要的配置

         1. ```xml
            mybatis.config-location=classpath:配置文件路径
            mybatis.mapper-locations= classpath:处理数据库的接口类
            mybatis.type-aliases-package=com.project.model 扫描实体类的路径
            ```

      ```
      
      ```

4. mybatis 全局配置文件

        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <!DOCTYPE configuration
                PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-config.dtd">
        <configuration>
        
            <settings>
                <setting name="mapUnderscoreToCamelCase" value="true"/>
            </settings>
        </configuration>

   ```
   
   ```

5. mapper.xml 映射文件

       ```xml
       <?xml version="1.0" encoding="UTF-8" ?>
       <!DOCTYPE mapper
               PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
               "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
       <mapper namespace="com.atguigu.springboot.mapper.EmployeeMapper">
          <!--    public Employee getEmpById(Integer id);
       
           public void insertEmp(Employee employee);-->
           <select id="getEmpById" resultType="com.atguigu.springboot.bean.Employee">
               SELECT * FROM employee WHERE id=#{id}
           </select>
       
           <insert id="insertEmp">
               INSERT INTO employee(lastName,email,gender,d_id) VALUES (#{lastName},#{email},#{gender},#{dId})
           </insert>
       </mapper>

   ```xml
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
   
   
         <mapper namespace="com.project.mapper.UserMapper">
             <select id="xml" resultType="com.project.model.User">
                 select  * from user
             </select>
         </mapper>
   ```

   

   

      5. 使用yml 文件进行数据的配置

         ```yml
         spring:
           datasource:
         #   数据源基本配置
             username: root
             password: 123456
             driver-class-name: com.mysql.jdbc.Driver
             url: jdbc:mysql://192.168.15.22:3306/mybatis
             type: com.alibaba.druid.pool.DruidDataSource
         #   数据源其他配置
             initialSize: 5
             minIdle: 5
             maxActive: 20
             maxWait: 60000
             timeBetweenEvictionRunsMillis: 60000
             minEvictableIdleTimeMillis: 300000
             validationQuery: SELECT 1 FROM DUAL
             testWhileIdle: true
             testOnBorrow: false
             testOnReturn: false
             poolPreparedStatements: true
         #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
             filters: stat,wall,log4j
             maxPoolPreparedStatementPerConnectionSize: 20
             useGlobalDataSourceStat: true
             connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
         mybatis:
         ```

   6. 指定全局配置文件位置

          config-location: classpath:mybatis/mybatis-config.xml
          # 指定sql映射文件位置
          mapper-locations: classpath:mybatis/mapper/*.xml

        //指定项目启动 时间执行的sql 语句的位置

      ```java
       schema:
          - classpath:sql/department.sql
      	- classpath:sql/employee.sql
      ```

   ## 3.配置文件

   ### 1. 配置文件类型

   1.  property
   2.  yaml
   3.  优先级 applocation.property > application.yaml

   ### 2. 外部文件加载顺序

   1.  使用命令行运行项目时间，修改默认配置。
   2.  优先加载 带   .profile 

   

   ### 3. 配置文件加载位置

   

   1.    file/config
   2.    file/
   3.    resources/ config 文件下面
   4.    resources 的根目录下面
   5.    优先级从高到低 高优先级的配置文件会覆盖低优先级的配置
   6.    可以使用spring.config.location= 路径 （写在配置文件的里面） 来改变配置文件的位置 

   

   ### 4 配置文件的文件名字的含义

   1. application.dev.property  开发环境

   2. application.prod.property 生产环境

   3. 默认使用的是  application.property 

   4. 可以激活 开发环境 或者生产环境----- 在 application.property 中加入 spring.profile.active=dev 激活开发环境

   5. 使用yaml 配置文件中使用多文档块 

      ```yml
      server:
        port: 8080
      spring:
        profiles:
          active: dev
      ---
      server:
        port: 8081
      spring:
        profiles: dev
      
      ---
      server:
        port: 8082
      spring:
        profiles: prod
      
      ```

      

   

   

   ​	

   

   

   

   