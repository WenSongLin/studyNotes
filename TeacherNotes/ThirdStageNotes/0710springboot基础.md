# 0710springboot基础

## 1.springboot框架能够带来什么好处

ssm框架:DispatcherServlet加载springmvc.xml文件,创建子容器;listener加载spring.xml创建父容器.

​		mybatis整合:添加mybatis.jar;mybatis-spring.jar,注册mybatis相关的bean对象;

​		事务整合:添加spring-jdbc;spring-tx;注册事务相关的bean;

​		分页整合:添加pageHelper.jar;注册PageInterceptor分页bean;

​		redis整合:添加redis.jar;commons-pool2.jar;注册jedis相关bean;

​		...

​		每一个中间件的整合,需要做两件事情:添加中间件的依赖jar包;添加xml配置注册bean.

​		ssm框架搭建,维护起来都很繁琐.相比于ruby,python语言进行web框架开发来说,比较落伍.



springboot是对ssm的简化,不再注重于框架的整合,注重于业务的开发.

springboot是一个javaweb开发的全家桶应用.各技术都在主动向springboot框架进行靠拢;springboot的更新也是springboot对主流中间件进行主动集成的过程.



ssm框架是springboot的前置技术;springboot是springcloud前置技术;springcloud是springalibabacloud的前置技术;

springboot 2.5.2  2.4.5   2.3.3-2.3.5最稳定.



## 2.创建springboot项目

- 手工创建maven项目,然后引入springboot的父工程

1.在pom文件引入引入springboot父工程

```
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.5.RELEASE</version>
    </parent>
    
```

2.在pom中添加依赖

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

3.创建启动类

```
@SpringBootApplication
public class BootApp {
    public static void main(String[] args) {
        SpringApplication.run(BootApp.class,args);
    }
}
```



- 基于springboot官方的项目引导器来自动创建springboot项目,要求联网

  http://start.spring.io


## 3.认识springboot目录结构

src/main/java:启动类与源码

src/main/resources/static:保存静态文件(上传的文件,js,css,html)

src/main/resources/templates:同步开发中,保存视图文件(jsp,thymleaf)

src/main/resources/application.properties|applicaitn.yml:springboot框架核心配置

src/test/java:测试目录



## 4.认识springboot的pom文件

```
1.spring-boot-starter-parent父工程:定义了maven编译插件的jdk版本,定义了resources插件的编码格式,定义了resources目录下的识别文件.
	spring-boot-dependencies:父工程:统一管理依赖版本.

2.一定要掌握启动器的概念:jar包与配置类的整合.向springboot框架中集成其他技术组件时,更加轻松.

3.spring-boot-maven-plugin:当执行maven的package指令时,先通过maven-jar-plugin插件打jar包;再通过spring-boot-maven-plugin插件对jar包进行重打包,写入springboot的配置(版本,启动类名,lib文件夹位置...),使jar包可运行,通过java -jar 包名启动内嵌的tomcat服务器
```



## 5.认识springboot启动器starter

分为两类:

官方:spring-boot-starter-*,官方对主流技术主动的集成进入springboot          spring-boot-starter-data-redis

第三方:*-spring-boot-starter:第三方中间自身提供的向springboot中集成方案.      mybatis-spring-boot-starter

什么是启动器:  某个功能集成容器需要添加的依赖jar包;通过配置文件需要注册的核心对象.

以spring-boot-starter-web启动器说明:spring-web;spring-webmvc;spring-boot-starter-tomcat;spring-boot-starter-json;  最重要的依赖:spring-boot-starter,该依赖中又引入spring-boot-autoconfigure,该jar包中全部是配置文件(以配置类的形式来存在)

再以mybatis-spring-boot-starter说明:mybatis.jar;mybatis-spring.jar;mybatis-spring-boot-autoconfiguration.jar;该包中有MybatisAutoCofiguration配置类,该配置类中注册了SqlSESSIOnFactory对象.



后续学习其他组件时,向springboot中添加启动器后,要能够找到该组件的配置类.知道配置类中注册了那些bean对象到容器.



## 6.spring的注解配置

BeanFactory:

​	ApplicationContext:

​		ClasspathXMLApplicationContext:加载类路径下的xml配置文件

​		AnnotationConfigApplicationContext:加载类配置文件.

​		WebApplicationCotnext:

​			XmlWebApplicationContext:在web环境下,加载类路径下的xml配置文件.



@Configuration:定义配置类

@ComponentScan:开启包扫描

@Import:导入其他配置类

@PropertySource:加载properties配置文件

@ConditionOnClass:当前工程类路径下有指定.class字节码文件生效

@ConditionOnBean:当前容器中有指定类型的bean对象才生效



## 6.认识springboot启动类,主类

@SpringBootApplicatin注解

```
@SpringBootConfiguration:本质就是Configuration注解,表示当前类是一个配置类
@EnableAutoConfiguration:该注解是一个复合注解,有AutoConfigurationPackage与Import两个子注解.
@ComponentScan:开启包扫描,没有指定扫描包范围.
```



@EnableAutoConfiguration注解:

```
@AutoConfigurationPackage:自动配置扫描包范围.
	该注解中得到启动类的包名,把包名注册给ComponentScan注解中的basePackage.进行包扫描
	
	
@Import({AutoConfigurationImportSelector.class}):通过selecter选择器,批量找到jar包中符合条件的配置类.
AutoConfigurationImportSelector对象中的selectImports方法,负责查找配置类,并进行过滤

String str="org.springframework.boot.autoconfigure.EnableAutoConfiguration"

Map<String,List<String>> map =loadSpringFactories(classloader)会去所有jar包查找"META-INF/spring.factories"

List<String> autoConfigs = map.get(str);//130个

//filter,对ConditionOnXXX进行配置类的筛选.得到有效配置类

//Import注解,把有效配置类导入主配置类.
```



SpringApplication.run方法

```
WebApplicationContext context = new AnnotationConfigServletWebServerApplicationContext(主配置类)

context.refreshContext();//加载配置类,创建容器,开启解析SpringBootConfiguration注解
```

总结:

第一步:run方法:初始化AnnotationConfigServletWebServerApplicationContext容器对象,再创建容器时,会加载主配置类.



第二步:@SpringBootApplicatin注解:

​	表示当前类是主配置类;

​	开启包扫描并把当前主配置类的包名作为包扫描范围;

​	通过selecter选择器对象去jar包总查找并筛选有效的XXXXXXAutoConfiguration配置类,导入主配置;



## 7.springboot配置文件详解

分两种格式:

properties:kv

yml: 阅读性比较好,因为前缀相同,会自动排版到同一级别.适合配置量大的时候使用.

yml文件注意点:(重点)

​	级别使用空格分割,对齐;

​	冒号后面必须使用空格分割值;

​	字符串值可以使用引号括,也可以不阔;

​	可以通过spring:profiles:active:引入其他yml文件;

​	可以在yml中配置日期,集合,map等格式的数据;



读取配置文件中的自定义数据:(重点)

- ```
  @Value注解获取少数的数据
  ```

- ```
  @ConfigurationProperties(prefix = "jdbc")来批量获取指定前缀的数据.
  ```

注意点:如果把配置信息拆分到不同的properties文件中,文件命名不以application开头,则需要通过@PropertyResources来引入文件.如果文件命名以application-XXX开头,则需要在主配置类通过active指定激活文件名称.



2.3.5版本中是properties文件优先级高.



## 8.springboot项目部署





## 9.其他



