集成组件步骤:pom中加依赖或启动器;全局ctrl+n查找预配置类;查看XXXXXXProperties配置对象;在yml文件中进行数据配置;在bean对象可以注入组件bean.



有时候需要在启动类上开启注解识别





# 0713springboot

## 1.redis集成

- 添加spring-boot-starter-data-redis启动器,如果要使用jedis作为客户端的话,需要排除lettuce,引入jedis依赖

- 配置redis服务器链接信息:RedisAutoConfiguration配置,RedisProperties配置对象

```
spring:
# redis链接配置
  redis:
    host: 127.0.0.1
    port: 6379
    password: root
    jedis:
      pool:
        min-idle: 10
#    lettuce:
#      pool:
#        min-idle: 10
```

- redis在springboot使用方式:
  - 第一种:基于RedisTemplate工具类:

配置类中自动注册的bean对象:

```
RedisTemplate<Object, Object>
	该对象内部把key,value,hashKey,hashValue都通过ObjectOutputStrema与ObjectInputStream进行序列化与反序列化,要求对象必须从Serializable接口派生.

StringRedisTemplate
	该对象内要求key,value,hashkEY,hashValue都是String
```



需要自定义序列化方式:

```
@Bean("redisTemplate")
public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
    RedisTemplate<String, Object> template = new RedisTemplate();
    template.setKeySerializer(RedisSerializer.string());
    template.setValueSerializer(RedisSerializer.json());
    template.setHashKeySerializer(RedisSerializer.string());
    template.setHashValueSerializer(RedisSerializer.json());
    template.setConnectionFactory(redisConnectionFactory);
    return template;
}
```



- 基于注解使用redis

没有RedisTemplate灵活,不建议使用.了解

```
1.先开启redis注解的识别.
2.在查询方法上使用Cacheable注解
3.在删除与修改方法是使用CacheEvict来清除缓存中指定的数据.
```



## 2.异步任务

配置类:TaskExecutionAutoConfiguration,该类中注册了ThreadPoolTaskExecutor异步任务线程池对象,有execute方法执行Runable对象.

使用方法:

- 手工执行异步任务,建议使用.最好封装AsyncFactory,把项目中所有的异步对象的生成都在工厂中实现.

```
在需要异步任务的对象中注入ThreadPoolTaskExecutor对象,执行exexute方法
```

- 基于注解执行异步方法

```
1.开启异步任务注解的识别@EnableAsync
2.在需要异步执行的方法上加@Async注解,表示该方法异步执行.
```



## 3.定时任务

配置类:TaskSchedulingAutoConfiguration,该类中注册了ThreadPoolTaskSchedulr定时任务线程池对象,有schedule方法来调度任务重复执行.该线程池默认大小是1.

- @EnableScheduling//@Scheduled定义任务注解识别.由于线程池只有1个线程,所有多任务是串行任务.
- 并行任务使用:
  - 使用async注解+Scheduled注解一起定义任务.建议使用.异步定时任务
  - spring.task.scheduling.pool.size指定调度线程池的大小.让多个任务在不同的线程中进行调度.



## 4.邮件

- 添加spring-boot-starter-mail启动器,自动注册JavaMailSender对象.

- MailSenderAutoConfiguration配置类中注册了JavaMailSenderImpl对象,该对象需要setHost,port,username,password

```
JavaMailSender对象即可.
MimeMessageHelper对象.
```




## 5.fastdfs-client

FastFileStorageClient对象,提供了文件的上传下载删除等方法.



## 6.mybatis-plus

