# 0610spring

## 1.什么是spring(容器Container)

spring是一个第三方的容器框架,用来管理对象(创建,状态,销毁).

spring是反射工厂模式的应用.



同时,spring框架还提供了很多工具类:RestTemplate,JdbcTemplate,RedisTemplate...

​         spring框架还提供了管理器对象:事务管理器对象.

​	 spring还提供了线程池的支持.支持异步任务,支持定时任务.



spring带来的最大的好处:层与层之间的解耦,达到下层变动,上层不变



## 2.入门使用

- 添加spring的4个核心包(beans,core,context,expression);一个commons-logging日志包;总共5个jar包.
- 创建spring风格的xml配置文件.

```
<bean id="sysuserController" class="com.javasm.controller.SysuserController"></bean>
```

- 加载xml文件,实例化spring的容器对象.

```
//1.ClassPathXmlApplicationContext:加载类路径下的xml文件,初始化BeanFactory工厂对象,创建bean
ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
```

- 测试使用.

```
//2.根据id从容器中获取bean对象
SysuserController uc = (SysuserController)ac.getBean("sysuserController");
System.out.println(uc);

//3.根据类型从容器中获取bean
SysuserController uc2 = ac.getBean(SysuserController.class);
System.out.println(uc2);
```

总结点:

```
* 总结:
* 1.spring管理的对象,默认是单例的
* 2.从spring中获取对象,有两种方式,byName,byType
* 3.有了spring后,除了实体类,以后不再new对象,是把对象注册到spring容器,然后需要用的时候,从容器中get.
```



## 3.spring中的几个概念

IOC:控制反转,对象的管理由应用自身维护交给第三方的容器来管理,称为控制反转.

​	                 对象注册spring容器的过程.

```
 <!--spring通过反射SysuserServiceImpl对象-->
    <bean id="sysuserService" class="com.javasm.service.impl.SysuserServiceImpl"></bean>
```

DI:依赖注入.bean对象依赖谁,注入谁.对象之间的关系,也交给spring来管理.

```
  <bean id="sysuserController" class="com.javasm.controller.SysuserController">
        <!--SysuserController依赖的us成员变量,也在容器,通过property标签进行set注入-->
        <property name="us" ref="sysuserService"></property>
        <property name="str" value="aaa123"></property>
    </bean>
```



## 4.spring的容器类图:

```
//0:了解下spring的容器接口与实现类的结构(类图):
//BeanFactory-->DefaultListableBeanFactory
//BeanFactory-->ApplicationContext-->FileSystemXmlApplicationContext|ClasspathXmlApplicationContext|AnnotationConfigApplicationContext
//---------->XmlWebApplicationContext|AnnotationConfigWebApplicationContext
```



## 5.bean标签属性

- ```
  id:bean的名称
  ```

- ```
  class:bean的类名
  ```

- ```
  scope:对象的单例多例状态.singleton|prototype,默认值是singleton单例,用处不大,除非Struts2
  ```

- ```
  lazy-init:true仍然单例状态,延迟初始化
  ```



## 6.指定bean对象的初始化方法(重要)

​	方法1:对jar包中类,使用bean标签的init-method

- ```
  一般用在jar包中的注册spring容器时,指定初始化与销毁的方法
  init-method="init":指定bean对象的初始化方法
  destroy-method="close":指定bean对象的销毁方法
  ```

  方法2:对自定义的类,使用InitializingBean接口

- ```
  如果自定义的类注册spring容器,需要指定初始化与销毁方法,可以通过InitializingBean接口实现.
  public class SysuserServiceImpl implements ISysuserSerivce,InitializingBean {
      //当该bean被实例化完成后,立即执行afterPropertiesSet方法进行初始化工作.
      @Override
      public void afterPropertiesSet() throws Exception {
          System.out.println("初始化方法.......");
      }
  }
  ```


## 7.ioc的实现方式(重要)

IOC:把对象注册到spring容器.

### 方法1:bean标签,指定类名的方式进行bean注册;(重要)

使用场景:为jar包中的类注册容器用的.

```
 <bean class="com.javasm.service.impl.SysuserServiceImpl"></bean>
```

### 方法2:工厂注册bean

使用场景:复杂对象的注册容器,可以使用工厂注册bean

目标:把SqlSessionFactory对象注册到spring容器

- 静态工厂注册bean

```
<!--静态工厂注册bean-->
    <!--把SessionFactoryBean.createFactory方法的返回值注册到容器-->
    <bean id="sqlSessionFactory" class="com.javasm.factory.SessionFactoryBean" factory-method="createFactory"></bean>
```

- 实例工厂注册bean

```
 <!--工厂对象-->
    <bean id="factoryBean2" class="com.javasm.factory.SessionFactoryBean2" ></bean>
    <!--factoryBean2.createFactory()-->
    <bean id="sqlSessionFactory" factory-bean="factoryBean2" factory-method="createFactory"></bean>
```

### 方法3:包扫描(重要)

使用场景:自定义的类注册容器

```
<!--对指定包下的所有类进行递归扫描,通过反射检测类上是否带有指定注解,有的话,则把改类注册容器-->
    <!--
        @Controller:注解控制层bean
        @Service:注册服务层bean
        @Repository:注解dao层bean
        @Component:注册其他所有bean
        @Scope("prototype"):指定bean的多例状态,默认单例
        @PostConstruct:指定初始化方法
        @PreDestroy:指定销毁方法
    -->
<context:component-scan base-package="com.javasm"></context:component-scan>
```



## 8.di的实现方式(重要)

di:依赖注入,维护对象之间的依赖关系.

### 方法1:set注入(重要)

要求成员变量必须有set/get方法

```
<bean id="ds" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
        <!--set注入,调用对象的set方法进行值得注入;ref|value:ref用来引用其他bean的id,value用来给简单类型赋值-->
        <property name="url" value="jdbc:mysql://localhost:3306/crm"></property>
        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
        <property name="username" value="root"></property>
        <property name="password" value="root"></property>
        <property name="initialSize" value="5"></property>
    </bean>
```



### 方法2:构造器注入(重要)

```
<!--new SysuserDaoImpl(ds,12)-->
    <bean class="com.javasm.dao.impl.SysuserDaoImpl">
        <!--index:形参索引;name:形参名-->
        <constructor-arg index="0" ref="ds"></constructor-arg>
        <constructor-arg name="sint" value="12"></constructor-arg>
    </bean>
```

### 方法3:集合注入

```
<!--new SysuserDaoImpl(ds,12)-->
    <bean class="com.javasm.dao.impl.SysuserDaoImpl">
        <!--index:形参索引;name:形参名-->
        <constructor-arg index="0" ref="ds"></constructor-arg>
        <constructor-arg name="sint" value="12"></constructor-arg>
        <property name="strList">
            <list>
               <value>aaa</value>
               <value>bbb</value>
            </list>
        </property>
        <property name="ints">
            <array>
                <value>11</value>
                <value>22</value>
            </array>
        </property>
        <property name="douMap">
            <map>
                <entry key="a" value="1.2"></entry>
                <entry key="b" value="1.3"></entry>
            </map>
        </property>
    </bean>
```

### 方法4:内部bean注入

```
<property name="suser">
    <bean class="com.javasm.entity.Sysuser"></bean>
</property>
```

### 方法5:自动装配(重要)

在开启了包扫描的情况下<context:component-scan base-package="com.javasm"></context:component-scan>>

 可以通过Autowired,Resource注解进行di自动装配.

```
@Resource//先byName再byType
//@Autowired//先byType在byName
private ISysuserSerivce us;
```



## 9.xml文件的补充

```
<import resource="dao.xml"></import>
```

```
<context:property-placeholder location="jdbc.properties"></context:property-placeholder>
```



## 10.反射工厂模式

简单静态工厂:这种工厂没有可扩展性.

反射工厂:基于反射+配置的方式,提高代码的扩展性





总结:

spring容器的类图.

ioc与di的概念.

InitializingBean接口,指定初始化方法.

注册bean的方式:bean标签,工厂bean,包扫描

di的方式:set注入,构造器注入,自动装配



@Controller

@Service

@Repository

@Component

@Autowired

@Resource

@PostConstract

@PreDestroy

