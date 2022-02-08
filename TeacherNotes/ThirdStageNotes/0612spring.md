## 0612spring

## 1.通过事务切面来学习aop

事务管理器对象:MyTransactinManager,是一个管理器工具对象,用来打开,关闭,回滚链接.

事务切面:TxAspect把通知织入带有Tx注解的连接点方法.

ThreadLocal对象:线程变量对象.在多线程情况下,确保线程安全的做法.

ApplicationContextAware接口:用来获取spring容器引用的接口.需要重写setApplicationContext方法.

​							解决在静态类的静态方法中获取容器中对象的问题.

​							在子线程下怎么获取容器中的对象.

```
@Component
public class SpringUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext;//静态被所有实例共享

    @Override
    public void setApplicationContext(ApplicationContext ac) throws BeansException {
        applicationContext=ac;
    }

    public static <T> T getBean(Class<T> clz){
        return applicationContext.getBean(clz);
    }
}
```



## 2.spring的AnnotationConfigApplicationContext容器对象

BeanFactory-->DefaultListableBeanFactory

BeanFactory-->ApplicationContext-->ClassPathXMLApplicationContext

​								  AnnotationConfigApplicationContext

- ClassPathXMLApplicationContext加载xml配置文件的.xml配置的方式应用在ssm框架中.

- AnnotationConfigApplicationContext加载class类配置文件的.类配置的方式应用在springboot框架中.

类配置常用注解:

```
@Configuration  //定义配置类
@Bean           //注册bean
@PropertySource  //加载properties文件,把配置数据注册容器
@ComponentScan   //开启包扫描
@EnableAspectjAutoProxy  //开启aspectj注解识别
@Import                   //引入其他配置类
@Value                   //获取容器中properties配置数据
@ImportResource         //引入其他xml配置文件
```



## 3.写下mybatis的几张表的操作

