# 0614springmvc

## 1.springmvc作用

springmvc是spring框架中一个模块,简化servlet开发.对servlet开发中繁琐的代码通过反射进行处理.

springmvc是控制层框架,职责接收数据(kv,json,文件),返回数据(视图,json,byte[]).



servlet开发:getParameter获取参数非空判断以及转型,一个servlet只能处理一个业务功能单一,接收json数据繁琐.返回json数据繁琐.

​	

前端传递后端参数,提交方式,参数在哪里,参数什么格式,是否加密?

get:

​	位置:参数在url后追加,http://域名?key=value&key=value

​	格式:?key=value&key=value

​	未加密



post:

​	位置:参数在http协议的请求体.

​	格式:key=value&key=value或者{key:value,key:value}

​	

关于参数加密,可以通过https协议对数据进行加密,金钱成本

​	                 可以手工加密,代码复杂



## 2.入门springmvc使用

- 添加spring的依赖包;添加springmvc的依赖包;

```
spring-core;spring-beans;spring-context;spring-expression;
commons-loging;
spring-aop;spring-aspect
aspectj;
cglib;
spring-web;
spring-webmvc
```



- 创建springmvc.xml,仍然是spring风格的xml文件.

```
 <!--识别@Controller,@Service,@Repository,@Component,Autowired,Resource-->
    <!--@Bean,@Value.@PostConstract,@PreDestroy,@Scope-->
    <context:component-scan base-package="com.javasm"></context:component-scan>

    <!--配置springmvc最核心注解的识别:@RequestMapping-->
    <!--处理器映射器-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>
    <!--处理器适配器-->
    <bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"></bean>
    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/page/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
```



- 在web.xml配置springmvc最核心的对象DispatherSerlvet

```
   <!--httpServlet中的方法:init,service,doget,dopost,destory-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--初始化,init方法中解析-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--更改servlet的实例化时间-->
        <load-on-startup>1</load-on-startup>
    </servlet>

    <!--所有请求都会进dispatcherServlet,jsp不进-->
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```



- 写控制层对象,并注册springmvc容器(使用@Controller)

```
//springmvc对容器中使用Controller|RequestController的bean对象,会继续解析RequestMapping注解.
@Controller
public class HelloHandler {
    //形参使用包装类对象
    @RequestMapping("hellomvc")
    public String hello(String uname,Double udouble,Integer uint){
        System.out.println("hello springmvc!"+uname+"-"+udouble+"-"+uint);
        return "main";//视图名
    }
}
```



## 3.springmvc执行流程

- 1.tomcat启动:进入DispachterServiet的init方法,WebApplicationContext容器初始化,url映射.

- 2.请求:http://localhost:8080/hellomvc,进入DispachterServiet的service方法,进行url解析,得到请求uri:/hellomvc字符串,通过处理器映射器找到HandlerMethod处理器方法,返回DispathcerServlet

- 3.处理请求:DispatcherServelt调用处理器适配器对象,执行处理器方法.主要一下任务:(
  ​	HttpMessageConveter获取前端请求参数,进行转型,进行封装;
  ​	执行处理器方法,传入封装参数;
  ​	得到处理器方法的返回值;

  把返回值再交给DispathcerServlet.

- 4.结果处理:DispatcherServlet得到返回值ModelAndView对象后,调用jsp视图解析器对象拼接完整视图路径,进行视图转发.

- 5.视图渲染:jsp引擎负责生成静态的html字符串通过输出流输出前端,浏览器进行显示.



## 4.springmvc几个核心对象

- ```
  DispatcherServlet:springmvc的前端控制器对象,该对象中的init方法中初始化WebApplicationContext对象.
  ```

- ```
  RequestMappingHandlerMapping:spirngmvc的处理器映射器对象,在初始化容器时,解析Controller注解的bean对象中的RequestMapping注解,进行url映射.把url字符串映射到一个处理器方法上.
  httpmvc--->HandlerMethod(SysuserHandler.hello(String uname,Double udouble,Integer uint);)
  ```

- ```
  RequestMappingHandlerAdapter:springmvc的处理器适配器对象,负责执行处理器方法的.(
  	获取前端请求参数,进行转型,进行封装;
  	执行处理器方法,传入封装参数;
  	得到处理器方法的返回值ModelAndView,把返回值再交给DispathcerServlet
  )
  ```

- ```
  InternalResourceViewResolver:jsp视图解析器,配置视图统一的前缀路径与统一的后缀格式.
  ```

- ```
  HttpMessageConverter:消息转换器,负责接收参数,格式参数;负责把返回值结果转json.
  ```



## 5.kv数据接收(前端ajax常用,前端axios用不上)

直接加形参.

```
//如果接收k=v&k=v数据,Date类型采用DateTimeFormat指定格式
//如果接收k=v&k=v数据,如果形参时简单类型,则形参名与表单参数名一致;
//如果接收k=v&k=v数据,如果形参是复杂对象,则对象成员变量名与表单参数名一致;
//如果接收k=v&k=v数据,可以在形参加@RequestParam注解,指定参数默认值.
```



## 6.json数据接收(前端axios的话常用)

springmvc内部依赖jackson组件进行JSON字符串转换.

- 添加jackson的3个依赖包;jackson-core;jackson-annotation;jackson-databind
- 在方法的形参加@RequesetBody注解,表示接收json字符串,并转为相应类型的对象(实体,Map,List)

- 如果json字符串有日期字符串,后端使用Date或LocalDate对象时,可以通过@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")//把字符串中的日期,转成对象;把对象中的Date转json字符串时,日期格式


## 7.返回视图

处理器方法返回值使用ModelAndView对象,指定视图名.同时配置视图解析器对象.



## 8.返回json

方法1:处理器方法上或处理器类上加@ResponseBody

​	表示方法的返回值通过MappingJackson2HttpMessageConverter消息转换器对象,把返回值转json字符串返回给前端.不再执行视图解析器对象,转发页面.



方法2:处理器方法返回值ReponseEntity(常用)

​	相比于ResponseBody注解,该方法更加强大,可以灵活的指定响应头,响应体,状态行数据.一般应用文件下载,token传输.



方法3:处理器类使用@RestController取代@Controller注解.(常用)

​	该注解等价于@Controller+@ResponseBody注解.



注意:jackson组件可以通过@JsonInclude(JsonInclude.Include.NON_NULL) 注解指定非null序列化.

## 9.限定请求方法(重要)

为了接口的安全,第一步限定接口的提交方式,get或post或put或delete

方法1:@RequestMapping(path = "add",method = RequestMethod.POST)

方法2:使用@GetMapping,PostMapping,PutMapping,DeleteMapping进行url映射,限定请求方法.



## 10.处理器转发与重定向(不重要)

只用在同步开发中.

```
方法返回值的视图名使用forward: 或 redirect:开头,表示转发或重定向到新的处理器方法.
```



## 11.数据乱码(不重要)

get请求:无乱码.

​	参数在url后面追加,tomcat7以上的版本url都支持中文.

post请求:

​	如果参数追加url后,无乱码;

​	如果参数放在请求体中,k=v格式数据中文有乱码;json格式数据中文无乱码;



k=v格式数据中文有乱码,是因为tomcat内部对请求体和响应体中的数据,编码格式采用的ISO-8859-1西文编码.不支持中文.

json无乱码是因为MappingJackson2HttpMessageConverter消息转换器中吧json字符串转对象时,采用的是utf-8编码,支持中文.

总结:只有当请求体中的数据格式时k=v的时候才会有中文乱码.解决方式是配置编码过滤器

```

    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



## 12.获取servlet对象引用

HttpServletRequest:请求对象,瞬时状态.在处理器方法加入形参

HttpServletReponse:响应对象,瞬时状态.在处理器方法加入形参

HttpSession:用户会话对象,被所有request共享,在处理器方法加入形参

ServletContext:全局唯一的上下文对象,被所有request,session都共享.必须通过request或session对象来获取.



总结:

@RequestMapping注解,用在类上,进行url第一层映射.

@GetMapping,@PostMapping,用来处理器方法上,进行url第二层映射.

@RequestParam注解,用在处理器方法的形参上,给k=v格式的简单类型数据默认值.只能用在简单类型上.

@RequestBody注解,用在处理器方法的形参上,用来获取json格式的数据,只能用在复杂对象上(实体,map,list)

@ReponseBody注解,用在类或方法上,表示处理器方法的返回值走json转换器转字符串返回前端;

ReponseEntity对象,用在处理器方法返回值类型,表示处理器方法的返回值走json转换器转字符串返回前端,该对象可以指定响应头,响应体

@RestController注解,用在类上,表示本类中处理器方法的返回值走json转换器转字符串返回前端;

@DateTimeFormat注解,用在Date或LocalDate类型的变量上,指定前端传递日期字符串格式.前提是参数格式k=v

@JsonFormat注解,用在Date或LocalDate类型的变量上,指定前端传递日期字符串格式.前提是参数格式{k:v}

@JsonInclude注解,用在类上,指定非null字段才进行序列化json字符串.

BeanUtils工具包,用来把Map中的数据分装到实体对象中.



springmvc的核心对象与执行流程:

















