## Spring Cache

### 四个重要注解

> 参数作用
>
> * value: 缓存的名称，每个缓存名称下面可以有多个key
> * key: 缓存的key 
> * condition:条件,满足条件时才缓存数据
> * unless:满足条件则不缓存
> * allEntries = true 用在@CacheEvict 中删除对应种类下的所有缓存

1. @CachePut
2. @Cacheable 执行方法前进行查询是否有缓存，没有则执行结束后添加缓存，有则直接返回
3. @CacheEvict 删除对应键值的缓存
4. @EnableCaching  开启缓存功能

### 在Spring Boot项目中使用Spring Cache的操作步骤(使用redis缓存技术)

1. 导入maven坐标

  ```xml
  	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
      </dependency>
  
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
      </dependency>
  ```

  

2. 配置application.yml

  ```yml
  spring:
      cache:
          redis:
             time-to-live: 1800000 #设置缓存有效期
  ```

  

3. 在启动类上加入@EnableCaching注解,开启缓存注解功能
4. 在Controller的方法上加入@Cacheable,@CacheEvict等注解，进行缓存操作
5. 缓存对象必须实现Serializable接口，使用注解缓存，默认缓存方法的返回值
