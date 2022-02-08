# 0621.redis缓存数据库(重要)

## 1.nosql非关系型数据库

mysql:关系型数据库(数据安全性,可靠性,一致性),对于超大数据量的查询效率低.分库分表中间件.

针对与高并发下的快速响应,需要引入nosql非关系型数据库(基于内存),响应速度很快.



非关系数据库针对不同的应用场景有不同的产品解决方案.

memcached:kv内存数据库,这套数据库产品只能够存储string字符串.

redis:kv内存数据库,用来做缓存,提高响应速度.

hbase:列式数据库,适合超大数据量的存储.做大数据开发

mongodb:文档数据库,(json集合).适合字段需要随意扩展,适合数据分析应用,金融行业,爬虫行业.

mongodb和mysql非常像,mysql能做到,mongodb都能做到.



redis特性:

- 每秒钟查询支持11万次.
- redis自身基于内存来操作,为了提高数据的安全性,做了数据持久化工作.
- redis是C语言设计,单线程模型,因此所有的操作指令都是原子性,不存在多线程并发数据安全问题.
- redis有丰富的数据类型,key:String,value:5种数据类型(String,hash,list,set,sortedSet)



## 2.redis缓存数据库的使用场景

临时数据:验证码数据,用户登录会话session对象的共享.redis可以设置数据的有效时间.

高频热点数据:用户信息,排名前10的主播.redis基于内存操作,数据查询效率高.并发优秀.

常量数据:不改变的数据,字典项.

计数器:数据库主键生成.秒杀商品的计数.redis 的原子性特性.



## 3.redis带来的问题

1.mysql与redis的一致性问题.

​	解决:同步修改mysql与redis,数据强一致.

​		修改mysql,把redis中的数据对应数据删除.(使用频繁)

​		在向redis中保存数据时,设置数据的时效性(随机时间).在有效期过后自动加载新数据到redis(使用频繁).



2.由于使用redis,降低mysql访问压力.mysql数据库服务器的配置降低.

​	由于数据在同一时刻大量失效,用户的高并发访问失效数据,同时进入mysql查询数据,缓存被击穿,造成mysql的雪崩.

​	在设置数据有效期时,设置浮动时间失效.



## 3.在windows下安装redis

redis.windows.conf

```
port 6379       redis服务器端口
databases 16    redis服务器默认数据库个数.名字:0-15
save 900 1      redis数据持久化rdb策略
save 300 10
save 60 10000
dbfilename dump.rdb    rdb持久化文件名称
dir ./                 文件存储目录.
requirepass root       服务器链接密码,可以不配置.
appendonly no          是否开启aof持久化

appendfilename "appendonly6379.aof"      aof持久化文件名
auto-aof-rewrite-percentage 100          aof持久化策略
auto-aof-rewrite-min-size 64mb           

集群方案的配置暂时不讲.
```



redis-service.exe

```
进入redis安装程序的cmd窗口:启动redis服务器
redis-service.exe redis.windows.conf
```



redis-cli.exe

```
客户端链接服务器:
redis-cli.exe -h 127.0.0.1 -p 6379 -a root
```



redis-check-aof.exe

```
用来修复格式错误的aof持久化文件.
redis-check-aof.exe --fix aof文件名
```



redis-check-dump.exe

```
用来修复格式错误的rdb持久化文件.
redis-check-dump.exe --fix rdb文件名
```



把redis安装后台服务运行.

```
//安装命令
redis-server.exe --service-install redis.windows.conf

//卸载命令
redis-server.exe --service-uninstall
```



安装可视化客户端程序:redis-desktop-manager



## 4.redis持久化

- rdb持久化.存的是数据

  ​	设定在指定时间内,发生多少数据改变,会触发redis持久化磁盘.

- aof持久化.存的是指令

  ​	每1秒把当前时间发生的操作指令保存到aof文件.当文件达到规定大小后,对aof进行压缩重写.



## 5.学习redis的操作指令(最重要.)

针对redis的value可以有5种类型:

- String(最经常使用):计数器,临时数据.

```
set,get,incr,incrby,decr,decrby,getset,setnx(实现分布式锁),setex
```

- hash(常用)(能够按照field获取value)(天气数据,系统用户集合),不能够按照索引取值.

```
hset,hget,hexists,hdel,hgetall,hkeys,hlen,hvals,hincrby,hmset,hmget
```

- list(常用),存储集合数据,可重复,有序(郑州的每天的天气数据,红包列表),遍历列表

```
lpush,lpop,lrange,lindex,llen
```

- set(不常用),存储集合数据,数据不可重复,切无序.适合做集合交并差运算.

```
sadd,smembers,sdiff,sinter,sunion
```

- sortedSet,做数据排名(主播粉丝数排名;阅卷成绩排名)(不常用)

```
zadd,
zcount:zcount key minScore maxSocre返回成绩区间内的成员数量
zincrby
zrange:zrange key minIndex maxIndex 返回成绩升序排列的排名区间内的成员
zrevrange:zrevrange key minIndex maxIndex 返回成绩奖序排列的排名区间内的成员
zrangebyScore:zrangeByScore key minScore maxScore 返回成绩升序排列的成绩区间内的成员
zrevrangeByScore,zrevrangeByScore key maxScore minScore  返回成绩将序排列的成绩区间内的成员
zrank:zrank key member, 返回成绩升序排列后的某个成员的排名
zrevrank:zrevrank key member,返回成绩降序排列后的某个成员的排名
zscore:zscore key member,返回某个成员的成绩值
```



key的相关操作指令:

```
keys,expire,ttl,exists,del,type 
```



## 6.redis与spring整合

jedis或lettuce客户端api组件包.

- 代码中引入jedis.jar,连接池依赖包.Commons-pool2.jar

代码案例:

```
单机使用
 Jedis j = new Jedis("127.0.0.1",6379);
j.auth("root");
String key="goods:";//组
for(int i=0;i<1000;i++){
     j.set(key+i,"v"+i);
}
j.close();
```



整合spring

redis.properties配置文件

```
jedis.minIdle=5
jedis.host=127.0.0.1
jedis.port=6379
jedis.timeout=3000
jedis.pwd=root
```

redis.xml配置文件

```
 <!--jedis连接池对象-->
    <context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true"></context:property-placeholder>

    <bean class="redis.clients.jedis.JedisPoolConfig" id="jedisPoolConfig">
        <property name="minIdle" value="${jedis.minIdle}"></property>
    </bean>

    <bean class="redis.clients.jedis.JedisPool">
        <constructor-arg index="0" ref="jedisPoolConfig"></constructor-arg>
        <constructor-arg index="1" value="${jedis.host}"></constructor-arg>
        <constructor-arg index="2" value="${jedis.port}"></constructor-arg>
        <constructor-arg index="3" value="${jedis.timeout}"></constructor-arg>
        <constructor-arg index="4" value="${jedis.pwd}"></constructor-arg>
    </bean>
```

切记加入ignore-unresolvable="true"属性;

切记把JedisPool对象注册到spring父容器中.



总结:

记着redis的5个类型.

记着以下三个类型的使用指令.String,hash,list













