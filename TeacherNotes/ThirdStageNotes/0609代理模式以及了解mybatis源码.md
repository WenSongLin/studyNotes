mybatis整体重点:

​	配置文件中的settings标签,修改mybatis运行行为.日志,驼峰映射.

​	映射文件中的resultMap(id,result,association,collection)

​	映射文件中的select,insert,update,delete(parameterType="实体类,map,简单类型")

​	映射文件中的#{}

​	映射文件中的if,where,set,foreach(collection,open,close,item,sperator)

​	

## 0609代理模式以及了解mybatis源码

## 1.代理模式

使用场景:在不改动源码的情况下,对一个已经存在的对象的方法进行扩展.日常做项目用不上,做中间件(框架,组件)必定使用代理模式+反射+观察者模式.



- 参与角色:被代理对象(原生已经存在的对象),代理对象(新创建的对象),调用者(调用代理对象)

- 方法1:静态代理,缺点:需要手工派生代理类,重写所有方法,只对需要扩展的方法做特殊处理.

- 方法2:动态代理,不需要手工创建代理类,在程序运行期间,由Proxy类在jvm动态代理类,并实例化代理对象.

  - 动态代理实现方式:Proxy,cglib组件

  - Proxy工具类要求被代理对象必须有接口.因为$Prxoy0代理类已经使用了extends关键字;

  - $Proxy0类中持有的是InvocationHandler对象.


Proxy工具类生成的代理类结构:

```
public class $Proxy0 extends Proxy implements Connection{

  protected InvocationHandler h;
  private Method m38;
  
  static{
      m38=Class.forName("java.sql.Connection").getMethod("prepareStatement", new Class[] { Class.forName("java.lang.String") });
  }
  
  public $Proxy0(InvocationHandler paramInvocationHandler)
  {
    this.h=h;
  }
  
  public final PreparedStatement prepareStatement(String sql) throws Exception
  {
      return (PreparedStatement)this.h.invoke(this, m38, new Object[]{sql});
  }   
}
```



## 2.Configuration对象的构建过程

了解配置文件与映射文件的加载过程.

```
new Configuration(){
    Properties valiableus;//放的是settings配置
    Map<String,Class> typeAlias;//放的是别名配置
    Map<String,MappedStatent> mappedStataments;//放的是映射文件的select|insert|update|delte
    Map<String,ResultMap> resultMaps;//放的是映射文件中的resultMap标签的解析
}

new XMLConfigBuilder(InputStream in);//解析配置文件,new Configuration()
Configuration config = XMLConfigBuilder.parse();//向Configuration对象填充数据
SqlSessionFactory factory = new DefaultSqlSessionFactory(config);//共产模式的应用


new XMLMapperBuilder(InputStream in,config);//解析映射文件
XMLMapperBuilder.parse()


new XMLStatementBuilder(config,"namespace","select标签")
XMLStatementBuilder.parseStatementNode()
```



构建者模式:构建复杂对象的对象.该对象的职责就是用来构建另外一个单例对象.

XXXBuilder{

​	new duixiang()

​	build(){}

​	parse(){}	

}



工厂模式:构建复杂对象,该对象的职责时用来构建一系列的对象.

XXXFactory(){

​	对象 create(){}

​	对象 parse(){}

}



//不把对象的new的过程,散乱在代码不同位置.而统一放在工厂类或构建器类中来创建对象.



## 3.getMapper方法的执行原理

getMapper方法,返回的是接口的代理对象(接口的实现类实例化对象$Proxy8)

```
//回调处理器对象,该对象内的invoke方法会在代理对象的方法执行时被调用.
class MapperProxy implements InvocationHandler{
    
    public Object invoke(Object proxy,Method method,Object[] args){
        
    }
}
```



```
public T newInstance(SqlSession sqlSession) {
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        //创建$Proxy1代理类,并实例化,代理对象
      return  Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
}
```



总结:

认识构建器模式,工厂模式;

认识Configuration对象;

认识MappedStatement对象;

认识代理模式.了解getMapper方法内返回的到底是什么对象







<敏捷软件开发 原则,模式,实践>

<UML模型设计>

<微服务架构>

<spring响应式编程>

<java高并发编程>