# SpringCloud

## 一、编码前准备

### 1.版本控制

![image-20200428192729155](D:\notes\SpringCloud\images\版本控制.png)

### 2.学习内容

![image-20200428193012210](D:\notes\SpringCloud\images\学习内容.png)

------

## 二、开始编码

### 1 微服务架构编码构建

#### 1.规矩

约定>配置>编码

------

#### 2.IDEA新建Project工作空间

总父工程POM：先建立project，然后再建立module。

微服务Cloud整体聚合父工程Project建立步骤：

1. New Project	![image-20200428200053148](D:\notes\SpringCloud\images\new pproject.png)
2. 聚合总父工程名字
3. Maven选版本![maven3-5-2 ](D:\notes\SpringCloud\images\maven3-5-2 .png)
4. 工程名字
5. 字符编码![image-20200428224127986](D:\notes\SpringCloud\images\字符编码.png)
6. 注解生效激活![image-20200428224350516](D:\notes\SpringCloud\images\注解生效激活.png)
7. java编译版本选8![image-20200428224552963](D:\notes\SpringCloud\images\java8.png)
8. File Type过滤![image-20200428224956223](D:\notes\SpringCloud\images\File Type过滤.png)

------

#### 3.配置DepencyManagement

1.DepencyManagement应用场景

在我们项目顶层的POM文件中，我们会看到dependencyManagement元素。通过它元素来==管理jar包的版本==，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着==父子层次==向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。

2.实现：

1. 指定这是一个POM工程![image-20200429082554595](D:\notes\SpringCloud\images\指定是一个pom工程.png)
2. 使用properties标签，可以在properties下声明相应的版本信息，然后在dependency下引用的时候用${spring-version}就可以引入该版本jar包了![image-20200429082622518](D:\notes\SpringCloud\images\properties统一管理jar包版本.png)
3. 引入各种jar包需要使用到的版本号![image-20200429082659404](D:\notes\SpringCloud\images\dependencyManagement.png)

#### 4.引入Build标签的时候会报错

当前解决方式：增加版本号

![image-20200429093045810](D:\notes\SpringCloud\images\spring-boot-maven-plugin爆红.png)

------

### 2 Rest微服务工程构建

#### 构建步骤

==微服务模块构建步骤：==

1. 建module
2. 改pom
3. 写yml
4. 主启动
5. 业务类

------

##### 1.cloud-provider-payment8001微服务提供者支付Module模块

![image-20200429093732866](D:\notes\SpringCloud\images\支付Module模块.png)

**1.建module**

第一步：

![image-20200429094950613](D:\notes\SpringCloud\images\建立支付module第一步.png)

第二步：

![image-20200429095024195](D:\notes\SpringCloud\images\建立支付模块第二步.png)

第三步：

![image-20200429095054772](D:\notes\SpringCloud\images\建立支付module第三步.png)

建立完成：

![image-20200429095119772](D:\notes\SpringCloud\images\支付Module建立完成.png)

------

**2.改pom**

![image-20200429111848721](D:\notes\SpringCloud\images\支付module改pom.png)



































## 三、导入jar包错误问题

1.默认的配置文件中使用的仓库是maven的中央仓库，有些老版本的spring包在最新的maven仓库中找不到，那么碰到这种情况我们应该怎么解决呢？

解决方案：在pom.xml文件中引入spring的插件仓库，在最后面添加：

```java
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
    	</repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>repository.spring.release</id>
            <name>Spring GA Repository</name>
            <url>https://repo.spring.io/plugins-release/</url>
		</pluginRepository>
    </pluginRepositories>
</project>
```

------

2.导入mybatis爆红

![image-20200429110916988](D:\notes\SpringCloud\images\导入mybatis爆红.png)

==解决方案：==

![image-20200429111044036](D:\notes\SpringCloud\images\解决mybatis爆红.png)