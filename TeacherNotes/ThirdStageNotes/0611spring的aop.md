@Controller,@Service,@Repository,@Component,@Resource,@Autowired

@PostConstract,@PreDestroy

InitializingBean

bean标签的用法:bean指定class属性进行注册容器;property子标签进行set注入,constractor-arg进行构造器注入.



set注入的另类写法:

```
<bean id="sysuserController" class="com.javasm.controller.SysuserController"-->
          <!--p:us-ref="sysuserService"></bean>
        <bean id="sysuserService" class="com.javasm.service.impl.SysuserService"></bean>
```



# 0611spring的aop

## 1.junit与spring整合使用

把测试类也注册spring容器,然后需要测哪个对象,注入哪个对象.

- 添加spring-test整合包
- 使用runwith注解指定springJunit4ClassRunner类,加载xml初始化容器

```
//加载xml初始化容器,并把当前测试类注册容器
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring.xml")// classpath:绝对路径的写法,表示从根路径开始查找spring.xml
```



## 2.什么是aop

aop:面向切面对象编程,是oop面向对象编程的一种补充完善.

通过oop面向对象编程实现了核心的业务逻辑.对于后期需要添加的辅助性的业务需求(日志,参数校验)使用aop的思想来进行扩展.辅助性的业务需求称为切面.

aop实现方式,在原核心对象的基础上,通过代理模式,创建出代理对象对原对象进行包装,执行的时候,执行的是代理对象的方法,需要扩展的切面对象,可以在代理对象的方法中进行统一扩展.



- aop编程中的几个概念:

切面aspect:即辅助业务对象(日志,效率监测,参数校验).是一个切面类

通知advice:即切面类中的方法.分为5类通知(前置通知,返回通知,最终通知,异常通知.环绕通知.)



织入weave:在程序运行期间,通过动态代理模式,给目标对象创建代理对象,并把切面中的通知方法   织入   到指定的连接点方法的前,后,异常,最终处.



目标对象target:即核心业务对象(需要扩展的业务对象)(被代理的对象)

连接点joinPoint:即核心业务方法,目标对象中的某个方法

切入点pointcut:连接点的集合,通过一个切入点表达式可以灵活的定义多个连接点



## 3.通过动态代理实现aop

```
public static void main(String[] args) {
        //被代理对象,该对象的pay方法需要扩展,添加日志,效率监测,参数校验.
        IPay p = new AliPay();
        IPay p2 = createProxy(p);//$Proxy100
        p2.pay("aa","bb",11.1);//p.pay()
    }

    public static IPay createProxy(IPay p){
        //p:目标对象
        ClassLoader loader= p.getClass().getClassLoader();
        Class[] interfaces = new Class[]{IPay.class};
        InvocationHandler handler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //proxy:代理对象
                String name = method.getName();
                if("pay".equals(name)){
                    Object obj = null;
                    //切面
                    MyAspect a = new MyAspect();
                    try{
                        a.beforeAdvice();//通知
                        obj = method.invoke(p,args);//执行连接点方法
                        a.afterReturnAdvice();
                    }catch (Exception e){
                        a.exceptionAdvice();
                    }finally {
                        a.afterAdvice();
                    }
                    return obj;
                }

                return method.invoke(p,args);//p.toString()
            }
        };

        // Object obj = new $Proxy100(handler);
        //IPay proxy = (IPay)obj;
       IPay proxy = (IPay) Proxy.newProxyInstance(loader,interfaces,handler);
        return proxy;
    }
```



## 4.spring中的aop注解使用

- 添加spring-aop;spring-aspect两个依赖包;spring的aop依赖的aspectj第三方组件
- 添加aspectj组件的三个依赖包;
- 添加cglib一个依赖包;是一个动态代理的包,类似于Proxy类,cglib不要求被代理有接口
- 在spring的xml配置文件中,开启aop注解识别.

```
 <!--aspectj组件的注解识别,@Aspect,Pointcut,Before,AfterReturning,AfterThrowing,After,Around-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

- 创建切面类,创建通知方法,

```
@Component//注册容器
@Aspect//定义该bean时切面对象
public class LogAspect {

    //切入点表达式两种写法:
    //方法1:execution(返回值类型  包名.类名.方法名(形参类型))-----包名.类名可以省略
    @Pointcut("execution(* com.javasm.service.impl.SysuserService.*(..))")
    public void pc(){}

    /**
     * 环绕通知
     * 返回值类型必须是Object
     * 形参必须是ProceedingJoinPoint
     * @return
     */
    @Around("pc()")
    public Object aroundAdvice(ProceedingJoinPoint jp){
        //jp对象能够得到所有连接点信息
        Object result = null;
        try {
            System.out.println("前置通知"+jp.getSignature().getName());
            result = jp.proceed();//Object result = method.invoke(target,args)
            System.out.println("返回通知"+result);
        } catch (Throwable throwable) {
            System.out.println("异常通知"+throwable.getMessage());
        }finally {
            System.out.println("最终通知");
        }
        return result;
    }
    
}
```





## 5.切入点表达式:

```
//切入点表达式两种写法:
//方法1:execution(返回值类型  包名.类名.方法名(形参类型))-----包名.类名可以省略
// 方法2:自定义注解,@annotation(注解类名),表示带有指定注解的方法全部时连接点方法
//@Pointcut("execution(* com.javasm.service.impl.*.*(..))")
@Pointcut("@annotation(com.javasm.annotation.tx)")
public void pc(){}
```



## 6.spring中的aop配置使用

```
<bean id="logAspect2" class="com.javasm.aspect.LogAspect2"></bean>

    <aop:config>
        <aop:aspect ref="logAspect2">
            <aop:pointcut id="pc" expression="@annotation(com.javasm.annotation.tx)"></aop:pointcut>
            <aop:around method="aroundAdvice" pointcut-ref="pc"></aop:around>
            <!--<aop:before method="beforeAdvice" pointcut-ref="pc"></aop:before>-->
            <!--<aop:after-returning method="afterReturnAdvice" pointcut-ref="pc" returning="o"></aop:after-returning>-->
            <!--<aop:after-throwing method="exceptionAdvice" pointcut-ref="pc" throwing="e"></aop:after-throwing>-->
            <!--<aop:after method="afterAdvice" pointcut-ref="pc"></aop:after>-->
        </aop:aspect>
    </aop:config>
```



## 7.dom4j解析xml文件

Properties类解析.properties文件

Dom4j组件解析xml文件,pull组件解析xml文件.

中间件开发,必须需要进行xml解析



- 添加dom4j.jar
- Document doc = SAXReader.read(InputStream)
- Element root = doc.getRootElement()
- String str  =root.attributeValue("属性名")
- List<Element> tags = root.elements("标签名")



## 8.easycode插件

- 安装插件
- 配置Type Mapper
- 配置Tmeplate settings











