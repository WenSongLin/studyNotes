1.redis在项目中常用场景:对热点数据数据缓存(有效期).对常量数据的缓存(永久).临时数据的存储(有效期).数据共享(spring-session).分布式锁.计数器

2.redis的几种数据类型:string(set,get,incr,decr,getset,setex,setnx),

​					hash(hset,hget,hkeys,hvals,hgetall,hexists,hdel,hlen),适合根据指定field获取数据.遍历

​					list(lpush,lpop,lrange,llen,lindex),适合做遍历,app接口列表数据的分页

​					set(sadd,smembers,sdiff,sunion,sinter)

​					zset(zadd,zrank,zscore,zrange,zrangeByScore,zcount)



3.redis的代码使用:jedis客户端

​	JedisPoolConfig+JedisPool对象注册到spring容器中.



# 0622 maven项目管理工具(重要)

## 1.maven的作用

- vue脚手架与maven作用是一样.分属于前后端的项目管理工具.

- maven进行服务端java项目管理,(创建,依赖,编译,打包,部署)



## 2.maven中几个概念

maven中央仓库(中央服务器):管理了所有maven资源(项目骨架,maven插件,jar包依赖)

maven企业私服:通过nexus搭建企业maven服务器,与maven中央仓库进行数据同步.

maven客户端:需要下载安装到本地开发机,客户端可以配置maven私服镜像地址;可以配置本地仓库路径

maven资源坐标:groupid:artifactid:version通过三个维度来确定一个资源的命名.



## 3.安装maven客户端

- 解压maven客户端到本地硬盘,(不要中文路径)
- 配置本地仓库,配置镜像地址

```
<localRepository>D:\apache-maven-3.6.0\repository</localRepository>
```

```
	<mirror>
		<id>nexus-javasm</id>
		<mirrorOf>central</mirrorOf>
		<name>Nexus javasm</name>
		<url>http://192.168.20.252:8081/repository/maven-public/</url>
	</mirror>
```



## 4.idea关联maven本地客户端

file-settings-maven:关联本地mavne客户端,关联到本地settings.xml;(只对当前项目生效)

file-otherSettings-maven:(对所有项目生效的mavne配置)



## 5.创建maven项目并了解maven项目的结构

java工程骨架:org.apache.maven.archetypes:maven-archetype-quickstart:

web工程骨架:org.apache.maven.archetypes:maven-archetype-webapp

目录结构:

src/main/java:源码目录

src/main/resources:资源配置目录

src/main/webapp:web资源目录

src/test/java:junit测试源码目录

pom.xml:maven工程核心配置文件



## 6.修改maven骨架错误,并自定义骨架

- 创建一个无错的项目结构.确保没有空文件夹.
- 在pom文件中配置maven自定义骨架的插件.

```
<plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-archetype-plugin</artifactId>
        <version>3.0.0</version>
      </plugin>
```

- 执行archetype:create-from-project,调用插件创建骨架工程,该骨架在target->archetypes
- 进入target-archetype目录下执行install命令,把骨架工程安装到本地仓库.
- 使用archetype:crawl命令,扫描本地仓库下所有的骨架.在本地仓库目录下生成骨架索引文件,在该文件找到自定义的骨架信息
- 在创建项目的时候,add archetype按钮,进行骨架安装即可.



按照以上步骤,进行java工程与web工程骨架自定义.



## 7.熟悉pom.xml文件

```
1.当前项目坐标信息<groupid><artifactid><version>
2.当前项目打包方式<packaging> jar|war|pom
3.当前项目插件配置<properties> maven-resources-plugin配置编码,maven-compiler-plugin的jdk版本
4.当前项目依赖jar包配置 <dependencies> 配置依赖jar包,类似lib文件夹
5.当前项目插件配置 <build>
```



细节点:

- 1.排除自动引入的依赖包.

```
 <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.3.8</version>
      <exclusions>
        <exclusion>
          <groupId>org.springframework</groupId>
          <artifactId>spring-jcl</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
```

- 2.dependency标签下的scope定义依赖的作用域

```
scope:compile,provided,test,默认不写,表示是compile
compile:表示当前依赖参与打包,当执行package命令时,从本地仓库把jar包copy到target目录下;
provided:不参与package命令,不影响代码编译,一般来说都是servlet-api.jar
test:不参与打包,只对test命令有效.
```

- 3.在propeties标签下统一管理依赖版本

```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <!--统一依赖项的版本-->
    <spring.version>5.3.8</spring.version>
  </properties>
```

- 4.maven-resources-plugin插件配置

该插件默认只对resources目录下的文件进行copy,输出到target目录

```
<!--对maven-resources-plugin插件对指定目录下的哪些文件做copy-->
    <resources>
      <resource>
        <directory>src/main/java</directory>
        <includes>
          <include>**/*.xml</include>
        </includes>
      </resource>
      <resource>
        <directory>src/main/resources</directory>
        <includes>
          <include>**/*</include>
        </includes>
      </resource>
    </resources>
```



## 8.熟悉maven常用命令

- clean:清理target目录(常用)
- compile编译命令(常用)
- test:执行src/test目录下的测试类的测试方法,一般不用test
- package:打包命令(常用)
- install:安装jar包到本地仓库,供其他项目依赖
- deploy:安装jar包到远程mavne仓库,供其他开发人员依赖.



## 9.ssm框架搭建maven版本

pom.xml

```
 <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <spring.version>5.3.8</spring.version>
        <commonslogging.version>1.2</commonslogging.version>
        <aspectj.version>1.9.6</aspectj.version>
        <aopalliance.version>1.0</aopalliance.version>
        <cglib.version>3.3.0</cglib.version>
        <jackson.version>2.12.3</jackson.version>
        <fastjson.version>1.2.75</fastjson.version>
        <fileupload.verion>1.3.3</fileupload.verion>
        <beanutils.version>1.9.3</beanutils.version>
        <collections.version>3.2.1</collections.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.23</druid.version>
        <mybatis.version>3.5.6</mybatis.version>
        <mybatis.spring.version>2.0.6</mybatis.spring.version>
        <pageHelper.version>5.2.0</pageHelper.version>
        <jsqlparser.version>3.2</jsqlparser.version>
        <log4j.version>2.12.1</log4j.version>
        <slf4j.version>1.7.25</slf4j.version>
        <disruptor.version>3.4.2</disruptor.version>
        <commons.pool2.version>2.6.2</commons.pool2.version>
        <jedis.version>3.3.0</jedis.version>
    </properties>

    <dependencies>
        <!--spring-core-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--common-loging-->
        <dependency>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
            <version>${commonslogging.version}</version>
        </dependency>

        <!--aspectj-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
        <dependency>
            <groupId>aopalliance</groupId>
            <artifactId>aopalliance</artifactId>
            <version>${aopalliance.version}</version>
        </dependency>
        <!--cglib-->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>${cglib.version}</version>
        </dependency>
        <!--spring-aop-->
        <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>${spring.version}</version>
    </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--springmvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--jackson-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>${jackson.version}</version>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${fastjson.version}</version>
        </dependency>
        <!--commons-fileupload;commons-io-->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>${fileupload.verion}</version>
        </dependency>
        <!--commons-beanUtils-->
        <dependency>
            <groupId>commons-beanutils</groupId>
            <artifactId>commons-beanutils</artifactId>
            <version>${beanutils.version}</version>
        </dependency>
        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>${collections.version}</version>
        </dependency>

        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>

        <!--Druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>

        <!--mybatis,mybatis-spring-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis.spring.version}</version>
        </dependency>

        <!--pageHelper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>${pageHelper.version}</version>
        </dependency>
        <dependency>
            <groupId>com.github.jsqlparser</groupId>
            <artifactId>jsqlparser</artifactId>
            <version>${jsqlparser.version}</version>
        </dependency>
        <!--spring-jdbc;spring-tx-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--log4j2-->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-web</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <!--slf4j,log4j-slf4j-impl-->
        <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>${log4j.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <!--disruptor-->
        <dependency>
            <groupId>com.lmax</groupId>
            <artifactId>disruptor</artifactId>
            <version>${disruptor.version}</version>
        </dependency>
        <!--jedis,commons-pool2-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>${jedis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>${commons.pool2.version}</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <!--servelt-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
```



## 10.运行maven工程

- 部署本地tomcat启动项目;

- 使用maven的内嵌服务器插件,一般开发中使用,把tomcat或jetty作为一个jar包的方式引入项目依赖;

  - tomcat的内嵌服务器插件,版本过低;
  - jetty-maven-plugin内嵌服务器插件

  ```
  <plugin>
          <groupId>org.eclipse.jetty</groupId>
          <artifactId>jetty-maven-plugin</artifactId>
          <version>9.4.40.v20210413</version>
            <configuration>
                <httpConnector>
                  <port>8080</port>
                </httpConnector>
              <webApp>
                <contextPath>/</contextPath>
              </webApp>
            </configuration>
        </plugin>
  ```


## 12.maven工程继承

- 定义父工程,<packaging>pom,只保留该工程下的pom.xml
- pom.xml

```
<!--父工程管理依赖项,子工程不直接继承,需要手工引入,但不用指定版本,确保多个子module之间依赖版本一致-->
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>5.3.8</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
```



```
<!--放在dependencies下的依赖,被所有子module自动继承-->
  <dependencies>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>fastjson</artifactId>
      <version>1.2.75</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13</version>
    </dependency>
  </dependencies>
```



## 13.maven子模块聚合工程的搭建

云课堂应用:对于中小型应用来说,在各个平台下开发应用,如何提高代码的复用性

后台系统(课程审核,课程分类管理,讲师用户管理,注册用户管理,后台管理用户,评论管理.用户积分管理).(服务端开发)

接口系统(用户注册,用户登录,课程发布,查看课程分类,发表评论.查看收藏课程,删除收藏,付费,发布课程,删除课程)

门户网站应用(前端开发)

android移动app(android开发)



引入maven聚合工程:

按照mvc-dao纵向对单个工程进行拆分成一个个子module,达到代码复用的目的.



## 14.jjwt组件

jjwt目的:为了对明文的token进行加密,得到有效防伪的token.同时还能解密校验token.

token的应用:登录校验,接口安全,有时效的url(给接口指定可用时间)



第一步:学习jjwt组件

- 添加jjwt环境

```
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.2</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.2</version>
    <scope>runtime</scope>
</dependency>
```

- 

第二步:token在登录以及登录校验中使用



































