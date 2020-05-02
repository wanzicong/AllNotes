---
title: springboot-缓存使用-高级
date: 2020-01-30 21:54:08
tags:
	- springboot
	- 缓存
---

### 1.SpringBoot 与缓存

#### 1. 缓存的重要概念

1. cache接口
   1. 实现类 RedisCache 类 EhCache类 ConcurrentMapCache 类
   2. 进行对缓存进行 增删改查操作
2. cacheManager接口
   1. 作用: 管理缓存组件
3. @CacheAble 注解
   1. 作用在方法上
   2. 将方法返回结果加入缓存当中
4. @CacheEvict注解
   1. 清空缓存
   2. 作用在方法上
5. CachePut注解
   1. 更新缓存
   2. 作用在方法上
6. EnableCaching 注解
   1. 开启基于注解的缓存模式
7. keyGenerater
   1. 缓存数据key 生成策略
8. serialize
   1. 缓存数据 value 生成策略

#### 2.环境搭建

1. 略

2. 略

3. 配置Mybatis信息

   1. 配置数据源 在application.property 文件当中

      ```properties
      spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
      spring.datasource.url=jdbc:mysql://localhost:3306/spring?useSSL=false&serverTimezone=UTC
      spring.datasource.username=root
      spring.datasource.password=password
      ```

   2. 使用注解版的mybatis

      1. 在入口类上加上 @MapperScan（全类名-包）

         1. 作用扫描mapper 类

      2. 开启驼峰命名法

         ```properties
         mybatis.configuration.map-  underscore-to-camel-case=true 
         ```

4. 体验注解步骤

   1. 开启基于注解缓存

      1. 在入口函数加上@EnableCaching

   2. **标注缓存注解即可**

      1.  **@Cacheable** 
          1.   **cacheName / value（将方法的返回结果保存起来）**
          2.   **key**
          3.   **keyGenerator 生成器**
          4.   **cacheManager cacheResolver 缓存管理器**
          5.   **condition 指定符合条件下才缓存**
          6.   **unless : 否定缓存 =true 时不缓存**
          7.   **sync 异步模式**
          8.   ​    **@Cacheable(cacheNames = "emp")**
      2.  **@CacheEvict**
      3.  **@Cacheput**
      4.  **@Caching**
      5.  `说明: 使用这些注解的前提是已经配置好了 and缓存已经开启的状态下`

   3. 打印日志

      ```properties
      # 打印日志
      logging.level.com.cache.mapper（包名）=debug
      ```

   4. 使用缓存过程中必须手动启动 redis 缓存,否则报错无法连接缓存    

      1. @Cacheable(cacheNames = "emp")

   5. 访问缓存数据使用spEl表达式 

5. 使用@Cacheable注解中的key 属性 + spel 表达式自定义缓存名

   1. ```java
      key = "#root.methodName+'['+#id+']'
      ```

6. 使用@Cacheable注解中的keyGenerator属性 

   1. 指定自己的可以的生成器

   2. 在配置类当中加入代码

      ```java
      public class MyCache {
          /**
           * 自定义cache生成器
           */
          @Bean("mykeyGenerator")
          public KeyGenerator keyGenerator(){
            return new KeyGenerator(){
                  @Override
                  public Object generate(Object target, Method method, Object... params) {
                      return method.getName() + "[" + Arrays.asList(params).toString() + "]";
                  }
              };
          }
      }
      ```

   3. 在注解中加入

      ```java
      keyGenerator = "mykeyGenerator" (bean 的id)
      ```

7. 使用@CachePut更新修改后的 缓存

   1. 注意: 修改后的缓存 需要与之前的缓存保持一致

   2. 与sql 语句的update 对应 

   3. 与跟新的key 保持一致 

      ```java
       @CachePut(value = "emp" ,key "#result.id")
      ```

8. 使用@CacheEvict 注解 清除之前的缓存 

   1. 对应的是sql 语句的delete 语句

   2. 与之前的缓存id 保持一致

   3. ```java
      (属性)allEntries = true 删除缓存中的所有数据
      ```

   4. ```
      @CacheEvict((属性)value = "emp",     		 
      （属性）key="#id",（属性）allEntries = true)
      ```

   5. 属性:beforeInvocation 属性 

      1. (删除缓存操作)是否在方法执行之前执行默认= false

      2. 如果方法出错 清空缓存不会执行

      3. 设置 =true 时间无论方法是否出错 删除缓存都会执行

         

9. @Caching注解 是他们几个的组合注解

   ```java
    @Caching(
           cacheable = {
                   @Cacheable(value = "emp",key = "#lastName")
           },
           put = {
                   @CachePut(value = "emp" ,key = "#result.id"),
                   @CachePut(value = "emp" ,key = "#result.email")
           }
               )
   
   ```

10. @CacheConfig 注解 

    1. 作用在类上
    2. 指定所有的缓存value值    ====   cacheName = “名字”
    3. 指定 不同缓存注解的公共属性：keyGenerator，cacheManager cacheReslover 属性



#### 3 源码

	1. [SpringBoot高级](https://github.com/garxin/SpringBoot-Advanced)

#### 4 高级搭建 redis环境

  1. 缓存没有进行配置时间默认使用的是 CacheManager(缓存管理器) 为 ConcurrentMapCacheManager

         	1. 将数据保存到了ConcurrentMap<> 中

  2. 开发中经常使用一些缓存中间件 redis  memcached ehcache

  3. ===== 整合redis  =======

  4. 配置redis 在配置文件当中

  5. 配置后就可用 RedisTemplate 类 和 StringRedisTemplate 类 来操作redis

     ```java
      stringRedisTemplate.opsForValue().append("msg","子聪");
      stringRedisTemplate.opsForValue().set("msg","王子");
      String msg = stringRedisTemplate.opsForValue().get("msg");
     ```

  6. 自定义RedisTemplate的序列化器

     ```java
     	//方法名就是bean的id
     @bean
             Jackson2JsonRedisSerializer<Employee> serializer = new 	      	 Jackson2JsonRedisSerializer<Employee>(Employee.class);
             redisTemplate.setDefaultSerializer(serializer);
             return redisTemplate;
         }
     }
     ```

  7. =======测试缓存 =======

  8. 自定义CacheManager 来进行序列化的改造

  9. @Primary 注解作用：指定默认 的缓存管理器

     





 