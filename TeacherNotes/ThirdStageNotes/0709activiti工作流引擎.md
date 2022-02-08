# 0709activiti工作流引擎

## 1.什么是工作流引擎

workflow:在项目中涉及到多人参与的流转型申请单任务.

工作流引擎:第三方的中间件,专处理workflow代码.

常用的引擎:jbpm(xml配置),acitiviti(可视化流程设计).一般每个公司都会有自己的流程引擎.

bpmn:流程设计语言

## 2.activiti环境准备

- 流程图设计插件

  官方提供的eclipse开发工具下的设计插件.

- 数据库

  运行database/create/mysql.sql三个sql脚本文件,创建数据库表结构.

- 代码依赖

```
<dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-engine</artifactId>
            <version>${activiti.version}</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti-spring</artifactId>
            <version>${activiti.version}</version>
        </dependency>
```



## 3.注册核心服务对象

```
<bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
    <property name="dataSource" ref="dataSource" />
    <property name="transactionManager" ref="transactionManager" />
    <property name="databaseSchemaUpdate" value="false" />
    <property name="jobExecutorActivate" value="false" /><!--并行任务-->
</bean>

<!--核心对象:ProcessEngine-->
<bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
    <property name="processEngineConfiguration" ref="processEngineConfiguration" />
</bean>

<!--IdentityService:操作act_id_*表-->
<bean id="identityService" factory-bean="processEngine" factory-method="getIdentityService" />
<!--RepositoryService:操作act_re_*表,act_ge_bytearray-->
<bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService" />
<!--RuntimeService:操作act_ru_*表.-->
<bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService" />
<!--TaskService:操作act_ru_task|act_hi_taskinst表-->
<bean id="taskService" factory-bean="processEngine" factory-method="getTaskService" />
<!--HistoryService:操作act_hi_*表-->
<bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService" />
```



## 4.认识表结构

act_ge_bytearray:保存设计的bpmn文件的二进制数据.

act_id_*:系统用户与角色(用户组)信息

act_re_*:bpmn流程文件的部署记录

act_ru_*:保存正在执行中的流程数据信息(任务记录,流程变量)

act_hi_*:保存执行完成的历史任务信息



## 5.流程设计注意点

1.整个文件的id与name有意义,要着重写,尤其id

2.每个userTask的id,name必须完善,mainConfig下的assignee|candidate users|candidate groups三个三选一

​	assignee:任务执行人

​	candidate users:任务候选人

​	candidate groups:任务候选组

3.分支,需要引入gateway网关(排他,并行)

​	在网关后的分支,需要指定条件表达式${}





1.${loginuser}

2.部门经理审批:tw

3.总经理:候选组:总经理

4.人事归档:候选人:lyk,lm

5.${result=='同意' && days>3}

${result=='不同意'}

${result=='同意' && days<=3}

${suc.notify(execution)}

${error.notify(execution)}



审批结果:同意,不同意,驳回



## 4.IdentityService

主要用来创建用户与用户组,并维护用户与组之间的关系.





## 5.RepositoryService

主要用来做流程文件的部署,向数据库进行存储.





## 6.TaskService.RuntimeService,HistoryService





先维护用户与组,再进行流程设计.





申请单对象:与流程实例1-1,持有流程实例的外检

流程实例对象:与任务实例1-N,

任务实例对象:与流程实例与N-1,持有流程实例外键

审批记录对象:与任务实例时1-1,与流程实例是N-1,持有着任务的外检,持有流程实例外键