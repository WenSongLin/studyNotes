# 0607mybatis入门

### 1.mybatis有什么用?

对jdbc的轻量级的封装,属于半自动化orm框架(oop+sql).提高开发效率.



oop:面向对象编程.

orm:对象关系映射持久层框架,比如:mybatis,hibernate,jpa,mybatis-plus



### 2.50分钟入门案例

- 安装mybatis的环境,准备mybats.jar包;准备mysql驱动包;
- 数据库环境,数据库表创建,对应的实体类.
- 创建mybatis的核心配置

```

```



- 创建实体类对应的映射文件,并把映射文件加入到配置文件中

```

```



- 进行测试

```
//运行mybatis,步骤如下:
        //0.通过ClassLoader类加载 类路径(src/resources)下的文件
//        InputStream in = TestSelectUserByKey.class.getClassLoader().getResourceAsStream("mybatis-config.xml");
        InputStream in = Resources.getResourceAsStream("mybatis-config.xml");

        //1.通过mybatis的核心对象SqlSessionFactoryBuilder,来加载mybatis-config.xml进行解析到Configuration对象
        //理解成DruidDataSource,数据库连接池对象(List<Connection>)
        //DefaultSqlSessionFactory对象-configuration
        SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(in);
        //2.数据库会话对象(HttpSession),理解成Connection
        SqlSession session = ssf.openSession();
        //3.执行数据库查询操作
        Object o = session.selectOne("aa.bb.selectUserByKey", 1);
        System.out.println(o);
        //4.释放链接
        session.close();
        //入门总结:
        /**
         * SqlSessionFactoryBuilder加载mybatis-config文件,根据数据库链接信息创建连接池对象;
         * 解析sysuser-mapper.xml文件,把该文件中的select标签进行解析   aa.bb.selectUserByKey--->MappedStatement
         * 执行selectOne方法,传入字符串,找到MappedStatement对象,传入参数,进行sql执行.
         *
         */
```

### 3.mybatis的核心对象

- SqlSessionFactoryBuilder:构建者模式的应用.用来做复杂对象的构建.用来解析配置文件与映射文件,创建DefaultSqlSessionFactory(Configuration),该对象生命周期很短暂,用完即销毁.
  - build(InputStream in)

- SqlSessionFactory-->DefaultSqlSessionFactory(Configuration):会话工厂对象,全局唯一的单例对象,因为该对象内有连接池,有文件解析结果Map.

  - openSession()

- SqlSession-->DefaultSqlSession:sql会话对象,用来执行数据库操作,用完即关闭.

  - selectOne
  - selectList
  - insert
  - update
  - delete
  - getMapper******************************************************************************重要

  ```
  1.创建dao接口(mapper接口)
  2.xml映射文件的namespace指定成dao接口的全名,同时映射文件中的id值与方法名完全一致;
  3.测试getMapper方法
  ```


### 4.mybatis配置文件

- properties:引入外部的properties文件

```
    <!--加载类路径下properties文件-->
    <properties resource="jdbc.properties"></properties>
```

- **<u>settings:修改mybatis的运行行为...必用</u>**

```
<!--修改mybatis运行的一些默认行为-->
    <settings>
        <!--日志-->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
        <!--开启驼峰命名映射-->
        <setting name="mapUnderscoreToCamelCase" value="true"></setting>
    </settings>
```

- **<u>typeAlias:mybatis类型别名配置,不同包下也不能出现同名的类...必用</u>**

```
 <typeAliases>
        <!--在创建Configuration对象,对指定包下的所有的类,做别名映射,把类名小写作为别名,映射文件中的resultType忽略大小写-->
        <package name="com.javasm"></package>
        <!--<typeAlias type="com.javasm.sys.entity.SysUser" alias="sysuser"></typeAlias>-->
    </typeAliases>
```

- environments:配置数据库环境信息

```
  <!--配置数据库环境-->
    <environments default="dev">
        <!--在一个项目中,做了分库分表操作,多数据库环境,后续有专门mycat中间件做分库操作-->
        <!--代表一个数据库环境-->
        <environment id="dev">
            <!--JDBC:开启事务-->
            <transactionManager type="JDBC"></transactionManager>
            <!--UNPOOLED|POOLED|JNDI-->
            <!--PooledDataSource-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
```

- mappers:配置映射文件路径,再创建Configuration对象时,解析映射文件中的select,insert,update,delete标签到Configuration中的mappedStatements集合中,格式如下:namespace.id-->MappedStatement

```
 <mappers>
        <mapper resource="com\javasm\sys\mapper\sysuser-mapper.xml"></mapper>
 </mappers>
```



### 5.mybatis映射文件

- mapper

```
namespace="aa.bb"必须唯一
```

- select

```
<!--
parameterType:参数类型:实体类,简单类型,map;
resultType:结果类型:实体类,简单类型,map;永远不会时list,set
如果parameterType是简单类型,#{随便写}
如果parameterType是实体对象,#{属性名}
如果parameterType是map,#{key}
-->
<select id="唯一标识" parameterType="参数类型" resultType="结果类型">
```

- insert,执行insert语句,改标签没有resultType属性.

```
    <!--添加操作-->
    <!--sysuser表是自增主键,有需要要获取新增记录id-->
    <insert id="addUser" parameterType="Sysuser" useGeneratedKeys="true" keyProperty="uid">
        insert into sysuser(uname, upwd, uphone, uwechat, uemail, create_by) values
        (#{uname}, #{upwd}, #{uphone}, #{uwechat}, #{uemail},#{createBy})
    </insert>
```

- update

- delete

```
<!--mybatis底层根据标签名,决定底层执行PreparedStatement对象的不同方法,executeQuery,executeUpdate-->
<update id="updateUserByUid" parameterType="Sysuser">
  update sysuser set upwd=#{upwd},uphone=#{uphone} where uid=#{uid}
</update>

<delete id="delUser" parameterType="int">
    delete from sysuser where uid=#{uid}
</delete>
```

- 总结注意点:

  注意sql语句中使用#{};

  注意#{写法}

  注意id重复.



### 6.多参数传递

1.多个参数封装到实体类中,重要

#{写对象的属性名},切记一定要有get方法

```
 <select id="selectUsersByUnameAndPwd" parameterType="Sysuser" resultType="sysuser">
        select * from sysuser where uname=#{uname} and upwd=#{upwd}
    </select>
```



2.多个参数封装到Map对象中.重要

```
 <!--#{map的key}-->
    <select id="selectUsersByUnameAndPwd2" parameterType="map" resultType="sysuser">
        select * from sysuser where uname=#{uname_key} and upwd=#{upwd_key}
    </select>
```



3.引入mapper接口以后,可以在接口的实参加@Param注解,指定key值.

```
 List<Sysuser> login(@Param("uname2") String uname, @Param("upwd2") String upwd);
 
    <select id="login" parameterType="map" resultType="Sysuser">
        select * from sysuser where uname=#{uname2} and upwd=#{upwd2}
    </select>
     <select id="login" parameterType="map" resultType="Sysuser">
        select * from sysuser where uname=#{0} and upwd=#{1}
    </select>
     <select id="login" parameterType="map" resultType="Sysuser">
        select * from sysuser where uname=#{param1} and upwd=#{param2}
    </select>
```



### 8.常见异常

看最下方的Caused by: 查看错误

1.别名重复,包下有同名的类

```
org.apache.ibatis.type.TypeException: The alias 'SysUser' is already mapped to the value 'com.javasm.utils.Sysuser'.
```

2.反射异常,#{写错}

```
ReflectionException: There is no getter for property named 'aa' in 'class com.javasm.sys.entity.Sysuser'
```

3.映射文件中id重复

```
java.lang.IllegalArgumentException: Mapped Statements collection already contains value for aa.bb.selectUsers2
```

4.绑定异常,#{写错}

```
org.apache.ibatis.binding.BindingException: Parameter 'a' not found. Available parameters are [0, 1, param1, param2]
```

5.绑定异常,MappedStatement不存在,(dao接口中的方法在映射文件中没有标签)

```
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found): com.javasm.sys.mapper2.SysuserMapper.del
```

6.绑定异常,接口未注册(映射文件中的namespace没有对应接口的名称),可能没引入映射文件

```
org.apache.ibatis.binding.BindingException: Type interface com.javasm.sys.mapper2.SysuserMapper is not known to the MapperRegistry.
```



注意:

关于日期:

数据库字段:date.,datetime(now()),timestamp(CURRENT_TIMESTAMP)

实体类:String

a_name..aName..setaName





总结:

三个核心对象的api.尤其是SqlSession中的getMapper方法.

配置文件中的settings,typeAlias必须会.

映射文件中全部必须会.

