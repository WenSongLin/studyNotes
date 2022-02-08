# 0615springmvc高级

## 1.resturl(重要)

是一个特殊的url写法.简洁的url路径,url路径包含参数,

| 传统url           | 参数           | rest风格url | 请求方式 | 参数           |
| ----------------- | -------------- | ----------- | -------- | -------------- |
| /user/queryById   | ?uid=1         | /user/1?k=v | get      | 属于url一部分  |
| /user/addUser     | 请求体json或kv | /user       | post     | 请求体json或kv |
| /user/updateUser  | 请求体json或kv | /user       | put      | 请求体json或kv |
| /user/delUserById | ?id=1          | /user/1     | delete   | 属于url一部分  |

springmvc是支持rest风格的url.

```
注意点:rest风格的url,路径变量不适合太多,一般控制在三个以下.

查询数据:get请求,可以接收路径变量,可以接收?k=v变量

添加数据:post请求,可以接收路径变量,可以接收url上的kv变量,可以接收请求体的json数据,可以接收请求体的kv数据

修改数据:put请求,接收路径变量,接收?k=v,接收请求体json,不能接收请求体的k=v的数据

删除数据:delete请求,接收路径变量,接收?k=v,接收请求体json,不能接收请求体中的k=v数据
```



如果要求必须接收put请求的请求体中的kv数据,需要配置过滤器.

```
判断请求方式是put,delete的话,对请求体中的数据做转换处理成Map,把参数保存request对象.
```

```
 <filter>
        <filter-name>formContentFilter</filter-name>
        <filter-class>org.springframework.web.filter.FormContentFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>formContentFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```



总结:

如果前端提交数据格式是json,后端对于路径变量,对于?k=v,对于JSON随便接收.

如果前端提交数据是k=v+数据在请求体+提交方式是put或delete,后端需要配置过滤器.



## 2.静态资源处理

对前后端分离开发的项目,此项不常用.

对于前端未分离开发的项目,需要掌握此配置方法2.

在springmvc环境下,静态资源无法直接访问,因为tomcat下的DefaultServlet被DispathcerServlet进行路径覆盖.

方法1:配置DefaultServlet的映射路径

```
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.html</url-pattern>
        <url-pattern>*.css</url-pattern>
        <url-pattern>*.js</url-pattern>
        <url-pattern>*.jpg</url-pattern>
        <url-pattern>*.png</url-pattern>
    </servlet-mapping>
```

方法2:配置springmvc内的静态资源处理,推荐使用

```
 <!--springmvc对于静态资源通过名字叫default的servelt来处理.default-servlet-name="servlet名称"-->
<mvc:default-servlet-handler></mvc:default-servlet-handler>
```

方法3:对静态资源做映射.

```
 <!--/s/css/login.css=====/static/css/login.css-->
<mvc:resources mapping="/static/**" location="/static/"></mvc:resources>
<mvc:resources mapping="/page/**" location="/page/"></mvc:resources>
```



## 3.异常处理(重要)

dao层,service层异常抛给控制层,控制层框架进行异常处理.



1.springmvc的局部异常处理:使用@ExceptionHandler注解对一个处理器类中的所有方法异常进行处理.

缺陷:适用面仅限于当前处理器类,对其他处理器类没用.

优点:能够对不同的异常分别做处理.

```
@ExceptionHandler(Exception.class)
    public AxiosResult doException(Exception e){
        System.out.println(e.getMessage());
        return AxiosResult.error();
    }

    //该方法对ArithmeticException处理
    @ExceptionHandler(ArithmeticException.class)
    public AxiosResult doArithmeticException(Exception e){
        System.out.println(e.getMessage());
        //TODO 异常需要记录文件,保存在服务器文件上,开发人员定时的拿到异常文件
        return AxiosResult.error();
    }
```

2.springmvc的全局异常处理

优点:所有处理器类全生效

缺陷:所有类型的异常都进入该方法,无法对不同的异常做不同的处理方案.

```
@Component
public class GlobalExceptionHandler implements HandlerExceptionResolver {
    //解析异常的方法,同步开发中使用,异常开发也可以使用.
    @Override
    public ModelAndView resolveException(HttpServletRequest req, HttpServletResponse resp, Object o, Exception e) {
        System.out.println(e.getMessage());
        resp.setCharacterEncoding("UTF-8");
        PrintWriter writer = null;
        try {
            writer = resp.getWriter();
            AxiosResult error = AxiosResult.error();
            writer.print(JSON.toJSONString(error));
            writer.close();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        return null;
    }
}
```



3.使用ControllerAdvice注解实现的全局统一异常处理.(重要)

ControllerAdvice或RestControllerAdvice控制层增强注解

```
@RestControllerAdvice
public class MvcExceptionHandler {
    //defalut异常处理
     @ExceptionHandler(Exception.class)
    public AxiosResult doDefaultException(Exception e){
         //TODO 记录异常
        e.printStackTrace();//向控制台打印堆栈错误信息
        return AxiosResult.error();
    }

    //该方法对ArithmeticException处理
    @ExceptionHandler(ArithmeticException.class)
    public AxiosResult doArithmeticException(Exception e){
       e.printStackTrace();
        //TODO 异常需要记录文件,保存在服务器文件上,开发人员定时的拿到异常文件
        return AxiosResult.error();
    }

    @ExceptionHandler(MvcException.class)
    public AxiosResult doMvcException(MvcException e){
         e.printStackTrace();
         //TODO 记录异常
         return AxiosResult.error(e.getS());
    }
}
```



## 4.文件上传下载

springmvc中的文件下载,基于commons-fileupload组件.

- 添加commons-fileupload.jar;commons-io.jar
- 配置文件解析器对象,该对象对请求头Content-type:multipart/form-data的请求进行文件解析.

```
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"></property><!--对中文文件名支持-->
        <property name="maxUploadSize" value="50000000"></property><!--文件上传最大大小-->
        <property name="maxInMemorySize" value="10000000"></property><!--内存存储文件最大大小-->
        <property name="uploadTempDir" value="/upload/tmp"></property><!--12312312asdasfd.tmp-->
    </bean>
```

- 写文件上传下载的处理器方法

```
 @PostMapping("upload")
    public AxiosResult doUploadFile(MultipartFile myfile, HttpServletRequest  req) throws IOException {
        byte[] bytes = myfile.getBytes();//文件内容,字节数据,适合小文件
        InputStream inputStream = myfile.getInputStream();//文件内容,输入流,适合大文件,没啥用
        String originalFilename = myfile.getOriginalFilename();//文件名
        long size = myfile.getSize();//文件大小
        //存文件
        String realPath = req.getServletContext().getRealPath("/");
        String path = "/upload/"+originalFilename;
        //原生输出流写数据的过程
        IOUtils.write(bytes,new FileOutputStream(new File(realPath,path)));
        Map<String,Object> result = new HashMap<>();
        result.put("path",path);
        result.put("realName",originalFilename);
        result.put("uploadTime",System.currentTimeMillis());
        result.put("size",size);
        return AxiosResult.suc(result);
    }

    @GetMapping("down")
    public ResponseEntity doDownload(String path,HttpServletRequest req) throws IOException {
        String realPath = req.getServletContext().getRealPath("/");
        File f = new File(realPath,path);
        String filename = f.getName();
        byte[] bytes = FileUtils.readFileToByteArray(f);
        //设置响应头
        HttpHeaders headers = new HttpHeaders();
        headers.add("Content-Disposition","attachment;filename="+URLEncoder.encode(filename,"UTF-8"));

        return new ResponseEntity(bytes,headers,HttpStatus.OK);
    }    
```



## 5.拦截器interceptor

拦截器是框架中的一种概念,类似于Filter,低于Filter.

1.定义拦截器

```
public class LoginInterceptor implements HandlerInterceptor {

    //前拦截
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
//        HandlerMethod m = (HandlerMethod)handler;
//        Object bean = m.getBean();//handler类的bean对象
//        Method method = m.getMethod();//处理器方法
        HttpSession session = request.getSession();
        Object loginuser = session.getAttribute("loginuser");
        if(loginuser==null)
            throw new MvcException(S.NO_LOGIN);//返回给前端数据

        return true;
    }
}

```



2.配置拦截器

```
 <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <mvc:exclude-mapping path="/user/login"></mvc:exclude-mapping>
            <mvc:exclude-mapping path="/upload/**"></mvc:exclude-mapping>
           <bean class="com.javasm.commons.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```



## 6.springmvc的跨域处理

跨域:在前端分离的情况下,由于协议,ip,端口不同,违反了浏览器的同源保护策略,不允许请求.

需要服务器端响应的时候带上跨域的响应头,表示这是一个安全的请求.

方案1:在处理器类或方法上加@CorsOrigin注解,不建议使用,适用面小.

方法2:配置mvc:cors,不建议使用,与自定义拦截器结合使用时,出问题.

```
<!--该配置进行跨域,是基于一个springmvc内置的跨域拦截器对象进行响应头设置.而且该拦截器是最后一个执行-->
    <mvc:cors>
        <mvc:mapping path="/**"
                     allow-credentials="true"
                     allowed-origins="http://localhost:8088"
                     allowed-headers="*"
                     allowed-methods="*"/>
    </mvc:cors>
```

方法3:使用CorsFilter与DelegatingFilterProxy两者结合基于过滤器进行跨域

```
<!--跨域过滤器对象注册到spring容器-->
    <bean id="corsFilter" class="org.springframework.web.filter.CorsFilter">
        <constructor-arg name="configSource">
            <bean class="org.springframework.web.cors.UrlBasedCorsConfigurationSource">
                <property name="corsConfigurations">
                    <map>
                        <entry key="/**">
                            <bean class="org.springframework.web.cors.CorsConfiguration">
                                <property name="allowCredentials" value="true"/>
                                <property name="allowedMethods">
                                    <list>
                                        <value>GET</value>
                                        <value>POST</value>
                                        <value>HEAD</value>
                                        <value>PUT</value>
                                        <value>DELETE</value>
                                        <value>OPTIONS</value>
                                    </list>
                                </property>
                                <property name="allowedHeaders" value="*"/>
                                <property name="allowedOrigins" value="http://localhost:8088"/>
                            </bean>
                        </entry>
                    </map>
                </property>
            </bean>
        </constructor-arg>
    </bean>
```



```
<!--执行请求过滤,拦截到请求后,去容器中获取id为corsFilter的  对象,执行真正的跨域配置-->
    <filter>
        <filter-name>myCorsFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetBeanName</param-name>
            <param-value>corsFilter</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>myCorsFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

```



总结:

rest风格url:getMapping,postMapping,PutMapping,DeleteMapping+@PathValiable

全局统一异常处理:@RestControllerAdvice+@ExceptionHandler

文件上传下载:CommonsMultipartFileResolver+MuiltipartFile

拦截器:HandlerInterceptor-->preHandle

跨域过滤器:CorsFilter

