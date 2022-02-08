1.mybatis配置文件

​	properties:加载类路径下的properties文件,把该文件中的数据加载到Configuration对象.

​	settings:mybatis运行日志,驼峰命名映射,文件中的数据加载到Configuration对象.

​	typeAlias:别名映射,mybatis默认有内嵌别名(String-string.Integer-int.HashMap-map,List-list,arrayList-list);

​					通过指定package包路径, 对包下的的类做别名映射.

​	environments:数据库环境配置,文件中的数据加载到Configuration对象.

​	mappers:映射文件的引入,文件中的数据加载到Configuration对象中的mappedStatements成员变量,Map<String,MappedStatement>.



2.xml映射文件,id不能重复

​	select:parameterType|resultType

​	insert:parameterType|useGenerateKeys|keyProperty

​	update:parameterType

​	delete:parameterType

​	接收动态参数:#{属性名|map的key}



3.api

​	SqlSessionFactoryBuilder-->build(InputStream in)    

​	SqlSessionFactory(DefaultSqlSessionFactory)

​	Configuration配置对象,配置文件与映射文件在内存中的对象.

​	SqlSession(DefaultSqlSession)-->selectOne|selectList|insert|update|delete|getMapper.​	



4.junit

​	@FixMethodOrder固定多个测试方法的执行顺序

​	@BeforeClass注解静态方法,全局初始化方法

​	@Before注解测试初始化方法

​	@Test注解测试方法

​	@After注解测试销毁方法

​	@AfterClass注解静态方法,全局销毁方法.



# 0608mybatis

## 1.#{}与${}区别

两者都是用来获取动态参数.

#{}:mybatis会把#{}置换成?占位符.

${}:mybatis直接把获取的动态参数拼接sql语句,大多数情况下不建议使用.一般用来做排序



## 2.映射文件中的sql标签

提取出公共的sql语句块,通过include来引入sql块.

```
<sql id="basicFields">
      uid,uname,uemail,uphone,uwechat
    </sql>
    
    <sql id="allFields">
      uid,uname,uemail,uphone,uwechat,create_time,update_time,create_by
    </sql>

    <sql id="basicQuery">
        select uid,uname,uemail,uphone,uwechat,create_time,update_time,create_by from sysuser
    </sql>

    <select id="selectUsers" resultType="sysuser">
        select
         <include refid="basicFields"></include>
         from sysuser order by ${by} ${orderType}
    </select>
```



## 3.resultMap标签(重要)

resultMap:结果集映射标签,把查询结果列映射到指定类的指定成员变量.

```
<resultMap id="sysuserResultMap" type="sysuser">
        <id column="uid" property="uid"></id><!--主键列映射-->
        <result column="uname" property="uname2"></result>
        <result column="uemail" property="uemail"></result>
        <!--建议所有字段都映射上-->
    </resultMap>

    <select id="selectByKey" parameterType="int" resultMap="sysuserResultMap">
        <include refid="basicQuery"></include>
        where uid=#{uid}
    </select>
```

注意点:select标签的resultMap属性不能与resultType同时出现.

​		一个映射文件中可能有很多resultMap标签



## 4.对象关系映射-多对一映射|持有关系映射(重要)

数据库表的设计关系: 1-多|多对多

1-N:在1方关联多方主键作为外键

N-N:使用中间表来关联两个表的主键作为外键

对象设计关系:持有关系|聚合关系,    建议<<uml模型设计>>



场景:查询用户,在查询用户的同时,并查询出用户所持有的角色对象.

方法1:手工进行两次单表查询,效率最高.这个适用于查询单个数据(重要)

方法2:使用sql语句进行表链接查询,一次查询出所有记录,分别做映射.这个适用于查询列表.(重要)

```
<resultMap id="userAndRoleMap" type="sysuser">
        <id column="uid" property="uid"></id>
        <result column="uname" property="uname2"></result>
        <result column="upwd" property="upwd"></result>
        <result column="uphone" property="uphone"></result>
        <result column="uwechat" property="uwechat"></result>
        <result column="uemail" property="uemail"></result>
        <result column="create_time" property="createTime"></result>
        <result column="update_time" property="updateTime"></result>
        <result column="create_by" property="createBy"></result>
        <!--sysuser类中的srole成员变量映射-->
        <association property="srole" javaType="Sysrole">
            <id column="rid" property="rid"></id>
            <result column="rname" property="rname"></result>
            <result column="rdesc" property="rdesc"></result>
            <result column="rctime" property="createTime"></result>
            <result column="rutime" property="updateTime"></result>
        </association>
    </resultMap>

    <select id="selectUserAndRoleByUid" parameterType="int" resultMap="userAndRoleMap">
        select u.*,r.rname,r.rdesc,r.create_time as rctime,r.update_time as rutime from sysuser u left JOIN sysrole r on u.rid=r.rid where u.uid=#{uid}
    </select>
```

方法3:使用mybatis内部的二次查询操作.(了解)

```
<resultMap id="userAndRoleMap2" type="sysuser">
        <id column="uid" property="uid"></id>
        <result column="uname" property="uname2"></result>
        <!--
            column:二次查询需要的参数列
            property:sysuser类中的成员变量名
            javaType:类型
            select:二次查询的位置
        -->
        <association column="rid" property="srole" javaType="Sysrole" select="com.javasm.mapper.SysroleMapper.selectRoleByKey"></association>
    </resultMap>
```



## 5.对象关系映射-1对多映射|聚合关系映射(重要)

场景:查询角色时,把该角色下的所有用户查询出来.

方法1:手工进行多次单标查询,把查询结果组合

```
SysuserMapper um = session.getMapper(SysuserMapper.class);
        SysroleMapper rm = session.getMapper(SysroleMapper.class);
        int rid=2;
        //第一次查询:单查角色表
        Sysrole sysrole = rm.selectRoleByKey(rid);

        //第二次查询:单差用户表
        List<Sysuser> users = um.selectUsersByRoleId(rid);
        sysrole.setUsers(users);
```

方法2:使用sql表链接查询

```
 <resultMap id="roleAndUsersMap" type="sysrole">
        <id column="rid" property="rid"></id>
        <result column="rname" property="rname"></result>
        <result column="rdesc" property="rdesc"></result>
        <collection property="users" ofType="Sysuser"><!--List<Sysuser>-->
            <id column="uid" property="uid"></id>
            <result column="uname" property="uname2"></result>
            <result column="upwd" property="upwd"></result>
            <result column="uphone" property="uphone"></result>
            <result column="uwechat" property="uwechat"></result>
            <result column="uemail" property="uemail"></result>
        </collection>
    </resultMap>

    <select id="selectRoleAndUsersByKey" parameterType="int" resultMap="roleAndUsersMap">
      select r.rid,r.rname,r.rdesc,u.uid,u.uname,u.upwd,u.uphone,u.uwechat,u.uemail from sysrole r left join sysuser u on r.rid=u.rid where r.rid=#{rid}
    </select>
```



方法3:mybatis内部发起二次单表查询,不建议使用

```
   <resultMap id="roleAndUsersMap2" type="sysrole">
        <id column="rid" property="rid"></id>
        <result column="rname" property="rname"></result>
        <result column="rdesc" property="rdesc"></result>
        <collection property="users" ofType="Sysuser" column="rid" select="com.javasm.mapper.SysuserMapper.selectUsersByRoleId"></collection>
    </resultMap>

    <select id="selectRoleAndUsersByKey2" parameterType="int" resultMap="roleAndUsersMap2">
      select * from sysrole where rid=#{rid}
    </select>
```



## 6.动态sql语句(重要)

在xml映射文件中进行sql语句拼接的一种方式.

- if:条件判断,test属性写boolean表达式
- where:生成where关键字,并忽略紧跟其后的and或or
- set:用在update语句中,生成set关键字,并忽略最后的逗号
- foreach:用在批量删除,批量添加上
- choose (when, otherwise),类似于switch-case-default



模糊查询:

```
CONCAT("%",#{rname},"%")
"%"#{uname}"%"
```

比较查询:

```
小于比较,小于号存在歧义,需要使用特殊符号
小于: and rid &lt; #{rid}
小于等于:and rid &lt;= #{rid}
```



## 7.mybatis延迟加载

只存在于mybatis内存在二次查询的情况.

```
 <!--全局开启延迟加载-->
<setting name="lazyLoadingEnabled" value="true"></setting>
<!--改积极加载为消极加载-->
<setting name="aggressiveLazyLoading" value="false"></setting>
<!--禁用toString,equals,hashcode,clone方法的触发延迟加载属性-->
<setting name="lazyLoadTriggerMethods" value=""></setting>
```



重要的做项目,有延迟加载的思想.

分页,树形节点的懒加载.



## 8.mybatis缓存使用

orm框架都有缓存的实现.在web开发中用处不大.

一级缓存:session级别的缓存,默认开启

​	查询过程:先去SqlSession对象的缓存空间查询数据,查询到则直接返回;查询不到则查询数据,并把查询结果插入缓存.

二级缓存:sessionFactory级别的缓存.未开启

​	不同会话之间共享缓存空间.

- ```
  <!--全局开启二级缓存-->
  <setting name="cacheEnabled" value="true"></setting>
  ```

- ```
  在某个映射文件中配置缓存策略.
  <!--配置缓存策略:FIFO先进先出-->
      <cache eviction="FIFO"
              flushInterval="60000"
              size="512"
              readOnly="true"/>
  ```

- ```
  当以上配置完成后,该映射文件中的所有select操作,全部启用二级缓存,如果要禁用掉某个查询的缓存.使用useCache="false"禁用缓存
   <select id="selectByKey" parameterType="int" resultMap="sysuserResultMap" useCache="false">
  ```

- ```
  如果执行了update,delte操作,可以通过flushCache强制刷新缓存,查询数据时候立即更新.
  <update id="updateUser" flushCache="true"></update>
  ```


## 9.代理模式-静态代理实现

Proxy代理模式使用场景:当一个已经存在对象,某个方法不满足需求,需要扩展的时候,使用代理模式.

代理模式分为静态代理,动态代理.





总结:

resultMap标签的association与collection与id与result四个子标签;

动态sql:if|where|set|foreach

静态代理.







disruptor高并发框架



































、
