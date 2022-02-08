# 0712springboot使用

## 1.springboot项目打包部署

不重要,了解

- jar包部署:通过spring-boot-maven-plugin插件对jar包进行重写,写入启动类的信息,直接运行jar包.

```
<plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
</plugins>
```

```
java -jar 包 
```

- war包部署:把项目打war包,war包部署到服务器上的第三方tomcat下的webapps目录,运行第三方tomcat,(weblogic)

```
1.修改打包方式:<packaging>war</packaging>
2.指定war包名:<finalName>boot</finalName>
3.把内嵌的tomcat启动器.<scope>provided</scope>
3.从SpringBootServletInitializer抽象类派生子类重写configure方法,该类即是程序的入口类,类似web.xml
```



## 2.静态资源处理

前端未分离开发场景中使用.静态资源(css,js,img,音频,视频,文件,静态html文件).

springmvc静态资源处理:

<mvc:default-servlet-handler>通过tomcat下的DefaultServlet来处理静态资源;

<mvc:resources location="/static/;" mapping="/**">,通过springmvc内部的静态资源处理器对象,对/static开头的url进行处理.



springboot中静态资源处理:再WebMvcAutoConfiguration中的有实现WebMvcConfigurer接口,通过addResourceHandler来指定了静态资源的处理方式.



/**--->四个静态资源目录,优先级由高到低

默认对所有请求,都去4个路径下去查找资源;找不到才执行handler方法

```
"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"
```



统一自定义静态资源url与路径:

```
spring:
  mvc:
    static-path-pattern: /s/**
  resources:
    static-locations: classpath:/views/,classpath:/js/
```



欢迎页,banner,ico图标统统不重要.



## 3.WebMvcConfigurer接口扩展

非常重要

addInterceptor:配置拦截器

```
 @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor).addPathPatterns("/**").excludePathPatterns("/rolelist", "/goadd");
    }
```

addCorsMapping:基于拦截器进行跨域配置,不建议使用

```
//跨域配置<mvc:cors >,基于跨域拦截器生效,一旦自定义了拦截器
//    @Override
//    public void addCorsMappings(CorsRegistry registry) {
registry.addMapping("/**").allowCredentials(true).allowedHeaders("*").allowedMethods("*").allowedOrigins("http://localhost:8088").exposedHeaders("token");
//    }
```

建议使用CorsFilter进行跨域配置,不受拦截器影响.

```
@Configuration
public class CorsConfig {

    @Bean
    public FilterRegistrationBean createCorsFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://localhost:8088");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        config.addExposedHeader("token");
        source.registerCorsConfiguration("/**", config);
        FilterRegistrationBean bean = new FilterRegistrationBean(new CorsFilter(source));
        bean.setOrder(0);
        return bean;
    }
}
```



静态资源处理:4个静态资源目录,/META-INF/resources--resources--static--public;

打war包的方式:修改打包方式;tomcat私有化;springbootSevletInitializer派生子类重写config方法指定主配置类.

WebMvcConfigurer接口中的addInterceptor配置拦截器;addCorsMapping来配置跨域拦截器.

自定义配置类:通过@Configuration来定义配置类,注册CorsFilter跨域过滤器对象,取代addCorsMapping跨域拦截器用法.



## 4.thymeleaf模板引擎

类似jsp技术,同步开发中使用.视图模板引擎的作用就是在静态html页面中使用引擎语法能够解析动态数据,生成新的不同的html页面.

springboot框架中不支持jsp引擎,因为内嵌的tomcat启动器中,没有jsp环境.springboot框架提供了thymeleaf启动器对该引擎进行支持.



jsp:  编写userlist.jsp-->编写html模板-->通过jstl+el+jsp标签在模板中填充动态数据-->当用户第一次访问userlist.jsp文件,jsp引擎会把改文件翻译成userlist_service.java类文件,jvm加载.java类文件,编程为userlist_serice.class文件,实例化userlist_service对象,执行对象中的service(req,resp)方法,该方法内拼接html字符串.返回客户端html文本.

thymleaf:编写userlist.html-->写html静态模板(div.table.h,span..css)-->通过thymeleaf语法(属性表达式,链接表达式,文档表达式,thymeleaf相关属性)在静态模板中填充动态数据-->当用户第一次访问时,thymeleaf引擎把html文件加载到内存中进行解析语法规则,并进行缓存-->解析动态数据拼接html文本返回客户端.



- 添加thymeleaf启动器,spring-boot-starter-thymeleaf

该启动器内添加了thymeleaf的核心jar包;thymleaf与spring的整合jar包;

​		   ThymeleafAutoConfiguration预配置生效,加载ThymeleafProperties配置对象,该对象中默认视图前缀"classpath:/templates/";后缀.html;注册了ThymeleafViewResolver视图解析器对象到容器.

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

- 使用thymeleaf语法:

  表达式:

```
属性表达式:${}.作用:获取request,session,servletContext域中的数据
	${request中的key}
	${session.key}或者${#session.getAttribute('key')}
	${#servletContext.getAttribute('key')}
	
链接表达式:@{}.作用:为src,href等链接属性补全根路径.
	@{/路径地址}
	@{/role/goadd(rid=${u.uid},rname='aaa')}
	
文档表达式:~{},作用:进行html公共页面的局部嵌套.
```

​	th属性:

```
th:text:结合这属性表达式${}来获取数据,可以简写成[[${}]],向双标签中间赋值
th:value:结合属性表达式${}来获取数据,向单标签赋值.
th:href:结合链接表达式@{}来补全链接根路径
th:src:结合链接表达式@{}来补全链接根路径
th:action:结合链接表达式@{}来补全链接根路径
th:replace:对公共页面进行嵌套,结合~{}可以进行全页面嵌套,结合th:fragment可以进行局部片段嵌套
th:insert:对公共页面进行嵌套
th:fragment:结合文档表达式,进行页面的局部嵌套
th:each:对集合数据进行遍历.th:each="u,s : ${集合的key}"
```



## 5.springboot异常处理

掌握springboot自身的异常机制,面试用;开发中仍然全局统一异常处理.

ErrorMvcAutoConfiguration配置类,

```
DefaultErrorAttributes:是ErrorAttributes接口的实现类,重写了getErrorAttributes方法,解析request,返回一个Map形式错误数据集合(status,path,time,message,error).


BasicErrorController:这是一个控制层处理器对象,映射路径是/error.当程序出异常,tomcat会去找映射路径为/error的服务.
	public ModelAndView errorHtml()  处理浏览器的同步请求,得到map形式的错误数据,然后去解析错误视图(通过DefaultErrorViewResolver对象首先识别当前工程下的模板引擎,判断templates/error/404.html;找不到404.html,再找4xx.html;找不到4xx,则去4个静态资源路径下去查找404.html,4xx.html)
	
	
	public ResponseEntity<Map<String, Object>> error  处理异步请求.调用DefaultErrorAttributes对象getErrorAttributes方法来获取错误数据.
		
error-->StaticView:默认的静态错误视图.名称是error.等价于error.html
```



面试题:

如何自定义错误视图?

```
如果要定义动态错误视图,需要依赖模板引擎,在templates目录下加error/4xx.html   5xx.html
如果仅需要静态错误视图,则可以在4个静态资源路径下添加error/4xx.html  5xx.html
```

如何自定义错误数据?

```
从DefaultErrorAttributes对象派生子类,重写getErrorAttributes方法,该对象注册容器.
```



在实际项目开发中,仍然使用RestControllerAdivce+ExceptionHandler来对异常进行统一的处理.返回有效的json数据.



## 7.日志

把springboot中默认的logback日志,替换为log4j2与slf4j

- 取出spring-boot-starter-logging,引入spring-boot-starter-log4j2

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
```

- 加入log4j2.xml即可;
- 如果需要用异步日志,再加入disruptor.jar包



## 8.mybatis

学习了mybatis启动器中的配置类,学习Druid启动器中的配置类DruidDataSourceAutoConfiguration

- 添加mysql驱动,添加druid启动器;为了注册DruidDataSource对象.

```
 <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!--凡是需要注册bean到容器,都有启动器.因为要有配置类-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.6</version>
        </dependency>
        
           <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
```



```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/crm?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&serverTimezone=GMT%2B8&useSSL=false
    username: root
    password: root
#    type必须指定,在引入mybayis启动后,不指定type的话,会自定创建HikariCP连接池
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      initial-size: 5
      min-idle: 5
      max-active: 100
```

- 了解MybatisAutoConfiguration注册SqlSessionFactory对象.

```
在MybatisAutoConfiguration配置类中,注册SqlSessionFactory对象,要依赖于mybatis配置信息
mybatis:
  type-aliases-package: com.javasm
  mapper-locations: classpath:mapper/*.xml
```

- 在启动类上添加MapperScan注解对dao包进行扫描创建代理对象.

```
@MapperScan("com.javasm.*.dao")
```



## 9.事务

```
在启动类上开始事务注解的识别@EnableTransactionManagement,在服务层的增删改方法上加    @Transactional注解即可.

因为在DataSourceTransactionManagerAutoConfiguration中已经自动注册事务管理器对象到容器,依赖dataSource.
```

 

## 10.分页

- 添加分页启动器

该启动器中有PageHelperAutoConfiguration,该配置类中有addPageInterceptor方法会实例化PageInterceptor对象,并把分页拦截器对象添加到SqLSessionFactory对象.可以直接使用.

```
    <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.5</version>
        </dependency>
```

