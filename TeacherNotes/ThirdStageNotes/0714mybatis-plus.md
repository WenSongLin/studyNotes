# 0714mybatis-plus

## 1.作用

为简化mybatis开发而生,该框架对mybatis只做增强不做改变，原来的mybatis代码完全不受影响,引入它不会对现有工程产生影响;

该框架简化的单表的curd操作.

该框架内部集成了分页插件.

mybatisx:提供了代码生成,提供了dao接口方法跳到xml文件中.



## 2.安装mp环境

mp基于springboot环境搭建,添加myatis-plus-boot-starter启动器,该启动不能与mybatis-spring-boot-starter重复引入.

```
       <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
          <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.2</version>
        </dependency>
```



### 特性:

自动开启驼峰命名映射;

单表的curd的sql语句,在运行时根据实体类自动生成.类名做表名;属性名做字段名,会开启驼峰映射;

插入数据时,insert语句,mp对主键列按照默认的id策略生成唯一id.







## 2.常用注解

- @TableId 
  - 作用: 指定主键列
  - 常用属性:
    - value: 当类中的属性名与数据库列表不一致时,指定数据库列名;
    -  type:指定主键的生成策略( IdType.AUTO|INPUT|ASSIGN_ID)
      - 建议,数据库主键varchar,实体类String
- @TableName
  - 作用:指定表名
  - 常用属性:value
- @TableField
  - 作用:注解非主键列
  - 属性:
    - value: 指定字段名
    - exist:对于实体类中的辅助属性,在数据中不存在对应的字段,通过exist=false.(最常用)
    - condition:指定查询条件,配合QueryWrapper查询条件构造器对象使用.指定条件like,eq,notlike,noteq.

- @Version

  - 作用:乐观锁注解,

    - 用法:数据库中加int型的version字段;实体类加Integer型的version属性,使用该注解;

      ​	在容器中注入OptimisticLockerInnerInterceptor


## 3.条件构造器对象

查询条件构建:

```
QueryWrapper w = new QueryWrapper();//查询,删除
//eq  .like .likeleft.  likeright.  isnull.  isnotnull.    in .notin   .between
dao.selectList(w);
```



```
Users u = new Users();
u.setUname("admin");
u.setUphone("1111");
QueryWrapper w  = new QueryWrapper(u);
//不按照等值查询.通过TableField的condition属性来指定条件表达式.
// @TableField(condition = "%s like concat('%%',#{%s},'%%')")
//    @TableField(condition = SqlCondition.LIKE)
//    private String uphone;
List<Users> list = ud.selectList(w);//
System.out.println(list);
```



## 4.乐观锁插件

在多人同时操作数据库中同一条记录.

```
@Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
```



## 5.分页插件

推荐使用PgaeHelper分页插件:

```
 <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.3</version>
            <exclusions>
                <exclusion>
                    <groupId>org.mybatis</groupId>
                    <artifactId>mybatis</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.mybatis</groupId>
                    <artifactId>mybatis-spring</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```



```
PageHelper.startPage(1,2);
List<Users> users = um.selectList(null);
System.out.println(new PageInfo(users));
```





## 6.代码生成

mybatisplus内有AutoGenerator,不建议使用

去搜索一个easycode的myplus模板.或者使用mybatisX插件进行代码生成

entity:

```
##引入宏定义
$!define

##使用宏定义设置回调（保存位置与文件后缀）
#save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity")

##使用全局变量实现默认包导入
$!autoImport
import java.io.Serializable;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;

##使用宏定义实现类注释信息
#tableComment("实体类")
@JsonInclude(JsonInclude.Include.NON_NULL)
@TableName("$tableInfo.name")
public class $!{tableInfo.name} implements Serializable {
    private static final long serialVersionUID = $!tool.serial();
#foreach($column in $tableInfo.fullColumn)
    #if(${column.comment})/**
    * ${column.comment}
    */#end
    
    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
#end

#foreach($column in $tableInfo.fullColumn)
##使用宏定义实现get,set方法
#getSetMethod($column)
#end

}
```



dao:

```
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Dao"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/dao"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}dao;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import org.apache.ibatis.annotations.Param;
import java.util.List;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;

public interface $!{tableName} extends BaseMapper<$!{tableInfo.name}> {

}

```



service:

```
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Service"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import java.util.List;
import com.baomidou.mybatisplus.extension.service.IService;

public interface $!{tableName} extends IService<$!{tableInfo.name}> {

}
```



serviceImpl:

```
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "ServiceImpl"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service/impl"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service.impl;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.dao.$!{tableInfo.name}Dao;
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;

@Service("$!tool.firstLowerCase($!{tableInfo.name})Service")
public class $!{tableName}  extends ServiceImpl<$!{tableInfo.name}Dao,$!{tableInfo.name}> implements $!{tableInfo.name}Service {
    
}
```



handler:

```
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Handler"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/handler"))
##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}handler;

import $!{tableInfo.savePackageName}.entity.$!{tableInfo.name};
import $!{tableInfo.savePackageName}.service.$!{tableInfo.name}Service;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import com.javasm.commons.base.BaseHandler;
import com.javasm.commons.entity.AxiosResult;
import java.util.Arrays;
import java.util.List;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;

@RestController
@RequestMapping("$!tool.firstLowerCase($tableInfo.name)")
public class $!{tableName} extends BaseHandler{
   
    @Resource
    private $!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service;

    @GetMapping("{id}")
    public AxiosResult selectById(@PathVariable("id") String id){
        $!{tableInfo.name} obj = this.$!{tool.firstLowerCase($tableInfo.name)}Service.getById(id);
        return suc(obj);
    }
    
   @GetMapping("list")
    public AxiosResult selectList($!{tableInfo.name} obj){
        startPage();
        List<$!{tableInfo.name}> list = this.$!{tool.firstLowerCase($tableInfo.name)}Service.list(new QueryWrapper<>(obj));
        return toTableDatas(list);
    }
    
    @PostMapping
    public AxiosResult add(@RequestBody $!{tableInfo.name} obj){
        boolean r = this.$!{tool.firstLowerCase($tableInfo.name)}Service.save(obj);
        return result(r);
    }
    
    @PutMapping
    public AxiosResult update(@RequestBody $!{tableInfo.name} obj){
        boolean r = this.$!{tool.firstLowerCase($tableInfo.name)}Service.updateById(obj);
        return result(r);
    }
    
     @DeleteMapping("{ids}")
    public AxiosResult delById(@PathVariable("ids")String ids){
        String[] split = ids.split(",");
        boolean r = this.$!{tool.firstLowerCase($tableInfo.name)}Service.removeByIds(Arrays.asList(split));
        return result(r);
    }
}
```



mapper:

```
##引入mybatis支持
$!mybatisSupport
##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Dao.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.dao.$!{tableInfo.name}Dao">

 </mapper>
```



